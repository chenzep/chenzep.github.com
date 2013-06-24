---
layout: post
title: "android日记保存"
date: 2013-06-22 12:11
comments: true
categories: Android

description: "android日记保存" 
keywords: Android log logcat 重定位 保存 文件
---
1. 概述  
	这几天在调试Android系统的LOG保存功能，把系统所有的LOG都保存到指定文件中，方便应用调试使用。查了网上的一些文章，处理方法就两种：
	a) 拦截`system\core\liblog\Logd_write.c`中的`__android_log_buf_write`函数。
	b) 参考或者直接调用`logcat`命令，把`/dev/log/...`块文件中的记录读取出来，并保存到文件中。
	测试了一下，发现这两种方法都有缺点:
	a) `__android_log_buf_write`函数是进程相关的，如果是通过条件变量来控制LOG保存与否，则条件变量只能在本进程中使用。也就是说不能控制系统或者其他应用的进程LOG的保存。还有一个就是权限问题，一般应用是无法把LOG保存到系统目录的。

	b)通过`logcat`方法，就是保存文件大小方面不好控制,如果没有这方面的需求，这种方法还是可以用的。

1. 代码流程  
	简单概况下Log.v的函数的实现代码，详细的原理说明参考[解读Android LOG机制的实现](http://www.cnblogs.com/hoys/archive/2011/09/30/2196199.html)。  
	a. Log.v函数实现文件为：`frameworks\base\core\java\android\util\Log.java`。
		函数直接调用JNI接口。
	
	    public static int v(String tag, String msg, Throwable tr) {  
       		return println_native(LOG_ID_MAIN, VERBOSE, tag, msg + '\n' + getStackTraceString(tr));
    	}
	b. JNI接口为：`android_util_Log_println_native`，实现文件是：`frameworks\base\core\jni\android_util_Log.cpp`，关键代码如下:

		static jint android_util_Log_println_native(JNIEnv* env, jobject clazz,
        	jint bufID, jint priority, jstring tagObj, jstring msgObj)
		{
    		......
			#tag, msg是tagObj, msgObj转换后的const char*类型指针。
    		int res = __android_log_buf_write(bufID, (android_LogPriority)priority, tag, msg);	
			......
    		return res;
		}
	c. __android_log_buf_write函数的实现文件为:`system\core\liblog\Logd_write.c`,代码只是把相关数据封装在vec数组中，然后把数组传递给write_to_log函数，完整代码如下：

		int __android_log_buf_write(int bufID, int prio, const char *tag, const char *msg)
		{
		    struct iovec vec[3];
		
		    if (!tag)
		        tag = "";
		
		    /* XXX: This needs to go! */
		    if (!strcmp(tag, "HTC_RIL") ||
		        !strncmp(tag, "RIL", 3) || /* Any log tag with "RIL" as the prefix */
		        !strcmp(tag, "AT") ||
		        !strcmp(tag, "GSM") ||
		        !strcmp(tag, "STK") ||
		        !strcmp(tag, "CDMA") ||
		        !strcmp(tag, "PHONE") ||
		        !strcmp(tag, "SMS"))
		            bufID = LOG_ID_RADIO;
		
		    vec[0].iov_base   = (unsigned char *) &prio;
		    vec[0].iov_len    = 1;
		    vec[1].iov_base   = (void *) tag;
		    vec[1].iov_len    = strlen(tag) + 1;
		    vec[2].iov_base   = (void *) msg;
		    vec[2].iov_len    = strlen(msg) + 1;
		
		    return write_to_log(bufID, vec, 3);
		}
	 d. write_to_log最终会调用__write_to_log_kernel函数,函数就是往文件中写数据。

		static int __write_to_log_kernel(log_id_t log_id, struct iovec *vec, size_t nr)
		{
		    ssize_t ret;
		    int log_fd;
		
		    if (/*(int)log_id >= 0 &&*/ (int)log_id < (int)LOG_ID_MAX) {
		        log_fd = log_fds[(int)log_id];
		    } else {
		        return EBADF;
		    }
		
		    do {
		        ret = log_writev(log_fd, vec, nr);
		    } while (ret < 0 && errno == EINTR);
		
		    return ret;
		}
	e. log文件是一个字符设备文件，而且和普通的块设备还不太一样,有自己独立的实现代码，文件为`lichee\linux-3.0\drivers\staging\android\logger.c`。这里要说明的是实际的写函数:`logger_aio_write`
		
		ssize_t logger_aio_write(struct kiocb *iocb, const struct iovec *iov,
			 unsigned long nr_segs, loff_t ppos)
		{
			struct logger_log *log = file_get_log(iocb->ki_filp);
			size_t orig = log->w_off;
			struct logger_entry header;
			struct timespec now;
			ssize_t ret = 0;
		
			now = current_kernel_time();
		
			header.pid = current->tgid;
			header.tid = current->pid;
			header.sec = now.tv_sec;
			header.nsec = now.tv_nsec;
			header.len = min_t(size_t, iocb->ki_left, LOGGER_ENTRY_MAX_PAYLOAD);
		
			/* null writes succeed, return zero */
			if (unlikely(!header.len))
				return 0;
		
			mutex_lock(&log->mutex);
		
			/*
			 * Fix up any readers, pulling them forward to the first readable
			 * entry after (what will be) the new write offset. We do this now
			 * because if we partially fail, we can end up with clobbered log
			 * entries that encroach on readable buffer.
			 */
			fix_up_readers(log, sizeof(struct logger_entry) + header.len);
		
			#写入时间，进程等信息
			do_write_log(log, &header, sizeof(struct logger_entry));
			#写入调试信息
			while (nr_segs-- > 0) {
				size_t len;
				ssize_t nr;
		
				/* figure out how much of this vector we can keep */
				len = min_t(size_t, iov->iov_len, header.len - ret);
		
				/* write out this segment's payload */
				nr = do_write_log_from_user(log, iov->iov_base, len);
				if (unlikely(nr < 0)) {
					log->w_off = orig;
					mutex_unlock(&log->mutex);
					return nr;
				}
		
				iov++;
				ret += nr;
			}
		
			mutex_unlock(&log->mutex);
		
			/* wake up any blocked readers */
			wake_up_interruptible(&log->wq);
		
			return ret;
		}	

		
1. 定制
	根据上面的代码流程，我们发现，在`logger_aio_write`添加保存LOG到文件中的功能是最好的，因为这里是内核层，不存在权限问题，也不存在进程共享的问题。
	比较头疼的是，在内核层，很多函数都无法使用，比如fopen,fread,...fclose,rename等。还好，内核层有对应的函数。
	下面是一些代码片段:

	
		const char *pri = iov[0].iov_base;

		const char *tag ;
		const char *msg;
		int taglen,msglen;
		const char *pCur;
		int ret;

		taglen = iov[1].iov_len;
		msglen = iov[2].iov_len;
		ret = copy_from_user(kernelBuffer, 		iov[1].iov_base, taglen);
		if (ret != 0) goto leave;
		ret = copy_from_user(kernelBuffer + taglen, 	iov[2].iov_base, msglen);
		if (ret != 0) goto leave;
	 	tag = kernelBuffer;
		msg = kernelBuffer + taglen;
		pCur = msg;

		if (saveFlag) {
			struct file *fileLog;
			mm_segment_t old_fs = get_fs();
			set_fs(KERNEL_DS);
			mutex_lock(&s_filemutex);
			fileLog = filp_open(path,O_CREAT|O_WRONLY,0666);
			if (!(IS_ERR(fileLog))) {
				int pos;
				int n;
				struct rtc_time tm;
				/filterPriToChar实现参考system\core\liblog\logprint.c文件
				char priChar = filterPriToChar(*pri);
				rtc_time_to_tm(header.sec, &tm);	
	
				n = sprintf(buffer,"%c %02d-%02d %02d:%02d:%02d.%03d %05d/%05d ",
					priChar, 
					tm.tm_mon+1,tm.tm_mday,tm.tm_hour,tm.tm_min,tm.tm_sec, header.nsec / 1000000, 
					header.pid, header.tid);

				#使用printf("%s",tag)有问题，Why.不太清楚是不是我那里搞错，还是内核不支持
				if (taglen > 1) {
					memcpy(buffer + n, tag, taglen -1);
					n += taglen - 1;
				}else{
					memcpy(buffer + n, "(null)", 6);
					n += 6;
				}
				buffer[n++] = ' ';
				
				if (msglen > 1) {
					memcpy(buffer + n, msg, msglen -1);
					n += msglen - 1;
				}else{
					memcpy(buffer + n, "(null)", 6);
					n += 6;
				}
				buffer[n++] = '\n';

				fileLog->f_op->llseek(fileLog,0,2);
				fileLog->f_op->write(fileLog,buffer,n,&fileLog->f_pos);
				pos = fileLog->f_pos;
				filp_close(fileLog,0);
			}
			mutex_unlock(&s_filemutex);
			set_fs(old_fs);
		}
		leave:
			ret = 0;
		