## 块设备层分析 ##

IO无论是经过EXT3文件系统还是块设备文件,最终都要通过writeback机制将数据刷新到磁盘，除非用户在对文件进行读写的时候采用了`Direct IO`的方式。为了提高性能，文件系统或者是裸设备都会采用Linux的cache机制对数据读写性能进行优化，因此都会采用writeback回写机制将数据写入磁盘。

`writeback`机制会调用不同底层设备的`address_space_operations`函数将数据刷新到设备。例如，EXT3文件系统会调用`blkdev_writepage`函数将radix tree中的page页写入设备。

在确认需要将一个page页写入设备时，最终都需要调用`submit_bh()`函数，该函数描述如下：


```
int submit_bh(int rw, struct buffer_head * bh)
{
	struct bio *bio;
	int ret = 0;
	BUG_ON(!buffer_locked(bh));
	BUG_ON(!buffer_mapped(bh));
	BUG_ON(!bh‐>b_end_io);
	BUG_ON(buffer_delay(bh));
	BUG_ON(buffer_unwritten(bh));
	/*
	* Only clear out a write error when rewriting
	*/
	if (test_set_buffer_req(bh) && (rw & WRITE))
	clear_buffer_write_io_error(bh);
	/*
	* from here on down, it's all bio ‐‐ do the initial mapping,
	* submit_bio ‐> generic_make_request may further map this bio around
	*/
	bio = bio_alloc(GFP_NOIO, 1);
	/* 封装BIO */
	bio‐>bi_sector = bh‐>b_blocknr * (bh‐>b_size >> 9); /* 起始地址 */
	bio‐>bi_bdev = bh‐>b_bdev; /* 访问设备 */
	bio‐>bi_io_vec[0].bv_page = bh‐>b_page; /* 数据buffer地址 */
	bio‐>bi_io_vec[0].bv_len = bh‐>b_size; /* 数据段大小 */
	bio‐>bi_io_vec[0].bv_offset = bh_offset(bh); /* 数据在buffer中的offset */
	bio‐>bi_vcnt = 1;
	bio‐>bi_idx = 0;
	bio‐>bi_size = bh‐>b_size;
	/* 设定回调函数 */
	bio‐>bi_end_io = end_bio_bh_io_sync;
	bio‐>bi_private = bh;
	bio_get(bio);
	/* 提交BIO至对应设备，让该设备对应的驱动程序进行进一步处理 */
	submit_bio(rw, bio);
	if (bio_flagged(bio, BIO_EOPNOTSUPP))
	ret = ‐EOPNOTSUPP;
	bio_put(bio);
	return ret;
}
```

`Submit_bh()`函数的主要任务是为page页分配一个bio对象，并且对其进行初始化，然后将bio提交给对应的块设备对象。提交给块设备的行为其实就是让对应的块设备驱动程序对其进行处理。在Linux中，每个块设备在内核中都会采用`bdev`(block_device)对象进行描述。通过bdev对象可以获取块设备的所有所需资源，包括如何处理发送到该设备的IO方法。因此，在初始化bio的时候，需要设备目标bdev，在Linux的请求转发层需要用到bdev对象对bio进行转发处理。

在通用块设备层，提供了一个非常重要的bio处理函数generic_make_request，通过这个函数实现bio的转发处理，该函数的实现如下：

```
void generic_make_request(struct bio *bio)
{
	struct bio_list bio_list_on_stack;

	if (!generic_make_request_checks(bio))
	return;
	/*
	* We only want one ‐>make_request_fn to be active at a time, else
	* stack usage with stacked devices could be a problem. So use
	* current‐>bio_list to keep a list of requests submited by a
	* make_request_fn function. current‐>bio_list is also used as a
	* flag to say if generic_make_request is currently active in this
	* task or not. If it is NULL, then no make_request is active. If
	* it is non‐NULL, then a make_request is active, and new requests
	* should be added at the tail
	*/
	if (current‐>bio_list) {
	bio_list_add(current‐>bio_list, bio);
	return;
	}/
	* following loop may be a bit non‐obvious, and so deserves some
	* explanation.
	* Before entering the loop, bio‐>bi_next is NULL (as all callers
	* ensure that) so we have a list with a single bio.
	* We pretend that we have just taken it off a longer list, so
	* we assign bio_list to a pointer to the bio_list_on_stack,
	* thus initialising the bio_list of new bios to be
	* added. ‐>make_request() may indeed add some more bios
	* through a recursive call to generic_make_request. If it
	* did, we find a non‐NULL value in bio_list and re‐enter the loop
	* from the top. In this case we really did just take the bio
	* of the top of the list (no pretending) and so remove it from
	* bio_list, and call into ‐>make_request() again.
	*/
	BUG_ON(bio‐>bi_next);
	bio_list_init(&bio_list_on_stack);
	current‐>bio_list = &bio_list_on_stack;
	do {
	/* 获取块设备的请求队列 */
	struct request_queue *q = bdev_get_queue(bio‐>bi_bdev);
	/* 调用对应驱动程序的处理函数 */
	q‐>make_request_fn(q, bio);
	bio = bio_list_pop(current‐>bio_list);
	} while (bio);
	current‐>bio_list = NULL; /* deactivate */
}
```

在`generic_make_request()`函数中，最主要的操作是获取请求队列，然后调用`make_request_fn()`方法处理bio。在Linux中一个块设备驱动通常可以分成两大类：有queue和无queue。有queue的块设备就是驱动程序提供了一个请求队列，`make_request_fn()`方法会将bio放入请求队列中进行调度处理，调度处理的方法有CFQ、Deadline和Noop之分。设置请求队列的目的是考虑了磁盘介质的特性，普通磁盘介质一个最大的问题是随机读写性能很差。为了提高性能，通常的做法是聚合IO，因此在块设备层设置请求队列，对IO进行聚合操作，从而提高读写性能。关于IO scheduler的具体算法分析请见后续文章。

在Linux的通用块层，提供了一个通用的请求队列压栈方法：`blk_queue_bio()`，在老版本的Linux中为`__make_request()`。在初始化一个有queue块设备驱动的时候，最终都会调用`blk_init_allocated_queue()`函数对请求队列进行初始化，初始化的时候会将`blk_queue_bio`方法注册到`q->make_request_fn`。在`generic_make_request()`转发bio请求的时候会调用`q‐>make_request_fn()`，从而可以将bio压入请求队列进行IO调度。一旦bio进入请求队列之后，可以好好的休息一番，直到unplug机制对bio进行进一步处理。另一类块设备是无queue的。无queue的块设备我们通常可以认为是一种块设备过滤驱动，这类驱动程序可以自己实现请求队列，绝大多数是没有请求队列的，直接对bio进行转发处理。这类驱动程序一个很重要的特征是需要自己实现`q‐>make_request_fn()`方法。这类驱动的`make_request_fn()`方法通常可以分成如下几个步骤：

> * 根据一定规则切分bio，不同的块设备可能存在不同的块边界，因此，需要对请求bio进行边界对齐操作。
> * 找到需要转发的底层块设备对象。
> * 直接调用generic_make_request函数转发bio至目标设备。

因此，无queue的块设备处理过程很直观。其最重要的作用是转发bio。在Linux中，device_mapper机制就是用来转发bio的一种框架，如果需要开发bio转发处理的驱动程序，可以在device_mapper框架下开发一个target，从而快速实现一个块设备驱动。

通过上述描述，我们知道，IO通过`writeback`或者`DirectIO`的方式可以抵达块设备层。到了块设备层之后遇到了两类块设备处理方法。如果遇到无queue块设备类型，bio马上被转发到其他底层设备；如果遇到了有queue块设备类型，bio会被压入请求队列，进行合并处理，等待unplug机制的调度处理。IO曾经在`page cache`游玩了很长时间，所有的请求在`page cache`受到得待遇是相同的，大家都会比较公平得被调度走，继续下面的旅程。但是，在块设备层情况就变的复杂了，不同IO受到的待遇会有所不同，这就需要看请求队列中的io scheduler具体算法了。因此，IO旅程在块设备这一站，最为重要的核心就是io scheduler。