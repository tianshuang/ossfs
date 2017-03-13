# OSSFS

[![Version](https://badge.fury.io/gh/aliyun%2Fossfs.svg)][releases]
[![Build Status](https://travis-ci.org/aliyun/ossfs.svg?branch=master)](https://travis-ci.org/aliyun/ossfs?branch=master)

### 简介

Forked from aliyun/ossfs，因为公司业务的关系，我对 ossfs 的源码进行了小小的修改，以适应公司业务的需要。

正如 commit 信息所述：disable list bucket and delete stat in memory，之所以要对这两处做修改是因为我们在测试环境中发现，一个简单的 list 请求可能会引发严重的性能问题，我们在测试环境的 bucket 根目录（其实OSS本身没有目录，只是路径拼接）中放入了 20 万个 object，只要使用 ls，机器在很长一段时间内都无法访问 OSS, 根据 DEBUG 日志发现，一次 ls 操作等于 1 dir + n objectattr 次 http 请求，大家可查看这个 [Issue](https://github.com/aliyun/ossfs/issues/13)，而我司业务的需求不需要用到 ls，并且其他监控进程扫描到挂载的目录也会触发 ls 导致严重的性能问题，所以我们将源码里 ls 那 n 次 objectattr 干掉了。

另外一处修改的地方就是不会再删除内存 map 中的 cachestat, 在 DEBUG 日志中我们发现每次 object 请求都会触发一次 HTTP 请求检查文件元信息，而我司的业务场景中自文件上传至 OSS 就不会再发生变化，所以我们也将这次检查干掉了，因为 OSS 的请求次数价格为 0.01元/万次，如果按每天千万次计算则为每天10元钱，啊，又帮公司省钱了～

### 功能

ossfs 基于s3fs 构建，具有s3fs 的全部功能。主要功能包括：

* 支持POSIX 文件系统的大部分功能，包括文件读写，目录，链接操作，权限，
  uid/gid，以及扩展属性（extended attributes）
* 通过OSS 的multipart 功能上传大文件。
* MD5 校验保证数据完整性。

### 局限性

ossfs提供的功能和性能和本地文件系统相比，具有一些局限性。具体包括：

* 随机或者追加写文件会导致整个文件的重写。
* 元数据操作，例如list directory，性能较差，因为需要远程访问oss服务器。
* 文件/文件夹的rename操作不是原子的。
* 多个客户端挂载同一个oss bucket时，依赖用户自行协调各个客户端的行为。例如避免多个客户端写同一个文件等等。
* 不支持hard link。
* 不适合用在高并发读/写的场景，这样会让系统的load升高

### 常见问题

[FAQ](https://github.com/aliyun/ossfs/wiki/FAQ)

### 相关链接

* [ossfs wiki](https://github.com/aliyun/ossfs/wiki)
* [s3fs](https://github.com/s3fs-fuse/s3fs-fuse) - 通过fuse接口，mount s3 bucket到本地文件系统。

### License

Copyright (C) 2010 Randy Rizun <rrizun@gmail.com>

Copyright (C) 2015 Haoran Yang <yangzhuodog1982@gmail.com>

Licensed under the GNU GPL version 2


[releases]: https://github.com/aliyun/ossfs/releases
[updatedb]: http://linux.die.net/man/8/updatedb
[faq-updatedb]: https://github.com/aliyun/ossfs/wiki/FAQ
[ecryptfs]: http://ecryptfs.org/
[xattr]: http://man7.org/linux/man-pages/man7/xattr.7.html
[supervisor]: http://supervisord.org/
[faq-supervisor]: https://github.com/aliyun/ossfs/wiki/FAQ#18
