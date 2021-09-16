
## 第 1 章 unix 基础知识

### 1. char *strerror(int errnum)
该函数将 errnum（就是 errno 值）映射为一个出错信息字符串，返回该字符串指针。声明在 string.h 文件中。


### 2.void perror(const char *s)
该函数基于当前的 errno 值，在标准出错文件中输出一条出错消息，然后返回。声明在 stdio.h 文件中。它首先输出由 s 指向的字符串，然后是一个冒号，一个空格，接着是 errno 值对应的出错信息，最后是一个换行符。


## 第 2 章 UNIX 标准化及实现

### 1. 获取系统运行配置
```c
long sysconf(int name)
long fpathconf(int fd, int name)
long pathconf(char *path, int name)
```
第一个函数用来获取系统运行时的配置信息。后两个函数用来获取文件的 name 选项的配置信息，它们的区别在于一个提供文件描述符，一个提供文件路径。声明在 unistd.h 文件中。这些 name 参数的常量值参考《apue》。


### 2. 系统基本数据类型
这些类型在不同系统上被声明为不同的 c 基本类型。因此使用它们可以增强代码的可移植性。

| **类型** | **含义** | **类型** | **含义** | **类型** | **含义** |
| --- | --- | --- | --- | --- | --- |
| caddr_t | 核心地址 | clock_t | 时钟滴答计数器（进程时间） | comp_t | 压缩的时钟时间 |
| dev_t | 设备号（主和次） | fd_set | 文件描述符集 | fpos_t | 文件位置 |
| gid_t | 用户组 ID | ino_t | i 节点编号 | mode_t | 文件类型，文件创建模式 |
| nlink_t | 目录项的链接技术 | off_t | 文件大小和偏移量 | pid_t | 进程 ID 和进程组 ID |
| ptrdiff_t | 两个指针相减的结果 | rlim_t | 资源限制 | sig_atomic_t | 能原子访问的数据类型 |
| sigset_t | 信号集 | size_t | 对象大小（例如字符串），无符号 | ssize_t | 返回的字节计数，带符号的（如 read，write 返回值，需要错误值） |
| time_t | 日历时间的秒计数器 | uid_t | 用户 ID | wchar_t | 能表示所有不同的字符码 |




## 第 3 章 文件 IO

### 1. 文件打开
```c
int open(const char *pathname, int flags)
int open(const char *pathname, int flags, node_t mode)
```
该函数用来打开文件，声明在 fcntl.h 中。flags 标志有如下：O_RDONLY，O_WRONLY，O_RDWR（这三个标志必须要包含其中之一），还有一些文件创建标志和状态标志，可以包含 0 个或多个，O_CLOEXEC，O_CREAT，O_DIRECTORY，O_EXEL，O_NOCITY，O_NOFOLLOW，O_TRUNC，O_TTY_INIT，O_NONBLOCK，O_SYNC。（这些都是打开的时候才设置，属于某个打开的文件，inode 中并不存在）
另外，当使用到 O_CREAT 标志来创建文件时，mode 中需要包含下列符号：S_IRWXU，S_IRUSR，S_IWUSR，S_IXUSR，S_IRWXG，S_IRGRP，S_IWGRP，S_IXGRP，S_IRWXO，S_IROTH，S_IWOTH，S_IXOTH。


### 2. 文件创建和关闭
```c
//该函数用来创建一个文件，声明在 fcntl.h 中。mode 中的值同上。
//当 open 中的 flags 包含 O_CREAT 时，open 和 creat 功能是一样的。
int creat(const char *pathname, mode_t mode);
该函数用来关闭有 open/creat 打开的文件描述符，声明在 unistd.h 中
int close(int fd);
```

### 4. 文件读写
```c
//该函数用来对已打开文件 fd 进行定位，声明在 unistd.h 中。
//whence 中可以包含如下值：SEEK_SET，SEEK_CUR，SEEK_END，分别表示从文件开头，文件当前位置
//文件结尾作为偏移量的开始。
off_t lseek(int fd, off_t offset, int whence);
//该函数用来从文件中读数据，声明在 unistd.h 中。
ssize_t read(int fd, void *buf, size_t count);
//该函数用来向文件中写数据，声明在 unistd.h 中。
ssize_t write(int fd, const void *buf, size_t count);
```

### 5. 文件原子读写
```c
//该函数类似于 read 函数，区别在于该函数是原子执行，并且不更新文件指针位置
//offset 用来设置在文件中读取的位置。声明在 unistd.h 中
ssize_t pread(int fd, void *buf, size_t count, off_t offset);
//该函数也类似于 pread 和 write。声明在 unistd.h 中。
ssize_t pwrite(int fd, const void *buf, size_t count, off_t offset);
//注意：两个函数主要用在多线程对同一文件进行读写的场合
```

### 9.int dup(int oldfd)



### int dup2(int oldfd, int newfd)

这两个函数用来复制文件描述符，声明在 unistd.h 中。dup 使用最小的空闲描述符作为新描述符，dup2 的 newfd 提供了新描述符。


### 10.void sync(void)



### int fsync(int fd)



### int fdatasync(int fd)

这三个函数用来将文件缓冲区中的数据写入磁盘文件中。声明在 unistd.h 中。区别在于，sync 将块缓冲区放入写队列就返回，并不等待实际写入磁盘中；fsync 将 fd 文件缓冲块
立即写入磁盘中，并等待写入后才返回，同时更新文件属性；fdatasync 类似于 syncfs，不过只影响文件的部分数据。


### 11. int fcntl(int fd, int cmd, ... /_ arg _/)

该函数用来更改已打开文件 fd 的属性，声明在 fcntl.h 中。该函数实现了 5 中功能，体现在不同的 cmd 所包含的标志中：

- 复制一个现有描述符（cmd=F_DUPFD）
- 获得 / 设置文件描述符标志（cmd=F_GETFD/F_SETFD）
- 获得 / 设置文件状态标志（cmd=F_GETFL/F_SETFL）
- 获得 / 设置异步 I/O 所有权（cmd=F_GETOWN/cmd=F_SETOWN）
- 获得 / 设置记录锁（cmd=G_GETLK/G_SETLK/F_SETLKW）


当 cmd 为 F_DUPFD，fcntl 函数相当于 dup/dup2 函数，返回复制后的新描述符，该描述符是空闲描述符中大于或等于第三个参数的值；当 cmd 为 F_GETFD，用来获取文件描述符的所有标志（当前只有一个标志 FD_CLOEXEC）;当 cmd 为 F_SETFD，用来设置文件描述父的标志（由第三个参数提供）;当 cmd 为 F_GETFL，用来获取文件的状态标志，文件状态标志如下所示 ;当 cmd 为 F_SETFL，用来设置文件的状态标志;当 cmd 为 F_GETOWN，用来取当前接收 SIGIO 和 SIGURG 信号的进程 ID 和进程组 ID;当 cmd 为 F_SETOWN，设置接收 SIGIO 和 SIGURG 信号的进程 ID 和进程组 ID。
文件状态标志：

- O_RDONLY               只读打开
- O_WRONLY              只写打开
- O_RDWR                   读写打开
- O_APPEND               追加打开
- O_NONBLOCK          非阻塞模式
- O_SYNC                   等待写完成（数据和属性）
- O_DSYNC                 等待写完成（仅数据）
- O_RSYNC                 同步读写
- O_FSYNC                 等待写完成（仅 FreeBSD 和 Mac OS X）
- O_ASYNC                 异步 I/O（仅 FreeBSD 和 Mac OS X）




### 12.int ioctl(int d, int request, ...)

该函数进行 I/O 设置，不能用本章中其他函数设置的 I/O 操作都可用该函数完成。比如终端 I/O 等等。声明在 sys/ioctl.h 中。


## 第 4 章 文件和目录



### 1.int stat(const char _path, struct stat _buf)



### int fstat(int fd, struct stat *buf)



### int lstat(const char _path, struct stat _buf)

这几个函数用来获取文件状态（从 inode 节点中取出，除了 st_ino 之外），将文件状态保存在 buf 指向的 struct stat 类型结构体中。声明在 sys/stat.h 中。区别在于，stat 和 lstat 中文件以路径名形式提供，fstat 中文件以文件描述符形式提供，另外，lstat 用来获得软连接的文件信息，而不是软连接所指向文件的信息。struct stat 结构体信息如下：

```c
struct stat {
               dev_t     st_dev;     /* ID of device containing file */
               ino_t     st_ino;     /* inode number */
               mode_t    st_mode;    /* protection */  // 该域中保存了文件的访问权限以及文件类型的信息，还包括了设置用户 ID 位和设置组 ID 位
               nlink_t   st_nlink;   /* number of hard links */
               uid_t     st_uid;     /* user ID of owner */
               gid_t     st_gid;     /* group ID of owner */
               dev_t     st_rdev;    /* device ID (if special file) */
               off_t     st_size;    /* total size, in bytes */
               blksize_t st_blksize; /* blocksize for filesystem I/O */
               blkcnt_t  st_blocks;  /* number of 512B blocks allocated */
               time_t    st_atime;   /* time of last access */
               time_t    st_mtime;   /* time of last modification */
               time_t    st_ctime;   /* time of last status change */
           };
```



### 2.S_ISDIR



### S_ISREG



### S_ISCHR



### S_ISBLK



### S_ISFIFO



### S_ISLNK



### S_ISSOCK

这几个都是宏定义，依据 struct stat 中 st_mode 成员的值进行判断（eg: S_ISDIR（buf.st_mode）），当前文件是否是目录文件，普通文件，字符设备，块设备，FIFO，软连接，套接字。声明在 sys/stat.h 中。


### 3.S_ISUID



### S_ISGID

这两个宏依据 struct stat 中 st_mode 成员的值，来测试该文件是否设置了 “设置用户 ID 位” 和“设置组 ID 位”。


### 4.int access(const char *pathname, int mode)

该函数用来检测当前进程的实际用户和实际组是否有访问由 pathname 所指示的文件。声明在 unistd.h 文件中。mode 可取以下值：

- R_OK      测试读权限:
- W_OK      测试写权限
- X_OK       测试执行权限
- F_OK       测试文件是否存在




### 5.mode_t umask(mode_t mask)

该函数用来设置当前进程的文件模式创建屏蔽字，这会屏蔽掉当前进程中创建的文件的那些权限。声明在 sys/stat.h 文件中。该函数不改变 shell 的屏蔽字。mask 取值集合和 open 函数的 mode 参数一致。注意：umask 中 mask 包含的权限会被屏蔽掉，这和 open 用法相反。


### 6.int chmod(const char *path, mode_t mode)



### int fchmod(int fd, mode_t mode)

这两个函数用来改变文件的访问权限。声明在 sys/stat.h 文件中。区别在于一个提供了文件的路径，另一个提供了文件描述符。此外，该函数的 mode 中除了能包含 open 的 mode 取值集合，还可以包含如下三个：S_ISUID，S_ISGID，S_ISVTX，其中前两个分别是设置用户 ID 位和设置组 ID 位，第三个是粘住位。


### 7.int chown(const char *path, uid_t owner, gid_t group)



### int fchown(int fd, uid_t owner, gid_t group)



### int lchown(const char *path, uid_t owner, gid_t group)

这几个函数用来设置文件的用户 ID 和组 ID。声明在 unistd.h 文件中。第一三个通过文件路径来提供文件，第二个用 fd 提供文件。第三个函数只是修改符号链接的用户 ID 和组 ID，没有修改符号链接指向的文件。只有调用这些函数的进程的有效用户为文件的拥有者时才能完成修改，超级用户可以修改任何文件。


### 8.int truncate(const char *path, off_t length)



### int ftruncate(int fd, off_t length)

这两个函数用来将文件截短为 length 字节（即将 length 字节以后的部分去掉）。声明在 unistd.h 文件中。


### 9.int link(const char _oldpath, const char _newpath)

该函数为 oldpath 文件创建一个硬链接 newpath（实际上就是个目录项）。声明在 unistd.h 文件中。最后 oldpath 和 newpath 指向了同一个 inode 节点。inode 中的链接计数加 1。
创建硬链接一般需要：1. 超级用户权限才能创建目录的硬链接，2. 硬链接和文件位于同一文件系统中。


### 10.int unlink(const char *pathname)

该函数删除一个目录项（也就是删除一个硬链接），该目录项指向的 inode 中的链接计数减 1。声明在 unistd.h 文件中。


### 11.int remove(const char *pathname)

这是 c 的标准库函数，声明在 stdio.h 中。用来删除文件或目录的链接。删除文件时相当于 unlink，删除目录时相当于 rmdir。


### 12.int rename(const char _oldpath, const char _newpath)

该函数是 c 的标准库函数，声明在 stdio.h 中。用来为文件或者目录重命名。


### 13.int symlink(const char _oldpath, const char _newpath)

该函数用来为 oldpath 文件创建一个符号链接 newpath。声明在 unistd.h 文件中。


### 14.ssize_t readlink(const char _path, char _buf, size_t bufsiz)

该函数用来读一个符号链接文件中的值（而不是所指向的文件）。声明在 unistd.h 文件中。虽然 open 函数可以打开文件，但是由于其跟随符号链接（打开符号链接所指向的文件），故不能打开符号链接文件本身。


### 15.int utime(const char _filename, const struct utimbuf _times)

该函数用来更改一个文件的访问和修改时间。声明在 utime.h 文件中。struct utimbuf 结构体如下：
struct utimbuf {time_t actime;       /_ access time _/time_t modtime;      /_ modification time _/};


### 16.int mkdir(const char *pathname, mode_t mode)

该函数用来创建一个新目录。声明在 sys/stat.h 文件中。创建目录时，至少需要设置一个执行权限位，以允许访问该目录的文件名。


### 17.int rmdir(const char *pathname)

该函数用来删除一个空目录。声明在 unistd.h 文件中。


### 18.DIR _opendir(const char _name)



### DIR *fdopendir(int fd)

这两个函数用来打开目录文件。声明在 dirent.h 文件中。返回一个指向目录流的指针，目录流不用关心。


### 19.struct dirent _readdir(DIR _dirp)

该函数从打开的目录文件中读取目录项，保存到 struct dirent 结构中，并返回指向该结构的指针。声明在 dirent.h 文件中。该结构体如下：

```c
struct dirent {
               ino_t          d_ino;       /* inode number */
               off_t          d_off;       /* not an offset; see NOTES */
               unsigned short d_reclen;    /* length of this record */
               unsigned char  d_type;      /* type of file; not supported
                                              by all filesystem types */
               char           d_name[256]; /* filename */
           };
```



### 20.void rewinddir(DIR *dirp)

该函数用来重置目录流内的指针位置，使之指向目录的开头。声明在 dirent.h 文件中。


### 21.int closedir(DIR *dirp)

该函数关闭 dirp 所指向的目录流。声明在 dirent.h 文件中。


### 22.long telldir(DIR *dirp)

该函数返回目录流内指针的当前位置。声明在 dirent.h 文件中。


### 23.void seekdir(DIR *dirp, long loc)

该函数设置目录流内的指针位置，下一次 readdir 将从设置好的位置上读取。声明在 dirent.h 文件中。


### 24.int chdir(const char *path)



### int fchdir(int fd)

这两个函数更改当前进程的当前工作目录。声明在 unistd.h 文件中。


### 25.char _getcwd(char _buf, size_t size)

该函数获取当前进程的当前工作目录。声明在 unistd.h 文件中。


## 第 5 章 标准 I/O 库



### 1.int fwide(FILE *stream, int mode)

该函数设置文件流 stream 的定向（字节定向或宽定向）。但是不设置已经定向的流的定向。该函数无出错返回（返回负值说明是字节定向，返回正值是宽定向，返回 0 无定向）。声明在 wchar.h 文件中。根据 mode 取值不同，函数功能不同：

- mode 值为负，将流设置为字节定向。
- mode 值为正，将流设置为宽定向。
- mode 值为 0，不设置流的定向，但返回该流定向值。




### 2.void setbuf(FILE _stream, char _buf)



### int setvbuf(FILE _stream, char _buf, int mode, size_t size)

这两个函数改变给定流 stream 的缓冲类型。声明在 stdio.h 文件中。对于 setbuf 而言，当 buf 为 NULL，则 stream 流被设置为无缓冲；否则，buf 的长度应该定义为 BUFSIZE（在 stdio.h 中定义），将根据 stream 流是磁盘文件或者终端设备而将其设置为全缓冲或者行缓冲。对 setvbuf 而言，根据 mode 取值不同，设置的缓冲区类型也不同：

- mode 取值_IOFBF，设置为全缓冲，buf 为非空，全缓冲为用户的 buf，大小为 size；buf 为 NULL，系统自动分配合适大小的全缓冲区。
- mode 取值_IOLBF，设置为行缓冲，同上，全缓冲改为行缓冲即可。
- mode 取值_IONBF，设置为无缓冲。（忽略 buf 和 size）


此外，setvbuf 函数要在所有对流进行操作的函数之前被调用（打开流以后就立即调用该函数）。


### 3. int fflush(FILE *stream)

该函数强制冲洗一个流，将 stream 流的所有未写数据传送到内核（然后写入磁盘 / 终端设备中）。若 stream 为 NULL，强制冲洗所有输出流。


### 4.FILE _fopen(const char _path, const char *mode)



### FILE _freopen(const char _path, const char _mode, FILE _stream)



### FILE _fdopen(int fd, const char _mode)

这几个函数打开一个标准 I/O 流。声明在 stdio.h 文件中。fopen 打开指定的文件；freopen 在指定的流上打开一个指定的文件，如果指定流已打开，那么先关闭该流，如果指定流已定向，那么清除该定向，该函数一般用来将一个指定的文件打开到标准输入 / 输出 / 出错流上；fdopen 为一个打开的文件描述符关联一个标准 I/O 流。这几个函数的 mode 取值如下：

| mode | value |
| --- | --- |
| r/rb | 为读而打开 |
| w/wb | 为写而打开或为写而创建，并将文件截短为 0 |
| a/ab | 为追加而打开或为追加而创建 |
| r+/r+b/rb+ | 为读写而打开 |
| w+/w+b/wb+ | 为读写而打开，并将文件截短为 0 |
| a+/a+b/ab+ | 为读或追加打开或创建 |


（b 仅代表二进制文件，不影响其他）
fdopen 的 mode 取值有些特殊，取各种 “写” 的时候，并不截短文件（因为 fd 表明该文件已经打开，而不是由标准 I/O 打开的，所以标准 I/O 没资格截短）。而且取各种 “写” 的时候不能没有创建功能，因为 fd 文件已存在，无须创建。
另外，如果多个进程使用标准 I/O 打开一个文件，那么并发去写文件都可以正确将数据写入文件，标准 I/O 流的强大之处。。还有，如果同时以读和写打开一个文件，那么：1. 如果中间没有 fflush，fseek，fsetpos 或 rewind，则在输出后不能直接跟随输入；2. 如果中间没有 fseek，fsetpos 或 rewind，或者一个输入操作没有达到文件尾部，则在输入操作之后不能直接跟随输出。
最后，用标准 I/O 创建的文件无法说明文件的访问权限。


### 5.int fclose(FILE *fp)

该函数关闭标准 I/O 流 fp。声明在 stdio.h 文件中。关闭之前冲洗缓冲区的输出数据，丢弃缓冲区中的输入数据。


### 6.int getc(FILE *stream)



### int fgetc(FILE *stream)



### int getchar(void)

这些函数从标准 I/O 流 stream 中每次读一个字符。声明在 stdio.h 文件中。其中 getc 是宏函数，getchar 相当于 fgetc（stdin）。返回值为读到的字符，字符类型本为 unsigned char，在这里强制转化成 int 类型，是为了当出错或者到达文件末尾的时候能够返回 - 1。在 stdio.h 中，EOF 定义为 - 1，因此将这些函数返回值和 EOF 比较就可知道是否出错或者是否到达文件末尾。


### 7.int ferror(FILE *stream)



### int feof(FILE *stream)



### void clearerr(FILE *stream)

前两个函数是为了判断流是否出错。每个流在 FILE 对象中维持了两个标志：出错标志和文件结束标志（用以区分出错和到达文件末尾）。它们都声明在 stdio.h 文件中。第三个函数清除这两个标志。


### 8.int ungetc(int c, FILE *stream)

该函数将字符 c 回送到 stream 流中，而不是文件中。下次从流中读取时，将读到该字符。声明在 stdio.h 文件中。


### 9.int putc(int c, FILE *stream)



### int fputc(int c, FILE *stream)



### int putchar(int c)

这三个函数将字符 c 输出到输出流中。和 6 中的三个函数类似，putc 是宏函数，putchar 相当于 fputc（c，stdout）。声明在 stdio.h 文件中。


### 10.char _gets(char _s)



### char _fgets(char _s, int size, FILE *stream)

gets 函数从 stdin 中读取一行字符存入 s 中（在 stdin 中遇到换行符或者 EOF 结束标志就算一行），不推荐使用该函数，容易造成缓冲区溢出。fgets 从指定的流 stream 中读取，并且指定了缓冲区大小为 size，也是每次读取一行，读取的字符数最多为 size-1，最后加‘\0’。对于这两个函数而言，读取到一行后，会将'\n'或者 EOF 替换为‘\0’。它们都声明在 stdio.h 文件中。


### 11.int fputs(const char _s, FILE _stream)



### int puts(const char *s)

fputs 函数将以‘\0’结尾的串 s 输出到 stream 流中。puts 函数将以‘\0’结尾的串 s 输出到 stdout 中。‘\0’都不写入流中。puts 函数最后会将一个‘\n’输出到 stdout 流中。它们都声明在 stdio.h 文件中。


### 12.size_t fread(void _ptr, size_t size, size_t nmemb, FILE _stream)



### size_t fwrite(const void _ptr, size_t size, size_t nmemb, FILE _stream)

这两个函数是二进制 I/O 函数，可以进行块的读写操作。声明在 stdio.h 文件中。


### 13.long ftell(FILE *stream)



### int fseek(FILE *stream, long offset, int whence)



### void rewind(FILE *stream)

这几个函数用来对标准 I/O 流进行定位（偏移量都以字节为单位）。ftell 获取流内指针的当前位置，  fseek 将流内指针偏移量设为 offset，whence 提供了偏移量的相对起始位置，这和第三章的第 4 个函数 lseek 的该参数取值相同。rewind 函数将流内指针设置到文件的开头。对于 fseek 函数而言，定位文本文件时，whence 只能取 SEEK_SET，offset 只能取两种值：0 或 ftell 的返回值。它们都声明在 stdio.h 文件中。


### 14.int fseeko(FILE *stream, off_t offset, int whence)



### off_t ftello(FILE *stream)

这两个函数和 13 中前两个函数用法相同，区别是这两个函数的偏移类型为 off_t。声明在 stdio.h 文件中。


### 15.int fgetpos(FILE _stream, fpos_t _pos)



### int fsetpos(FILE _stream, fpos_t _pos)

第一个函数将定位的流内指针位置保存到 pos 中，第二个函数可以使用 pos 将流内指针设置为该位置。声明在 stdio.h 文件中。


### 16.int printf(const char *format, ...)



### int fprintf(FILE _stream, const char _format, ...)



### int sprintf(char _str, const char _format, ...)



### int snprintf(char _str, size_t size, const char _format, ...)

这几个函数用于格式化输出。声明在 stdio.h 文件中。第一个函数输出到标准输出流 stdout。第二个函数输出到 stream 流中。后两个函数输出到 str 缓冲区中。sprintf 可能会造成缓冲区溢出，因此不建议使用。snprintf 规定了缓冲区大小为 size，因此比较安全。


### 17.int vprintf(const char *format, va_list ap)



### int vfprintf(FILE _stream, const char _format, va_list ap)



### int vsprintf(char _str, const char _format, va_list ap)



### int vsnprintf(char _str, size_t size, const char _format, va_list ap)

这四个函数和 16 中的函数类似，区别是将可变参数替换成了 ap 参数。声明在 stdarg.h 文件中。


### 18.int scanf(const char *format, ...)



### int fscanf(FILE _stream, const char _format, ...)



### int sscanf(const char _str, const char _format, ...)

这几个函数用于格式化输入。声明在 stdio.h 文件中。第一个函数从标准输入流 stdin 中读取，第二个函数从 stream 流中读取，第三个函数从 str 缓冲区中读取。


### 19.int vscanf(const char *format, va_list ap)



### int vsscanf(const char _str, const char _format, va_list ap)



### int vfscanf(FILE _stream, const char _format, va_list ap)

这几个函数类似于 18 中的函数，也是将可变参数替换成了 ap 参数。声明在 stdarg.h 文件中。


### 20.int fileno(FILE *stream)

该函数将 stream 流所关联的文件描述符返回。声明在 stdio.h 文件中。


### 21.char _tmpnam(char _s)

第一个函数用来产生一个与现有文件名不同的有效路径名字符串。如果 s 为 NULL，函数自动申请空间来存放有效路径名字符串，并返回空间地址，如果 s 不为 NULL，则将有效路径名字符串存放于 s 中，并将 s 地址返回，然后就可用该路径名在程序中手动创建文件了。声明在 stdio.h 文件中。该函数有个问题，就是在产生出一个有效文件名和使用该文件名创建文件中间一般会有时间差，在该时间差内可能别的进程会使用该文件名创建文件，这样就会出现问题。


### 22.FILE *tmpfile(void)



### char _tempnam(const char _dir, const char *pfx)

第一个函数创建一个临时二进制文件，在关闭该文件或程序结束时将自动删除该文件。第二个函数也是用来创建一个临时文件。关闭该文件或者程序运行完后文件会被自动删除。声明在 stdio.h 文件中。其中，dir 规定了创建临时文件所在的目录，pfx 如果非 NULL 的话，是一个最多包含 5 个字符的字符串，作为文件名的头几个字符。第二个函数也存在 21 函数的时间差问题。dir 有如下要求：

- 如果定义了环境变量 TMPDIR，则用其作为目录。
- 如果 dir 非空，则用 dir 作为目录。
- 将 stdio.h 中的字符串 P_tmpdir 用作目录。
- 将本地目录（通常是 / tmp）用作目录。


该函数自动分配空间来存放临时的文件名，不接受用户自己的 buf。


### 23.  int mkstemp(char *template)

该函数也是用来创建一个临时文件，返回文件描述符。与上边的函数不同的是，它所创建的文件不会被自动删除。临时文件名由 template 参数提供，该串的最后 6 个字符为 XXXXXX，然后该函数用不同字符代替 XXXXXX，以创建唯一路径名。声明在 stdio.h 文件中。


## 第 6 章 系统数据文件和信息



### 1.struct passwd *getpwuid(uid_t uid)



### struct passwd _getpwnam(const char _name)

这两个函数通过口令文件（/etc/passwd，该文件中存放了用户的所有信息除了用户密码），将 inode 中的用户 ID 或者登录时输入的登录名在口令文件中的项找出来，填入 struct passwd 结构中，并返回该结构指针，该结构是函数中定义的静态结构，以后的函数调用会不断覆盖该结构的内容。该结构中包含了用户的各种信息。声明在 pwd.h 文件中。


### 2.struct passwd *getpwent(void)



### void setpwent(void)



### void endpwent(void)

第一个函数在循环中，可以逐个取出口令文件中的项。第二个函数用于返绕口令文件，即设置口令文件指针到文件的开头。第三个函数用于关闭由第一个函数打开的口令文件。它们都声明在 pwd.h 文件中。


### 3.struct spwd _getspnam(const char _name)



### struct spwd *getspent(void)



### void setspent(void)



### void endspent(void)

这几个函数用来读取阴影文件（/etc/shadow，保存着用户名和用户密码）的项。声明在 shadow.h 文件中。这些函数用法和 1，2 函数类似。第一个函数根据用户名 name 获得对应的项，第二个函数在循环中可以逐个取出阴影文件中的项，最后两个函数用来返绕和关闭阴影文件。


### 4.struct group *getgrgid(gid_t gid)



### struct group _getgrnam(const char _name)

这两个函数通过组文件（/etc/group，该文件中存放了所有用户组的信息），将组 ID 或者组名对应的项读取出来，写入 struct group 结构，并返回该结构的指针。类似于 1 中的函数。声明在 grp.h 文件中。struct group 结构体如下：

```c
struct group {
               char   *gr_name;       /* group name */
               char   *gr_passwd;     /* group password */
               gid_t   gr_gid;        /* group ID */
               char  ### gr_mem;        /* group members */  // 该组的所有用户名构成的数组
           };
```



### 5.struct group *getgrent(void)



### void setgrent(void)



### void endgrent(void)

这些函数用来读取组文件中的项，类似于 2 中的函数。声明在 grp.h 文件中。


### 6.int getgroups(int size, gid_t list[])

该函数用来获取当前用户的附加组 ID。声明在 unistd.h 文件中。每个用户除了可以加入口令文件中显示的那个组外，同时可以加入若干个附加组。对于该函数，list 数组用来存放所有的附加组 ID 数值，size 说明了数组的元素个数。当 size 为 0 是，该函数返回当前用户的附加组数（可以用来分配 list 的长度）。


### 7.int setgroups(size_t size, const gid_t *list)

该函数为当前用户设置附加组 ID 表。声明在 grp.h 文件中。该函数一般只在 initgroups 函数中进行调用。


### 8.int initgroups(const char *user, gid_t group)

该函数用来为当前用户初始化附加组 ID 表。声明在 grp.h 文件中。该函数会读整个组文件，找到 user 所在的所有组，构成 list 数组后，调用 setgroups 函数来设置。并将当前用户所在组的 ID 号 group 也存入附加组 ID 表。


### 9.struct utmp


```c
struct utmp {
    short   ut_type;              /* Type of record */
    pid_t   ut_pid;               /* PID of login process */
    char    ut_line[UT_LINESIZE]; /* Device name of tty - "/dev/" */
    char    ut_id[4];             /* Terminal name suffix,
                                     or inittab(5) ID */
    char    ut_user[UT_NAMESIZE]; /* Username */
    char    ut_host[UT_HOSTSIZE]; /* Hostname for remote login, or
                                     kernel version for run-level
                                     messages */
    struct  exit_status ut_exit;  /* Exit status of a process
                                     marked as DEAD_PROCESS; not
                                     used by Linux init(8) */
    /* The ut_session and ut_tv fields must be the same size when
       compiled 32- and 64-bit.  This allows data files and shared
       memory to be shared between 32- and 64-bit applications. */
#if __WORDSIZE == 64 && defined __WORDSIZE_COMPAT32
    int32_t ut_session;           /* Session ID (getsid(2)),
                                     used for windowing */
    struct {
        int32_t tv_sec;           /* Seconds */
        int32_t tv_usec;          /* Microseconds */
    } ut_tv;                      /* Time entry was made */
#else
     long   ut_session;           /* Session ID */
     struct timeval ut_tv;        /* Time entry was made */
#endif

    int32_t ut_addr_v6[4];        /* Internet address of remote
                                     host; IPv4 address uses
                                     just ut_addr_v6[0] */
    char __unused[20];            /* Reserved for future use */
};
```

/run/utmp 文件中保存了当前所有登录到系统中的用户。通过该结构体可以对文件中的项进行读取和写入。


### 10.int uname(struct utsname *buf)

该函数可以获取当前主机和操作系统的有关信息。声明在 sys/utsname.h 文件中。struct utsname 结构体如下：

```c
struct utsname {
    char sysname[];    /* Operating system name (e.g., "Linux") */
    char nodename[];   /* Name within "some implementation-defined
                          network" */
    char release[];    /* Operating system release (e.g., "2.6.28") */
    char version[];    /* Operating system version */
    char machine[];    /* Hardware identifier */
#ifdef _GNU_SOURCE
    char domainname[]; /* NIS or YP domain name */
#endif
};
```



### 11.int gethostname(char *name, size_t len)

该函数获取主机的名字存入 name 数组中，len 指定了数组长度。声明在 unistd.h 文件中。


### 12.time_t time(time_t *t)

该函数返回当前时间和日期（从 1970.1.1 0:0:0 开始至今的秒数），存入 t 所指向的变量中。声明在 time.h 文件中。


### 13.int gettimeofday(struct timeval _tv, struct timezone _tz)

该函数也是返回当前时间，将时间保存在 struct timeval 结构体中。声明在 sys/time.h 文件中。该函数比 time 函数有更高分辨率（微秒级），tz 一般为 NULL。struct timeval 结构体如下：

```c
struct timeval {
    time_t      tv_sec;     /* seconds */   // 秒，也是从 1970.1.1 0:0:0 开始至今的秒数，和 time 函数获取数值的相同
    suseconds_t tv_usec;    /* microseconds */    // 微秒
};
```



### 14.struct tm _gmtime(const time_t _timep)



### struct tm _localtime(const time_t _timep)

第一个函数将 timep 的秒数转换成国际标准时间（年月日时分秒星期），第二个函数将 timep 的秒数转换成本地时间。转换后的时间保存在 struct tm 静态结构体中，然后返回结构体指针。声明在 time.h 文件中。


### 15.time_t mktime(struct tm *tm)

该函数将本地时间转换成 time_t 类型的值（秒数）。声明在 time.h 文件中。


### 16.char _asctime(const struct tm _tm)



### char _ctime(const time_t _timep)

这两个函数分别将 struct tm 类型的时间和 time_t 类型的秒数转换成 26 字节的字符串。eg：Sun Sep  7 22:27:44 CST 2014。声明在 time.h 文件中。


### 17.size_t strftime(char _s, size_t max, const char _format,  const struct tm *tm)

该函数将 tm 中的时间格式化输出到长度为 max 的 s 数组中。若 max 大小不够，则函数返回 0。否则返回存入的字符数。声明在 time.h 文件中。format 格式和 printf 的 format 类似，但有一些不同，相见《APUE》p145 页。


## 第 7 章 进程环境



### 1.void exit(int status)



### void _Exit(int status)

这两个函数用来正常终止一个进程。第二个函数立即进入内核，第一个函数执行完清理工作（关闭标准 IO）然后进入内核。status 是进程的终止状态。声明在 stdlib.h 文件中。
在 main 函数中调用 exit 等价于 return 语句。


### 2.void _exit(int status)

该函数也是用来正常终止一个进程。并且直接进入内核。声明在 unistd.h 文件中。status 是进程的终止状态。
注：在我的 ubuntu14.04 上测试，main 函数的结束处未显式的写出返回语句（18，19 函数或 return），或者返回语句中没有明确的返回状态（数字），则进程终止状态均为 6，而不是《apue》中提到的 0。


### 3.进程的 5 种正常终止方式


- 在 main 函数中执行 return。（等价于执行 exit，也就是在主线程中结束进程，如果仅想结束主线程而不是进程，则在 main 最后调用 pthread_exit 即可）
- 调用 exit 函数（主线程中或者任意子线程中调用）。该函数将调用由 atexit 登记过的终止处理函数，并关闭所有的标准 I/O 流。
- 调用_exit 或_Exit 函数。这两个函数直接陷入内核，因此就不会执行终止处理程序或者信号处理程序。
- 进程的最后一个线程从其启动例程（就是线程的执行函数）中返回。线程的返回值不会做为进程的返回值，进程将以终止状态 0 返回。
- 进程的最后一个线程调用 pthread_exit 函数。这种情况下，进程的终止状态为 0，而不是 pthread_exit 的参数




### 3 种异常终止方式


- 调用 abort。
- 当进程接收到某些信号。
- 最后一个线程对 “取消” 请求做出响应。


无论进程以何种方式结束，最终会执行内核中的同一段代码，来关闭所有打开的文件描述符，释放它所使用的存储器等。如果子进程希望父进程知道它是如何结束的，通过调用 exit，_exit，_Exit 函数，并把退出状态作为参数传递给它们，而在异常终止情况下，内核会产生一个指示其异常终止原因的终止状态，最终父进程通过调用 wait 或 waitpid 函数就可以获取到终止状态。exit 和_Exit 最终还是调用_exit 函数，将退出状态转换成进程的终止状态。exit，_exit，_Exit 三个函数的参数（退出状态）必然等于进程的终止状态，但是线程的结束函数 pthread_exit 的参数（退出状态）不会等于进程的终止状态。


### 20.int atexit(void (*function)(void))

由该函数登记的 function 函数，会被 exit（或者主线程的 return / 最后一个线程的 return）自动调用，也就说只有在进程正常结束时才调用，线程结束不会调用，就算是在某个非主线程中用 atexit 设置的清理函数，也得等到整个进程结束时才调用。声明在 stdlib.h 文件中。


### 21.void *malloc(size_t size)



### void *calloc(size_t nmemb, size_t size)



### void _realloc(void _ptr, size_t size)



### void free(void *ptr)

前三个函数在堆中为进程分配空间，最后一个函数释放这些空间。其中，malloc 函数不对分配的空间进行初始化，calloc 函数将分配的空间初始化为 0，realloc 用来调整（变大或减小）由前两个函数非配空间的大小，其中 size 参数是新存储区的大小，不是新旧存储区大小之差，当扩大空间时，如果原存储区周围的空闲空间不足，则在其他地方重新开辟空间，再将 原存储区中的值拷贝到新空间；如果原存储区周围空间足够，则在原存储区位置上向高地址方向扩充。声明在 stdlib.h 文件中。


### 22.char _getenv(const char _name)

该函数在进程的环境表（环境表是一个指针数组，每个元素指向了 “name=value” 形式的串）中查找 name 的环境变量值。返回一个字符串指针。声明在 stdlib.h 文件中。


### 23.int putenv(char *string)



### int setenv(const char _name, const char _value, int overwrite)



### int unsetenv(const char *name)###

这几个函数用来设置进程环境表中的环境变量值。声明在 stdlib.h 文件中。对于 putenv 函数，会将 string 指向的串（形式为 “name=value”）放到环境表 name 的项中，如果 name 已存在则删除 name 的定义。对于 setenv 函数，name 和 value 值分别由两个参数提供，如果 overwrite 非 0，则会删除环境表中已存在的 name 定义，如果 overwrite 为 0，则不会删除删除环境表中已存在的 name 定义，也不设置新的 value 值，也不出错。对于 unsetenv 函数，将删除环境表中已存在的 name 定义，若环境表中不存在 name 的定义也不出错。
在改变或者增加环境变量的时候，有如下规定：

- 修改一个现有 name，如果新 value 长度小于或等于现有 value 长度，则就在原空间中写入新字符串。
- 修改一个现有 name，如果新 value 长度大于现有 value 长度，则用 malloc 在堆中为新 value 分配空间，然后将该空间地址放入环境表中原来的 name 元素中。
- 新增一个 name，如果是第一次新增，调用 malloc 分配一个新的环境表，并将原来环境表中的值复制过来，将新的 value 串的指针放在环境表的表尾，再在其后放入一个空指针。此时环境表中的其他大多数指针仍指向了栈顶之上的各个 value 串。
- 新增一个 name，如果这不是第一次新增，说明之前调用过 malloc，则使用 realloc 改变环境表的大小，之后操作类似于 c。




### 24.int setjmp(jmp_buf env)



### void longjmp(jmp_buf env, int val)

第一个函数将调用时的堆栈内容保存在 env 中，以供第二个函数使用。第二个函数使用 env 值，会跳到设置 env 的函数中（即 setjmp），由于 setjmp 可以对应多个 longjmp 函数，因此 val 变量传递的数字会对多个 longjmp 函数加以区分。
《apue》上说，对于无优化的编译，使用 longjmp 函数跳回后，全局变量，静态局部变量，自动变量，寄存器变量，volatile 声明的变量都不会回滚到以前的值。对于优化后的编译，自动变量和寄存器变量会回滚到以前的值。笔者未进行过测试。


### 25.int getrlimit(int resource, struct rlimit *rlim)



### int setrlimit(int resource, const struct rlimit *rlim)

每个进程都有一组资源限制，这两个函数可以获取和设置每个资源限制。声明在 sys/resource.h 文件中。resource 代表资源限制的类型，struct rlimit 结构体中装有该资源的软限制值和硬限制值。该结构体如下：

```c
struct rlimit {
               rlim_t rlim_cur;  /* Soft limit */
               rlim_t rlim_max;  /* Hard limit (ceiling for rlim_cur) */
           };
```


- 任何一个进程都可将一个软限制值更改为小于或等于其硬限制值。
- 任何一个进程都可降低其硬限制值，但它必须大于或等于其软限制值，这种降低对普通用户而言是不可逆的。
- 只有超级用户进程可以提高硬限制值。




## 第 8 章 进程控制



### 1.pid_t getpid(void)



### pid_t getppid(void)



### uid_t getuid(void)



### uid_t geteuid(void)



### gid_t getgid(void)



### gid_t getegid(void)

这些函数用来返回当前进程的进程 ID，父进程 ID，实际用户 ID，有效用户 ID，实际组 ID，有效组 ID。它们均无出错返回。声明在 unistd.h 文件中。注意：当前进程的保存的设置用户 ID 无法返回。


### 2.pid_t fork(void)

该函数为调用进程创建一个子进程。声明在 unistd.h 文件中。由该函数创建的子进程和其父进程使用了” 写时复制 “技术。在父进程中，该函数返回一个正数；在子进程中，该函数返回 0；出错的话该函数返回 - 1。


### 子进程会继承父进程的以下属性或资源


- 实际用户 ID，实际组 ID，有效用户 ID，有效组 ID。
- 附加组 ID。  c. 进程组 ID。   d. 会话 ID。  e. 控制终端
- 设置用户 ID 标志和设置组 ID 标志。   g. 当前共组目录。
- 根目录     i. 文件模式创建屏蔽字。
- 信号屏蔽和安排。
- 针对任一打开文件描述符的在执行时关闭（close-on-exec）标志。
- 环境。  m. 链接的共享存储段。  n. 存储映射    o. 资源限制。
- 父进程所打开的文件描述符的文件表项（父子进程共享由父进程打开的文件）。




### 父子进程有如下区别


- fork 返回值不同。
- 进程 ID 不同。
- 两个进程具有不同的父进程 ID。
- 子进程的 tms_utime，tms_stime，tms_cutime，tms_ustime 均被设置为 0。
- 父进程设置的文件锁不会被子进程继承。
- 子进程未处理的闹钟（alarm）被清除。
- 子进程的未处理信号集设置为空集。




### fork 失败的两个主要原因


- 系统中的进程数太多。
- 该实际用户拥有的进程总数超过了系统限制。




### fork 有下面两种用法


- 父进程希望子进程复制父进程的代码，使得父，子进程执行不同的代码段。网络服务器进程一般这么用。
- 子进程执行其他的程序。子进程的 fork 返回后立即调用 exec。




### 3.pid_t vfork(void)

该函数和 fork 类似，也是用来为当前进程创建一个子进程。区别是：该函数不使用 “写时复制” 技术。也就是说，该函数创建的子进程目的就是要用 exec 执行一个新程序。子进程如果不执行 exec 函数，就永远运行在父进程的地址空间中，就算执行写操作，也不会复制父进程的地址空间到子进程。此外，vfork 会保证子进程先运行，子进程调用 exec 或 exit 之后父进程才可能被调度。（如果在调用者两个函数之前子进程依赖于父进程的进一步动作，则会导致死锁）


### 4.pid_t wait(int *status)



### pid_t waitpid(pid_t pid, int *status, int options)

在父进程中调用这两个函数来等待子进程的结束并收回子进程所占资源。声明在 sys/wait.h 文件中。二者区别是：wait 函数等待第一个结束的子进程，如果还没有子进程结束的话父进程会阻塞；waitpid 进程则可以等待由 pid 指定的进程，并且 option 提供了若干选项，可以选择该函数非阻塞。有若干个宏可以用来检测这两个函数返回的子进程状态 status，详见 man 手册。下面说下 waitpid 函数的 pid 和 options 参数的用法：

- pid == -1             等待任一子进程，相当于 wait 函数
- waitpid > 0          等待进程 ID 为 pid 的子进程
- pid == 0              等待进程组 ID 等于调用进程组 ID 的任一子进程
- pid < -1               等待进程组 ID 等于 pid 绝对值的任一子进程


options 选项：

- WCONTINUED          若系统支持作业控制，那么由 pid 指定的任一子进程在暂停后已经继续，但其状态尚未报告，返* 回其状态
- WNOHANG               若由 pid 指定的子进程还没有结束，则 waitpid 函数不阻塞，立即返回
- WUNTRACED           若系统支持作业控制，那么由 pid 指定的任一子进程处于暂停状态，并且自暂停状态以来还未报告过，返回其状态




### 5. int waitid(idtype_t idtype, id_t id, siginfo_t *infop, int options)

该函数和 waitpid 函数类似，也是等待指定 pid 的子进程返回。声明在 sys/wait.h 文件中。但区别是：该函数把要等待的子进程 ID 和子进程所在组 ID 区分开了，分别表示。infop 中包含了引起子进程状态改变的信号的详细信息。idtype 提供了要等待的 ID 类型：

- P_PID        等待进程号为 id 的子进程
- P_PGID     等待进程组号为 id 的子进程
- P_ALL       等待任一子进程，忽略 id


options 选项：

- WCONTINUED     同上
- WEXITED             等待已退出的进程
- WNOHANG          同上
- WNOWAIT　　     不破坏子进程的退出状态，该退出状态可有后续的 wait，waitid，waitpid 调用取得
- WSTOPPED　　  等待一个已暂停但是状态还未报告的子进程




### 6.pid_t wait3(int _status, int options, struct rusage _rusage);



### pid_t wait4(pid_t pid, int _status, int options, struct rusage _rusage)

这两个函数和上面函数类似，只是多了一个功能：rusage 可以收集到子进程的资源统计信息（用户 CPU 时间总量，系统 CPU 时间总量，页面出错次数，接收到信号的次数等）。声明在 sys/wait.h 文件中。


### 7.int execl(const char _path, const char _arg, ...)



### int execlp(const char _file, const char _arg, ...)



### int execle(const char _path, const char _arg,  ..., char * const envp[])



### int execv(const char _path, char _const argv[])



### int execvp(const char _file, char _const argv[])



### int execvpe(const char _file, char _const argv[], char *const envp[])

这些函数将调用进程所执行的程序替换为新程序，一般在子进程中使用。声明在 unistd.h 文件中。它们之间的区别：函数名中包含 p 的，以文件名（file）为参数提供可执行文件（若 file 串中包含 /，将它视为路径名；否则安 PATH 环境变量来找），函数名不包含 p 的，是以路径名（path）为参数来提供可执行文件；函数名中包含 l 的（前三个函数），命令行参数为可变参数（除了第 0 个参数，它已经用 arg 给出来了），其他函数是以指针数组形式提供命令行参数；函数名以 e 结尾的，最后一个参数是环境表，其他函数则没有环境表参数。
子进程中通过 exec 函数执行新程序之后，2 中所列出的 a-o 那些属性不会改变，外加 tms_utime，tms_stime，tms_cutime，tms_ustime 这些值也不会改变，还要注意实际用户 ID 和实际组 ID 是否改变根据新可执行文件的设置用户 ID 位和设置组 ID 位是否设置而定；p 的话，要根据所打开文件描述符是否设置 FD_CLOEXEC 标志（执行时关闭 close-on-exec）有关，若设置了就关闭，否则不关闭。


### 8.int setuid(uid_t uid)



### int setgid(gid_t gid)

这两个函数分别用来设置当前进程的实际用户 ID / 有效用户 ID 和实际组 ID / 有效组 ID。声明在 unistd.h 文件中。有如下设置规则：

- 如果进程具有超级用户权限，可将进程的实际用户 ID，有效用户 ID，保存的设置用户 ID 设置为 uid。
- 如果进程没有超级用户权限，但是 uid 等于进程的实际用户 ID 或保存的设置用户 ID，则只将有效用户 ID 设*uid，不改变实际用户 ID 和保存的设置用户 ID。
- 如果 a 和 b 都不满足，则将 error 设置为 EPERM，并返回 - 1。


还要注意以下三点：

- 只有超级用户进程可以更改实际用户 ID。
- 仅当对程序文件设置了设置用户 ID 位时，exec 函数才会设置有效用户 ID 为程序文件的拥护者 ID。否则 exec 函数不改变有效用户 ID，维持原先值。上边的 b 表明了任何时候都可调用 setuid 将有效用户 ID 设置为实际用户 ID 或保存的设置用户 ID。
- 保存的设置用户 ID 是有 exec 复制有效用户 ID 而得来的。




### 9.int setreuid(uid_t ruid, uid_t euid)



### int setregid(gid_t rgid, gid_t egid)

这两个函数用来交换实际用户 ID 和有效用户 ID，实际组 ID 和有效组 ID。声明在 unistd.h 文件中。任一参数值为 - 1，则相应的 ID 应当保持不变。


### 10.int seteuid(uid_t euid)



### int setegid(gid_t egid)

这两个函数类似于 8 中的函数，但是只改变当前进程的有效用户 ID，无论是当前是特权用户还是非特权用户。声明在 unistd.h 文件中。特权用户进程可将有效用户 ID 设置为 euid，非特权用户进程可将有效用户 ID 设置为实际用户 ID 或保存的设置用户 ID。
** 注意：上述所有的组 ID 设置和用户 ID 设置类似，不再赘述。附加组 ID 不受这些函数影响。**


### 11. 解释器文件

第一行包含 “#!pathname” 的文件就是解释器文件。shell 会创建一个子进程来执行 pathname 所指示的命令，然后该命令再去解释执行解释器文件。解释器文件不一定非得是 shell 脚本，也可能是其他脚本 (比如 awk 程序脚本)。每一种类型的解释器文件都会有自己的命令解释器。内核在执行 pathname 命令时，传递给该命令的参数如下：argv[0]：pathname 路径；argv[1]：解释器文件 #!pathname 后边跟的参数；argv[2]：exec 的第 0 个参数（解释器文件的绝对路径）；argv[3] 及以后的参数：exec 的第 2 个及以后的参数。
`eg：execl(“home/sar/bin/testinterp”, "testinterp", "myarg1", "MY ARG2", (char *)0)；`
其中 home/sar/bin/testinterp 是解释器文件。查看该解释器内容：cat home/sar/bin/testinterp ===>#!/home/sar/bin/echoarg foo。该解释器文件第一行包含了解释器命令为 / home/sar/bin/echoarg，foo 为该命令的参数。execl 将执行 / home/sar/bin/echoarg 程序，将 “/home/sar/bin/echoarg”，“foo”，“home/sar/bin/testinterp”，"myarg1"，"MY ARG2" 作为参数传递进去。


### 12.int system(const char *command)

该函数用来执行一个 shell 命令。声明在 stdlib.h 文件中。在该函数内部调用了 fork，exec，waitpid。command 字符串中包含了命令以及参数等（包括由管道连接的多个命令，以及输入输出重定向），shell 会对其进行语法分析，将其分成命令行参数。一般情况下，fork 和 exec 之间会有安全漏洞问题，需要在 fork 之后，exec 之前将子进程的有效用户改为普通用户，然后再让子进程执行 exec。该函数中 fork 出的子进程去执行 / bin/sh 程序，然后 shell 子进程再创建子进程去执行 command。由于该函数中调用了 fork，exec 和 waitpid，因此该函数的返回值较为特殊，需要注意，有三种返回值：

- 如果 fork 失败或者 waitpid 返回除 EINTR 之外的出错，则 system 返回 - 1，并且 errno 中设置了错误类型。
- 如果 exec 失败（表示不能执行 shell），则 system 返回值如同 shell 执行了 exit(127) 一样。
- 否则三个函数都执行成功，则 system 返回值是 shell 的终止状态。


注：该函数对信号的处理有要求。该函数在 fork 子进程前，会先屏蔽掉 SIG_INT 和 SIGQUIT 信号，同时屏蔽掉调用进程的 SIGCHLD 信号（防止用户在该信号处理函数中调用了 wait 等函数回收子进程，这样 system 函数中的 wait 函数就获取不到子进程的终止状态），函数返回时会恢复一切。


## 第 10 章 信号



### 1.typedef void (*sighandler_t)(int);



### sighandler_t signal(int signum, sighandler_t handler)

该函数用来设置进程的信号处理函数。一般不推荐使用该函数(**signal原始的系统调用只会生效一次，有的linux版本是用sigaction实现的，有些是直接调用原始系统调用，因此无法移植**)，而应该使用 sigaction 函数替代它。声明在 signal.h 文件中。signum 参数是要设置的信号名，handler 参数是一个函数指针变量，该变量也可以取以下三种值值：SIG_IGN，SIG_DFL，信号处理函数的地址。第一个表示忽略此信号，第二个表示采用系统的默认动作处理此信号，第三个表示此信号发生时，调用该处理函数来处理此信号。函数返回值是前一个信号处理函数的指针。一般情况下，执行一个程序时，如果没有设置信号的处理函数，则系统会把信号设置为默认的处理动作。当进程执行 exec 函数后，也会把之前设置的信号处理动作更改为信号的默认处理动作。当进程创建了一个子进程时，子进程会继承父进程的信号处理方式。
注：信号的三种处理方式：忽略，捕获，默认方式（默认方式一般是终止接收进程）。


### 2.int kill(pid_t pid, int sig)



### int raise(int sig)

第一个函数将新号发送给进程或者进程组。第二个函数向本进程发送信号。声明在 signal.h 文件中。第二个函数等价为 kill(getpid(),sig)。kill 函数的 pid 有以下四种情况：

- pid > 0          将该信号发送给进程 ID 为 pid 的进程
- pid == 0        将该信号发送给和发送进程属于同一个进程组的所有进程（不包括系统进程），而且发送进程具有向这些进程发送信号的权限。
- pid < 0          将信号发送给进程组 ID 等于 | pid | 的所有进程（不包括系统进程）。并且也要具有发送权限，类似于 b
- pid == -1       将信号发送给系统上的所有进程（不包括系统进程）。并且也要具有发送权限，类似于 b 和 c


上边提到的发送权限是这样的：超级用户可将信号发送给任意进程；非超级用户，是否有权限的依据是发送者的实际或者有效用户 ID 是否等于接收者的实际或者有效用户 ID，相等的话就具有权限，否则不具有权限。另外，普通进程可以将 SIGCONT 信号发送给同一个会话中的所有进程，包括系统进程。
如果 kill 的 sig 参数为 0，用来测试 pid 号进程是否存在，不存在的话 kill 返回 - 1 并设置 errno 为 ESRCH，这种测试一般没有什么价值。另外，kill 函数向本进程发送信号时，会在函数返回前将信号传递至该进程。


### 3.unsigned int alarm(unsigned int seconds)

该函数设置一个闹钟（计时器），当闹钟时间到之后会投递一个 SIGALRM 信号至调用进程。该信号的默认处理是终止调用进程。每个进程只能有一个闹钟时间，如果调用 alarm 函数设置了一个闹钟时间，在该时间到之前又调用 alarm 函数设置了一个闹钟时间，则第二次调用 alarm 函数返回值为第一次 alarm 调用设置时间的剩余值，然后按照第二次设置的时间重新计时；如果第二次 alarm 调用参数为 0，则取消闹钟时间，并将第一次 alarm 调用设置时间的剩余值返回。声明在 unistd.h 文件中。


### 4.int pause(void)

该函数将进程挂起直到捕捉到一个信号。声明在 unistd.h 文件中。当该挂起进程接收到一个信号，执行完信号处理函数并从其返回时，pause 函数才返回，返回值为 - 1，并将 errno 设置为 EINTR。


### 5.int sigemptyset(sigset_t *set)



### int sigfillset(sigset_t *set)



### int sigaddset(sigset_t *set, int signum)



### int sigdelset(sigset_t *set, int signum)



### int sigismember(const sigset_t *set, int signum)

这五个函数用来初始化信号集。声明在 signal.h 文件中。sigemptyset 函数会清空 set 所指向的信号集。sigfillset 函数会将所有信号放入 set 指向的信号集中。所有信号集在使用之前，都要调用这两个函数的其中之一来初始化信号集。一旦初始化了一个信号集，就可使用其余的函数在信号集中增加或删除特定的信号。
如果实现的信号数目少于一个整型所包含的位数，则可以用整型变量作为信号集，整型的每一位代表一个信号。此时，sigemptyset 函数将整型量设置为 0，sigfillset 函数将整型量的各个位都设置为 1。这两个函数被实现为宏函数：

```c
#define  sigemptyset(ptr)    {*(ptr) = 0}
#define  sigfillset(ptr)          {*(ptr) = ~(sigset_t)0, 0}
```

可看出，这两个宏函数返回值均为 0。
sigaddset 函数打开一位（将该位置为 1），sigdelset 函数关闭一位（将该位置为 0），sigismember 函数测试一个指定位。


### 6.int sigprocmask(int how, const sigset_t _set, sigset_t _oldset)

该函数设置进程的信号屏蔽字。声明在 signal.h 文件中。如果 oldset 参数非空，则进程的当前信号屏蔽字通过 oldset 返回；如果 set 参数非空，这时 how 指定当前信号屏蔽字的修改方法。how 的取值如下：

- SIG_BLOCK              该进程新的信号屏蔽字是其当前的信号屏蔽字和 set 指向信号集的并集。set 包含了我们希望阻塞的附加信号。
- SIG_UNBLOCK         该进程新的信号屏蔽字是其当前的信号屏蔽字和 set 指向信号集的交集。set 包含了我们希望解除阻塞的信号。
- SIG_SETMASK          该进程新的信号屏蔽字将被 set 指向的信号集所替代。


如果 set 为空，则不改变该进程的信号屏蔽字，how 值也无意义。另外，调用 sigprocmask 函数后如果有任何没被阻塞的未决信号，会在该函数返回前被递送给该进程。注意：sigprocmask 函数仅能在单线程的进程中使用。多线程中需要使用 pthread_sigmask 函数。


### 7.int sigpending(sigset_t *set)

该函数用来查看当前进程的未决信号（实际上就是被阻塞的信号）集。该信号集通过 set 参数返回。声明在 signal.h 文件中。


### 8. int sigaction(int signum, const struct sigaction _act, struct sigaction _oldact)

该函数类似于 signal 函数，用来查看或者设置信号的处理函数。声明在 signal.h 文件中。一般推荐使用该函数，去替代 signal 函数。参数 signum 是要查看或者修改的信号编号，若参数 act 非空，则要修改信号处理动作，若参数 oldact 非空，返回该信号的上一个处理动作。struct sigaction 结构体如下：

```c
struct sigaction {
               void     (*sa_handler)(int);
               void     (*sa_sigaction)(int, siginfo_t *, void *);
               sigset_t   sa_mask;
               int        sa_flags;
               void     (*sa_restorer)(void);
           };
```

sa_handler 字段指向了信号处理函数的地址；sa_mask 字段指定了一个信号集，调用上述信号处理函数之前会屏蔽掉该信号集，从信号处理函数中返回时恢复信号屏蔽字（这样就保证了在处理一个给定信号时，若这种信号再次发生，它将被阻塞到对前一个信号的处理结束为止，如果同一种信号发生多次，通常部将它们排队，也就是说当解除信号阻塞后，该信号只会被投递一次）。sa_flags 字段指定对信号进行处理的各个选项：

- SA_INTERRUPT        由此信号中断的系统调用不会自动重启。
- SA_NOCLDSTOP       若 signum 参数是 SIGCHLD，当子进程暂停（作业控制）时或者由暂停继续运行时，不产生此信号；当子进程终止时产生此信号。
- SA_NOCLDWAIT        若 signum 参数是 SIGCHLD，则当调用进程的子进程终止时，不产生将死进程；若调用进程依然 wait 子进程，则调用进程阻塞知道所有子进程都终止，才会返回，返回值为 - 1，并将 errno 设置为 ECHILD。
- SA_NODEFER            当捕捉到该信号并执行其信号处理函数时，系统不自动阻塞此信号（除非 sa_mask 中包括了此信号）。此种类型操作对应于早期的不可靠信号。
- SA_ONSTACK            若用 sigaltstack 函数声明了一个替换栈，则将此信号递送给替换栈上的进程。
- SA_RESETHAND        在此信号函数的入口处，将此信号的处理方式复位为 SIG_DFL，并清除 SA_SIGINFO 标志。此种类型操作对应于早期的不可靠信号。此标志不能自动复位 SIGILL 和 SIGTRAP 这个信号。设置了此标志也会有设置 SA_NODEFER 标志后的效果。
- SA_RESTART             由此信号中断的系统调用会自动重启动。
- SA_SIGINFO               此选项会用 sa_sigaction 指向的信号处理函数来处理此信号，而不用 sa_handler 所指向的信号处理函数。


sa_sigaction 指针指向的信号处理函数类型为：void  _ function(int, siginfo_t _, void *)。siginfo_t 结构体将包含信号产生原因的有关信息。


### 9.int sigsetjmp(sigjmp_buf env, int savesigs)



### void siglongjmp(sigjmp_buf env, int val)

如果要在信号处理函数中实现远跳转，应该使用这两个函数进行堆栈保存和恢复，而避免使用 setjmp 和 longjmp。因为执行信号处理程序时系统自动设置当前信号的屏蔽字，信号处理程序结束时系统自动恢复屏蔽字，而 setjmp 和 longjmp 函数不对屏蔽字进行任何操作，因此使用 longjmp 远跳转之后，当前信号的屏蔽字就无法恢复了（以后无法处理当前信号了）。因此需要使用 9 中这两个函数。sigsetjmp 函数比 setjmp 多出了一个 savesigs 参数，该参数非 0 时，该函数会保存当前进程的信号屏蔽字，然后 siglongjmp 函数会将保存的信号屏蔽字恢复，这样当从信号处理函数中远跳转后，进程的信号屏蔽字仍然会恢复到以前的状态。声明在 setjmp.h 文件中。


### 10.int sigsuspend(const sigset_t *mask)

进程中调用该函数用来等待一个信号。声明在 signal.h 文件中。mask 参数用来设置进程的信号屏蔽字，也就是说，该函数将等待除了被屏蔽掉的信号以外的任意一个信号到来，在这一信号到来之前当前进程处于阻塞状态。信号到来之后并执行完信号处理函数后，才从该函数返回。该函数返回时，会自动将信号屏蔽字恢复为该函数被调用之前的状态。该函数的返回值总是 - 1，并将 errno 设置为 EINTR，表示一个被中断的系统调用。


### 11.void abort(void)

该函数发送 SIGABRT 信号至当前进程，使当前进程非正常终止。无论程序中在调用 abort 之前，对 SIGABRT 信号的处理进行何种设置，abort 都会终止该进程（abort 内部会重新将 SIGABRT 信号的处理设置为默认方式）。声明在 stdlib.h 文件中。


### 12.unsigned int sleep(unsigned int seconds)

该函数使调用线程睡眠 seconds 秒。该函数一般使用 alarm 实现，因此该函数和用户调用的 alarm 的之间有交互作用，需要注意。该函数是调用线程睡眠，并不是整个进程睡眠，当然，如果进程只有一个主线程，那么主线程睡眠就相当于进程睡眠。
还有nanosleep和clock_nanosleep提供纳秒级的参数。
注意：信号这一章节有个重要的知识，就是中断的系统调用。一般情况下，系统调用将进程阻塞后，如果进程收到了任意可以处理的信号，就会激活进程，若设置了该信号的处理函数，则去执行该处理函数并且返回后也会从该引起进程阻塞的系统调用处返回。也有一部分系统调用会重新启动。


### 13. 与作业控制有关的信号：


- SIGCHLD         子进程已暂停或终止
- SIGCONT         使暂停的进程继续运行
- SIGSTOP          使进程暂停（不能捕获或忽略）
- SIGTSTP           交互式暂停信号（用终端命令使进程暂停，比如 ctrl+z）
- SIGTTIN            后台进程组成员读控制终端
- SIGTTOU          后台进程组成员读控制终端


除了 SIGCHLD 以外，其它信号应用程序不去处理。当按 ctrl+z 时，SIGTSTP 信号被送至前台进程组的所有进程，使之暂停（并转入后台）。在命令行下使用 fg %n 或 bg %n 时，SIGCONT 信号被发给相应进程，使之转向前台继续运行或者在后台继续运行。对一个进程产生四种停止信号（SIGTSTP，SIGSTOP，STGTTIN，SIGTTOU）中的任意一种时，对该进程的任一未决 SIGCONT 信号就会被丢弃。类似，当对一个进程产生 SIGCONT 信号时，对同一进程的任一未决的停止信号将被丢弃。当对一个暂停的进程产生 SIGCONT 信号时，无论 SIGCONT 信号是被阻塞或是忽略，进程都将继续运行。


### 14. void psignal(int sig, const char *s)

该函数将字符串 s 输出到标准出错文件，后接一个冒号和空格，再接着是对 sig 信号的说明，最后是换行符。声明在 signal.h 文件中。类似于 perror 函数。


### 15.char *strsignal(int sig)

给出一个信号 sig，函数会返回说明该信号的字符串。应用程序可用该字符串打印关于接收到信号的出错消息。声明在 string.h 文件中。类似于 strerror 函数。


### 16. int sigqueue(pid_t pid, int sig, const union sigval value)

向单个进程发送排队信号。排队信号需要如下步骤才能使用：

- 使用sigaction函数注册信号处理函数，指定SA_SIGINFO标志
- 使用sa_sigaction指定信号处理函数
- 使用sigqueue函数发送信号




## 第 11 章 线程概念



### 1.int pthread_equal(pthread_t t1, pthread_t t2)

该函数用来比较两个线程 IDti 和 t2 是否相等。相等返回非 0 值，不相等返回 0。声明在 pthread.h 文件中。由于每个平台的 pthread_t 类型不一定相同，因此不能直接比较，只能通过该函数比较。


### 2.pthread_t pthread_self(void)

该函数获取当前线程的线程 ID。声明在 pthread.h 文件中。1，2 函数一般可以配合来使用。


### 3.int pthread_create(pthread_t _thread, const pthread_attr_t _attr, void _(_start_routine) (void _), void _arg)

该函数用来创建一个新线程。声明在 pthread.h 文件中。thread 中包含了新线程的线程 ID。attr 结构可以为新线程设置线程属性。start_routine 是线程的启动例程（线程要执行的函数）。arg 为线程传递参数，一般是结构类型指针。线程创建后并不保证哪个线程先运行，所以不能做任何假设。新线程继承了调用线程的浮点环境和信号屏蔽字，新线程的未决信号集被清除。并且每个线程都有 errno 的副本，当线程出错后，错误码会保存在各自的 errno 中。新创建的线程一般通过 2 中函数来获取自己的线程 ID，而不能使用 thread 参数来获取，因为新线程可能在创建函数返回前就运行了，而此时也许 thread 还未设置成线程 ID。其他平台同一个进程中的多个线程，它们的进程号是相同的，线程号是不同的；但是 Linux 系统，由于它的线程和进程实现机制是一样的，都用 clone 系统调用来完成创建，因此 Linux 系统中的线程实际上是其创建进程的子进程，该子进程可以共享父进程一定数量的执行环境。


### 4.void pthread_exit(void *retval)

该函数用来结束调用线程。声明在 pthread.h 文件中。参数 retval 是线程的返回码。其他线程可以通过 pthread_join 函数获得该返回码。线程的三种退出方式如下：

- 线程从启动例程中返回，返回值是线程的退出码
- 线程可以被同一进程中的其他线程取消
- 线程调用 pthread_exit 返回




### 5.int pthread_join(pthread_t thread, void* retval)

该函数等待一个线程 ID 号为 thread 的线程结束。类似于 waitpid。该函数会一直阻塞直到等待的线程调用 pthread_exit，或从启动例程返回，或者被其他线程取消。声明在 pthread.h 文件中。该函数可以获得 4 中函数的返回码，保存到 retval 指向的单元，如果对等待线程的返回码不感兴趣，可将 retval 置为 NULL，则该函数不会获取等待线程的返回码。如果等待线程是被其他线程取消的，则 retval 指向的内存单元被置为 PTHREAD_CANCELED。
4，5 两个函数的 retval 参数可以传递结构体指针，来包含更多的数值。该结构体一般要用全局结构或者是 malloc 分配的单元。


### 6.int pthread_cancel(pthread_t thread)

线程通过调用该函数来取消同一进程中的其他线程。声明在 pthread.h 文件中。默认情况下，被取消的线程会表现得跟调用了 pthread_exit(PTHREAD_CANCELED) 函数一样。线程可以选择忽略取消方式或者控制取消方式。此外，pthread_cancel 并不等待线程终止，它仅提出请求。


### 7.void pthread_cleanup_push(void (_routine)(void _)



### void pthread_cleanup_pop(int execute)

第一个函数用来注册线程结束时要执行的清理函数。类似于进程中的 atexit 函数。清理函数的执行顺序与注册顺序相反。这两个函数需要配对使用，调用了几次 pthread_cleanup_push 函数，也需要对应的调用几次 pthread_cleanup_pop 函数。它们均声明在 pthread.h 文件中。当线程执行以下动作时会调用清理函数：

- 调用 pthread_exit 时
- 响应取消请求(**pthread_cancel的请求**)时
- 用非 0 的 execute 参数赖掉用 pthread_cleanup_pop 时


当使用参数 execute=0 来调用 pthread_cleanup_pop 函数后，无论在上边那种情况下，清理函数都不会被调用，而会被清理掉。此外，还要注意，当线程从它的启动例程中返回的话（比如使用 return），即使注册了清理函数，清理函数也不会被调用。


### 8.int pthread_detach(pthread_t thread)

该函数用来分离一个线程。声明在 pthread.h 文件中。一个线程被分离后，它的底层存储资源在线程终止时立即被收回。而不用等待其他线程调用 pthread_join 函数或者整个进程结束。注意，一个线程分离后，不能再调用 pthread_join 函数来等待该分离线程。


### 9. 线程同步



#### a. 互斥量


```c
int pthread_mutex_init(pthread_mutex_t *restrict mutex, const pthread_mutexattr_t *restrict attr)
int pthread_mutex_destroy(pthread_mutex_t *mutex)
```

这两个函数用来初始化互斥量（动态初始化）和清理互斥量资源。声明在 pthread.h 文件中。如果想静态初始化互斥量，定义一个全局的 pthread_mutex_t 类型变量，并给其赋值为 PTHREAD_MUTEX_INITIALIZER。如果互斥量单元是用 malloc 等进行分配的，那么在释放内存之前需要调用 pthread_mutex_destroy 对资源进行清理。用默认属性初始化互斥量时只需把 attr 置为 NULL。

```c
int pthread_mutex_lock(pthread_mutex_t *mutex)
int pthread_mutex_trylock(pthread_mutex_t *mutex)
int pthread_mutex_unlock(pthread_mutex_t *mutex)
```

使用第一个函数对互斥量进行加锁，使用第三个函数对互斥量解锁。它们均声明在 pthread.h 文件中。如果互斥量已经上锁，那么调用进程将阻塞直到互斥量被解锁。第二个函数也是用来对互斥量进行加锁，如果互斥量已经上锁，它不会阻塞进程，返回 EBUSY，如果成功加锁则返回 0。
带有超时机制的加锁函数(注意超时是**绝对时间**)：

```c
int pthread_mutex_timedlock(pthread_mutex_t *restrict mutex,
              const struct timespec *restrict abs_timeout);
```



#### b. 读写锁


```c
int pthread_rwlock_init(pthread_rwlock_t *restrict rwlock, const pthread_rwlockattr_t *restrict attr)
int pthread_rwlock_destroy(pthread_rwlock_t *rwlock)
```

这两个函数用来初始化读写锁和清理读写锁。它们和 a 中的函数类似。声明在 pthread.h 文件中。如果希望读写锁有默认属性，将 attr 置为 NULL。在释放动态分配的内存前，需要先调用第二个函数释放资源。

```c
int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock)
int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock)
int pthread_rwlock_unlock(pthread_rwlock_t *rwlock)
```

这几个函数分别在读模式下或者写模式下锁定读写锁，无论哪种模式下锁定读写锁，都可用第三个函数来解锁。它们均声明在 pthread.h 文件中。读写锁实现的时候可能会对共享模式下可获取的锁数量进行限制，所以需要检查 pthread_rwlock_rdlock 函数返回值。

```c
int pthread_rwlock_tryrdlock(pthread_rwlock_t *rwlock)
int pthread_rwlock_trywrlock(pthread_rwlock_t *rwlock)
```

这两个函数也类似于 a 中的函数，获得锁时返回 0，锁被占时返回 EBUSY，也不会阻塞线程。声明在 pthread.h 文件中。
带有超时机制的加锁函数(注意超时是**绝对时间**)：

```c
int pthread_rwlock_timedrdlock(pthread_rwlock_t *restrict rwlock,
              const struct timespec *restrict abs_timeout);
int pthread_rwlock_timedwrlock(pthread_rwlock_t *restrict rwlock,
              const struct timespec *restrict abs_timeout);
```



#### c. 条件变量


```c
int pthread_cond_init(pthread_cond_t *restrict cond, const pthread_condattr_t *restrict attr)
int pthread_cond_destroy(pthread_cond_t *cond)
```

这两个函数和上边的函数是类似的。声明在 pthread.h 文件中。第一个函数用来对条件变量进行初始化。第二个函数用来对条件变量进行资源清理。如果条件变量是由 malloc 等手动分配的，那么在释放该空间前，要先调用第二个函数进行资源清理。如果使用默认属性进行初始化，attr 传递 NULL 即可。

```c
int pthread_cond_wait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex)
int pthread_cond_timedwait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex, const struct timespec *restrict abstime)
```

这两个函数用来等待一个条件变量 cond 为真。声明在 pthread.h 文件中。如果 cond 不为真，会将调用线程放到等待条件的线程列表上，并阻塞当前线程，释放互斥锁 mutex。第二个函数仅仅等待 abstime 时长，若 cond 还不为真则返回错误码，同时获取互斥锁。这两个函数的条件变量 cond 均受到互斥锁 mutex 的保护，当函数将线程放入等待条件列表上后，会释放该互斥锁。该互斥锁 mutex 将会保护两个资源：一个是消息队列，另一个是条件变量 cond。具体的使用例子参考《apue》。第二个函数的 struct timespec 参数类型如下：

```c
struct timespec {
　　time_t       tv_sec,         /*seconds*/
　　long          tm_nsec;     /*nanoseconds*/
};
```

第二个函数的等待时间是个绝对数而不是相对数，也就是说，如果需要等待 3 分钟，就要在当前时间上加 3 分钟然后转换到 struct timespec 结构中。

```c
int pthread_cond_signal(pthread_cond_t *cond)
int pthread_cond_broadcast(pthread_cond_t *cond)
```

这两个函数用于通知线程条件已经满足。第一个函数将唤醒等待该条件的某个线程，第二个函数会唤醒等待该条件的所有线程。


#### d. 自旋锁


```c
int pthread_spin_destroy(pthread_spinlock_t *lock);
int pthread_spin_init(pthread_spinlock_t *lock, int pshared);

int pthread_spin_lock(pthread_spinlock_t *lock);
int pthread_spin_trylock(pthread_spinlock_t *lock);
int pthread_spin_unlock(pthread_spinlock_t *lock);
```



#### e. 屏障

屏障允许每个线程等待，直到所有的合作线程都达到某一点，然后该点继续执行。**pthread_join**就是一种屏障，允许一个线程等待，直到另一个线程退出。

```c
int pthread_barrier_destroy(pthread_barrier_t *barrier);
int pthread_barrier_init(pthread_barrier_t *restrict barrier, const pthread_barrierattr_t *restrict attr, unsigned count);  //count为必须达到屏障的线程数

int pthread_barrier_wait(pthread_barrier_t *barrier);
```



## 第 12 章 线程控制



### 1.int pthread_attr_init(pthread_attr_t *attr)



### int pthread_attr_destroy(pthread_attr_t *attr)

这两个函数用于对线程的属性结构体进行初始化和清理。声明在 pthread.h 文件中。Linux 系统支持的线程属性有下边四个：
detachstate       线程的分离属性guardsize          线程栈末尾的警戒缓冲区大小（字节数）stackaddr          线程栈的最低地址stacksize           线程栈的大小（字节数）
可取消状态可取消类型并发度


### 2.int pthread_attr_getdetachstate(pthread_attr_t _attr, int _detachstate)



### int pthread_attr_setdetachstate(pthread_attr_t *attr, int detachstate)

这两个函数用来获取和设置线程的分离属性。声明在 pthread.h 文件中。detachstate 参数代表了属性值，有两个：PTHREAD_CREATE_DETACHED（以分离状态启动线程），PTHREAD_CREATE_JOINABLE（正常启动线程）。如果一开始就知道不需要获取线程的返回码，那就可以分离属性来创建线程，这样线程就会以分离状态来启动，线程结束时系统就会收回线程资源。


### 3.int pthread_attr_getstack(pthread_attr_t *attr, void ** stackaddr, size_t **stacksize)



### int pthread_attr_setstack(pthread_attr_t _attr,  void _stackaddr, size_t stacksize)

这两个函数用来获取和设置线程的线程栈地址属性和线程栈大小属性。声明在 pthread.h 文件中。一个进程中的所有线程共用该进程的栈地址空间，当线程数太多时，可能会消耗完进程的栈地址空间，于是就需要使用 malloc 或者 mmap 为线程栈重新分配空间。分配好之后需要用第二个函数来设置栈。stackaddr 是线程栈的最低地址，因此它可能是线程的开始或者结尾（栈从高地址向低地址方向伸展）。


### 4.int pthread_attr_getstacksize(pthread_attr_t _attr, size_t _stacksize)



### int pthread_attr_setstacksize(pthread_attr_t *attr, size_t stacksize)

这两个函数用来专门获取和设置线程栈的大小属性。声明在 pthread.h 文件中。如果希望改变线程栈的默认大小，但又不想自己处理栈的分配问题，就可以用该函数。


### 5.int pthread_attr_getguardsize(pthread_attr_t _attr, size_t _guardsize)



### int pthread_attr_setguardsize(pthread_attr_t *attr, size_t guardsize)

这两个函数用来获取和设置线程栈的警戒缓冲区属性。声明在 pthread.h 文件中。参数 guardsize 控制着线程栈末尾之后用意避免栈溢出的扩展内存（警戒缓冲区）的大小。该缓冲区默认大小是 PAGESIZE 字节。当把 guardsize 置为 0 时，则不为线程栈提供警戒缓冲区。另外，如果对线程属性 stackaddr 做了修改，也会使线程栈缓冲区机制无效，等同于把 guardsize 置为 0。当线程的栈指针溢出到警戒区域，应用程序就会收到出错信号。


### 6.int pthread_getconcurrency(void)



### int pthread_setconcurrency(int new_level)

这两个函数用来获取和设置线程的并发度。声明在 pthread.h 文件中。如果系统当前正控制着并发度，那么第一个函数会返回 0。用第二个设置函数设置并发度时，只是对系统做一个提示，系统并不保证一定采用 new_level 参数指定的并发度。使用 new_level 参数为 0 来调用设置函数，会撤消在这之前用非 0 的 new_level 参数设置的并发度。


### 7. 同步属性



#### a. 互斥量属性


```c
int pthread_mutexattr_init(pthread_mutexattr_t *attr)
int pthread_mutexattr_destroy(pthread_mutexattr_t *attr)
```

这两个函数用来初始化和清理互斥量的属性。声明在 pthread.h 文件中。第一个函数会用默认的互斥量属性初始化 pthread_mutexattr_t 结构。有两个互斥量属性值得注意：互斥量共享属性和类型属性。

```c
int pthread_mutexattr_getpshared(const pthread_mutexattr_t * restrict attr, int *restrict pshared)
int pthread_mutexattr_setpshared(pthread_mutexattr_t *attr, int pshared)
```

这两个函数用来获取和设置互斥量的共享属性。在一个进程中，多个线程默认可以访问同一个同步对象（指互斥量，读写变量，条件变量），因此线程共用的同步对象属性需要设置为 PTHREAD_PROCESS_PRIVATE，这也是默认的属性值；而多个进程可以把同一个内存区映射到它们各自独立的地址空间中，多个进程访问同一个内存区的数据时也需要同步，那么在共享内存区域中分配的互斥量就可用于这些进程的同步，这时需要将互斥量的共享属性设置为 PTHREAD_PROCESS_SHARED。

```c
int pthread_mutexattr_gettype(const pthread_mutexattr_t *restrict attr, int *restrict type)
int pthread_mutexattr_settype(pthread_mutexattr_t *attr, int type)
```

这两个函数用来获取和设置互斥量的类型属性。互斥量类型属性值如下：
![](imgclip.png)
在一个函数的加锁，解锁调用之间需要调用另外的函数，而另外的函数内部也要获取同一个锁，这时就需要设置递归锁属性，否则就会死锁。此外，使用递归锁时，需要一定的编程技巧。


#### b. 读写锁属性


```c
int pthread_rwlockattr_init(pthread_rwlockattr_t *attr)
int pthread_rwlockattr_destroy(pthread_rwlockattr_t *attr)
```

这两个函数用来初始化和清理读写锁的属性。声明在 pthread.h 文件中。类似于互斥量的该函数。

```c
int pthread_rwlockattr_getpshared(const pthread_rwlockattr_t *  restrict attr, int *restrict pshared)
int pthread_rwlockattr_setpshared(pthread_rwlockattr_t *attr, int pshared)
```

这两个函数用来获取和设置读写锁的共享属性。声明在 pthread.h 文件中。类似于互斥量的该函数。读写锁仅支持共享属性，没有类型属性。


#### c. 条件变量属性


```c
int pthread_condattr_init(pthread_condattr_t *attr)
int pthread_condattr_destroy(pthread_condattr_t *attr)
```

这两个函数用来初始化和清理条件变量的属性。声明在 pthread.h 文件中。类似于互斥量和读写锁的该函数。

```c
int pthread_condattr_getpshared(const pthread_condattr_t *restrict attr, int *restrict pshared)
int pthread_condattr_setpshared(pthread_condattr_t *attr, int pshared)
```

这两个函数用来获取和设置条件变量的共享属性。声明在 pthread.h 文件中。类似于互斥量和读写锁的该函数。条件变量仅支持共享属性，没有类型属性。

```c
int pthread_condattr_getclock(const pthread_condattr_t *restrict attr,
              clockid_t *restrict clock_id);
int pthread_condattr_setclock(pthread_condattr_t *attr,
              clockid_t clock_id);
```

这两个函数控制计算pthread_cond_timedwait函数的超时时采用的哪个时钟。


#### d. 屏障属性


```c
int pthread_barrierattr_destroy(pthread_barrierattr_t *attr);
int pthread_barrierattr_init(pthread_barrierattr_t *attr);
```

这两个函数用来初始化和清理屏障的属性。声明在 pthread.h 文件中。类似于互斥量的该函数。

```c
int pthread_barrierattr_getpshared(const pthread_barrierattr_t *
              restrict attr, int *restrict pshared);
int pthread_barrierattr_setpshared(pthread_barrierattr_t *attr,
              int pshared);
```

这两个函数用来获取和设置条件变量的共享属性。声明在 pthread.h 文件中。类似于互斥量和读写锁的该函数。屏障仅支持共享属性。


### 8.void flockfile(FILE *filehandle)



### int ftrylockfile(FILE *filehandle)



### void funlockfile(FILE *filehandle)

这几个函数用来为 FILE 对象加锁和解锁。声明在 stdio.h 文件中。其中第二个函数不会阻塞当前线程。标准 IO 函数都是线程可重入的函数（所谓一个函数是可重入的，指的是该函数可以被并发的调用），因为它们会对标准 IO 流 FILE 进行加锁访问。并不是说标准 IO 函数内部调用了这几个函数，而是表现出来的特性就像是调用了这几个函数。这几个函数主要是留给应用程序来调用，对没有加锁的 FILE 流进行加锁保护。、
注：可重入函数一般分为对线程可重入（线程安全）或者对信号处理程序可重入（异步 - 信号安全），前者的话就是上边的定义，后者的话指的是在该函数中发生信号后函数仍然是安全的（执行完信号处理函数，返回到该函数中）。


### 9.int getc_unlocked(FILE *stream)



### int getchar_unlocked(void)



### int putc_unlocked(int c, FILE *stream)



### int putchar_unlocked(int c)

这几个函数是标准 IO 函数的不加锁版本。声明在 stdio.h 文件中。在使用这几个函数的时候就需要用到 8 中的函数。


### 10.int pthread_key_create(pthread_key_t _key, void (_destructor)(void*))

在分配线程的私有数据之前，需要调用该函数创建与该数据关联的键。这个键用于获取对线程私有数据的访问权。声明在 pthread.h 文件中。该函数的一般用法：在主线程中调用完成键的创建，然后在其他线程中使用该键去绑定私有数据即可（因此键的变量要定义为全局变量）。创建好的键存放在 key 所指向的内存单元中，这个键可以被进程中的所有线程使用，而每个线程把这个键与不同的线程私有数据地址进行关联。创建新键时，每个线程的数据地址设为 NULL。destructor 参数用来为该键关联析构函数，当线程正常退出时，如果传递的 destructor 参数非 NULL，就会调用该析构函数。而当线程调用 exit，_exit，_Exit 函数或者以非正常方式退出时，就不会调用该析构函数。通常会使用 mallocl 为线程私有数据分配内存空间，在析构函数中释放这些空间，否则，就可能使线程所属的进程出现内存泄漏。线程可以为私有数据分配多个键，每个键的析构函数可以相同也可以不同。当所有析构函数都调用完后，系统会检查是否还有非 NULL 的线程私有数据与键关联，有的话继续执行相应析构函数，直到所有私有数据为 NULL。


### 11.int pthread_key_delete(pthread_key_t key)

该函数用来取消键与私有数据值之间的关联关系。声明在 pthread.h 文件中。该函数在释放键时，并不会激活键的析构函数，因此需要在程序中手动释放由 malloc 函数分配的私有数据空间。


### 12.pthread_once_t once_control = PTHREAD_ONCE_INIT;



### int pthread_once(pthread_once_t _once_control, void (_init_routine)(void))

当多个线程中同时调用 10 中函数创建键的时候，很可能会发生冲突（其实键只需要被创建一次就行）。该函数用来避免此冲突。因为该函数只要在线程中被调用一次后，init_routine 参数指向的函数将只会被调用一次，因此在多个线程中都调用一次 pthread_once 函数，系统就能保证 init_routine 函数只会被调用一次，然后再在 init_routine 函数中调用 10 中的函数来完成键的创建，这样多个线程之间就不会发生冲突。另外，该函数的 once_control 参数必须是全局或者静态变量，而且被初始化为 PTHREAD_ONCE_INIT。具体例子可以参考《apue》。


### 13.void *pthread_getspecific(pthread_key_t key)



### int pthread_setspecific(pthread_key_t key, const void *value)

这两个函数用来用来获取和设置 key 键所对应的线程私有数据。声明在 pthread.h 文件中。如果 key 还没有对应的私有数据，第一个函数返回 NULL，否则返回私有数据空间地址，第二个函数可以根据第一个函数的返回值进行判断，进而选择是否要为键设置私有数据空间。另外，只有键所对应的私有数据非空时，析构函数才会被调用。


### 14.int pthread_setcancelstate(int state, int *oldstate)

该函数用来设置当前线程的可取消状态。声明在 pthread.h 文件中。该函数把当前的可取消状态设置为 state，把原来的可取消状态保存到 oldstate 指向的单元中。pthread_cancel 调用并不等待线程终止，该调用执行后，被取消线程继续运行，当运行到某个取消点时，才被检查自己是否要被取消。《apue》书中列举了 POSIX.1 所定义的取消点函数。当线程长时间都执行不到这些取消点函数时，可以调用 15 中的函数，它是专门的取消点函数。线程的可取消状态 state 有两个值：PTHREAD_CANCEL_ENABLE 和 PTHREAD_CANCEL_DISABLE，线程启动时默认取值是前者。如果线程的可取消状态值设为前者，则线程在取消点可以被取消；否则设为后者的话，在取消点也不会杀死线程，这时，取消请求对该线程而言处于未决状态，当该线程的可取消状态再次被设为 PTHREAD_CANCEL_ENABLE 时，该线程将在下一个取消点上对所有的未决请求进行处理。


### 15.void pthread_testcancel(void)

该函数用来为当前线程添加取消点。声明在 pthread.h 文件中。如果当前线程的可取消状态被设为 PTHREAD_CANCEL_DISABLE，则该函数没有任何效果。


### 16.int pthread_setcanceltype(int type, int *oldtype)

该函数用来用当前线程设置可取消类型。声明在 pthread.h 文件中。type 是新类型，oldtype 存放的是之前的旧类型。线程的可取消类型值有两个：PTHREAD_CANCEL_DEFERRED 和 PTHREAD_CANCEL_ASYNCHRONOUS。默认情况下，线程的可取消类型是前者，即延迟取消（就是执行完 pthread_cancel 调用后，被请求线程并不马上被取消）。后者是异步取消，也就是说线程可以在任意时间取消，而不是非得遇到取消点才能被取消。


### 17.int pthread_sigmask(int how, const sigset_t _set, sigset_t _oldset)

该信号用来屏蔽当前线程的某些信号。声明在 pthread.h 文件中。信号是属于一个进程的，因此可以被进程的所有线程共享，一般情况下，信号到达进程时，哪个线程正在运行，就在该线程的上下文中执行信号处理函数。但是，每个线程都有自己的信号屏蔽字，也就是说每个线程都可以屏蔽自己不想处理的信号，但这不影响其他线程的信号屏蔽字。另外，由于信号是属于进程的，所以任何线程中设置或修改了信号的处理函数，整个进程的信号处理函数也就随之改变了。


### 18.int sigwait(const sigset_t _set, int _sig)

线程通过调用该函数等待一个或多个信号发生。声明在 pthread.h 文件中。函数的返回值存放在 sig 指向的变量中，表明信号发送的数量。如果信号集 set 中的某个信号在 sigwait 调用的时候处于未决状态，那么 sigwait 函数将无阻塞的返回，返回前 sigwait 将从进程中移除那些处于未决状态的信号。线程在调用 sigwait 函数之时，必须阻塞它所等待的信号，从而避免错误发生（在函数返回之前的时间内，又会有新的等待信号产生）。sigwait 函数在返回前会取消信号集的阻塞状态（即恢复线程的屏蔽字）。
使用该函数使得线程简化了信号处理，将异步产生的信号用同步的方式来处理，说白了就是当信号产生时，不去执行进程的信号处理函数，而是被等待信号的线程所拦截（通过 sigwait 函数）而从 sigwait 函数返回，这样就相当于将等待信号的线程环境（线程中 sigwait 以后的代码）作为了信号处理程序，而不是进程的信号处理函数。当有多个线程调用 sigwait 函数来等待同一个信号，信号发生时只会有一个线程从 sigwait 返回。这块讨论的信号到底是由线程的 sigwait 捕获还是进程的信号处理函数捕获，是由操作系统实现的，操作系统要么实现前者，要么实现后者，而不会二者皆可。


### 19.int pthread_kill(pthread_t thread, int sig)

线程通过调用该函数向其他线程发送信号，thread 参数是其他线程的线程 id，sig 是信号名。声明在 pthread.h 文件中。当传递 sig=0 的参数到函数中时，可以用来检查 thread 线程是否存在。另外，如果信号的默认处理动作是终止进程，那么把信号传递给某个线程也仍然会杀死整个进程。


### 20.int pthread_atfork(void (_prepare)(void), void (_parent)(void), void (*child)(void))

该函数用于清除子进程中的锁状态，在fork之前注册三个处理函数。声明在 pthread.h 文件中。当在一个线程中使用 fork 创建子进程时，子进程通过继承父进程的整个地址空间，从而也继承了父进程的所有互斥量，读写锁和条件变量状态。但是，在子进程中只存在一个线程，那就是父进程中调用 fork 的那个线程的副本。由于子进程不包含其他线程副本，于是子进程就无法感知到其他线程中的锁，因此需要借助该函数来清理锁状态。如果子进程创建好后，立即调用了 exec 函数，就不需要清理那些锁了，因为整个旧的地址空间会被抛弃。
该函数的作用是来帮助 fork 后的子进程把其所有继承而来的锁打开。该函数的原理：该函数在 fork 函数之前进行调用，来注册三个处理函数 prepare，parent 和 child。注册成功后:

- prepare 函数会在 fork 函数创建子进程之前调用，作用是获取父进程定义的所有锁；
- parent 会在 fork 创建完子进程，将要返回父进程环境时调用，作用是打开父进程的所有锁；
- child 函数会在 fork 创建完子进程，将要返回子进程环境时调用，作用是打开子进程的所有锁。


至此，子进程中所有继承而来的锁全部被打开了。注意：如果不想用哪个处理函数，注册时该函数的参数传递为 NULL。该函数可以在 fork 之前多次调用，或者在多个模块中调用，详细例子参考《apue》。


## 第13章 守护进程



### 1. 守护进程编程规则


1. 将文件模式屏蔽字设置为0
2. 调用dork，然后关闭父进程
3. 调用**setsId()**创建新会话，使刚才创建的子进程变为进程组组长
4. 将当前工作目录设置为根目录
5. 关闭所有文件描述符
6. 将标准IO指向/dev/null
7. 初始化syslog，用于出错记录




### 2. syslog产生日志方法

![](imgclip_1.png)


#### 2.1 void openlog(const char *ident, int option, int facility)



#### void closelog(void);

打开log接口，ident一般是模块或程序名，options为选项，facility指定以什么方式处理日志消息。closelog为关闭用于与syslogd进程通信的描述符。


#### 2.2 void syslog(int level, const char *format, ...)

产生一个日志消息，消息真正的priority是facility和level的组合。format中每个出现的**%m**都会被替换成errno对应的消息字符串，不需要手动转换。


#### 2.3 int setlogmask(int mask)

设置进程的几率优先级屏蔽字，返回之前的old屏蔽字。各条消息除非在屏蔽字中进行了设置，否不不被记录。


## 第 14 章 高级 I/O



### 记录锁

功能：当第一个进程正在读或修改文件的某个部分时，使用记录锁可以组织其他进程修改同一文件区域。


#### fcntl函数设置记录锁

cmd为G_GETLK，G_SETLK，F_SETLKW。第三个参数为指向flock结构的指针：

```c
struct flock {
               ...
               short l_type;    /* 锁类型: F_RDLCK（共享读锁）, F_WRLCK（独占性写锁）, F_UNLCK（解锁一个区域） */
               short l_whence;  /* 加解锁的区域起始位置: SEEK_SET, SEEK_CUR, SEEK_END */
               off_t l_start;   /* 加解锁区域的起始偏移量 */
               off_t l_len;     /* 区域字节长度，为0表示可以扩展到的最大范围*/
               pid_t l_pid;     /* 当前锁定该区域的进程id (set by F_GETLK and F_OFD_GETLK) *，/
}
```

**Note**：自身持有的锁无法通过l_pid参数和G_GETLK命令获取，因为调用进程不会阻塞在自己持有的锁上。


### 1.int select(int nfds, fd_set _readfds, fd_set _writefds, fd_set _exceptfds, struct timeval _timeout)

该函数具有进行同步 I/O 多路转接功能（确定多个文件描述符 fd 的状态）。声明在 sys/select.h 文件中。nfns 参数告诉内核需要确定的描述符范围为 0～nfds-1，内核将来只会返回该范围的文件描述符状态。readfds，writefds，excpetfds 三个参数用来设置需要了解其状态的描述符，同时用来返回已准备好的可读，可写，有异常消息的描述符。timeout 参数用来设置 select 的阻塞时间。
先来说明最后一个参数，它指定了 select 函数愿意等待的时间，struct timeval 结构如下：

```c
struct timeval {
               long    tv_sec;         /* seconds */
               long    tv_usec;        /* microseconds */
           };
```

当 timeout == NULL 时，select 函数永远等待。当所指定的文件描述符之一准备好时或者函数捕捉到一个信号时，等待结束。如果捕捉到一个信号函数返回 - 1 并设置 errno 值。
当 timeout->tv_sec == 0 && timeout->tv_usec == 0 时，select 函数完全不阻塞。测试完所指定的描述符后立即返回。这种方式适合用轮询来获得描述符状态。
当 timeout->tv_sec ！= 0 || timeout->tv_usec ！= 0 时，select 函数等待所指定的秒数或微秒数。当指定的描述符之一准备好时或者指定的时间已超过时立即返回。
中间的三个参数 readfds，writefds，excpetfds 用下面的函数来设置。（这三个参数类型是 fd_set，也就是描述符集，每个描述符在其中占据一位，下面的函数使用描述符 fd 来指定是哪一位）

```c
void FD_CLR(int fd, fd_set *set)
int  FD_ISSET(int fd, fd_set *set)
void FD_SET(int fd, fd_set *set)
void FD_ZERO(fd_set *set)
```

第一个函数将 fd 指定的一位清除。第二个函数测试一个指定位是否设置。第三个函数用来设置一个指定位。最后一个函数将 fd_set 变量的所有位设置为 0。一般在定一个了一个描述符集变量后，必须先用 FD_ZERO 清除其所有位，然后再设置我们关心的各个位。
select 函数的中间三个参数 readfds，writefds，excpetfds 中的任意一个或者全部都可以为空指针，这表示对相应状态并不关心。如果这三个参数都是空指针，select 函数就相当于 sleep 函数，不过睡眠的时间更加精确。select 有三个可能的返回值：

- 返回值为 - 1 表示出错。比如所有指定的描述符都没有准备好时（select 正阻塞着），进程接收到了一个其它的信号。
- 返回值为 0 表示没有描述符准备好。这种情况出现在，要么 select 不阻塞立即返回，要么阻塞的时间已到，描述符都没有准备好。这是 readfds，writefds，excpetfds 三个描述符集的所有位均为 0。
- 返回值为正值表示已经有描述符准备好了。如果有同一个描述符的读和写都准备好了，返回值为 2。这时，readfds，writefds，excpetfds 三个描述符集中打开的位（为 1）对应于准备好的描述符。


另外，对于普通文件描述符，获取其读，写，异常状态时总是返回准备好了。如果在一个描述符上碰到了文件结尾处，select 指示该描述符是可读状态，而不指示其为异常状态。


### 2.int pselect(int nfds, fd_set _readfds, fd_set _writefds, fd_set _exceptfds, const struct timespec _timeout, const sigset_t *sigmask)

该函数域 select 函数类似。声明在 sys/select.h 文件中。有以下区别：

- select 的超时值用 timeval 结构指定，但 pselect 使用 timespec 结构指定。该结构如下所示：



```c
struct timespec {
               long    tv_sec;         /* seconds */
               long    tv_nsec;        /* nanoseconds */
           };
```


- pselect 的超时值被设置为 const，这保证了函数调用期间不会改变该该结构值。
- pselect 函数可以设置信号屏蔽字，如果 sigmask 为 null，那么在信号有关方面该函数和 select 运行状况相同。该函数以原子操作安装信号屏蔽字，在函数返回时恢复屏蔽字。




### 3.int poll(struct pollfd *fds, nfds_t nfds, int timeout)

该函数和 select 类似，不过接口不同。声明在 poll.h 文件中。该函数使用了一个结构体数组 fds，为每个要获取其状态的描述符分配了一个结构体（结构体中包含了描述符的各种状态），而不是为每个状态构造一个描述符集。stuct pollfd 结构体如下：

```c
struct pollfd {
               int   fd;         /* file descriptor */
               short events;     /* requested events */
               short revents;    /* returned events */
           };
```

结构体中的 events 成员用来告诉内核我们关心该描述符的什么状态。函数返回时，内核设置 revents 成员，以说明对于该描述符已经发生了什么事件。events 和 revents 取值如下：
events：

- POLLIN                    不阻塞地可读除高优先级外的数据（等效于 POLLRDNORM | POLLRDBAND）
- POLLRDNORM        不阻塞地可读普通数据（优先级波段为 0）
- POLLRDBAND          不阻塞地可读非 0 优先级波段数据
- POLLPRI                   不阻塞地可读高优先级数据


前四行用来测试可读性

- POLLOUT                 不阻塞地可写普通数据
- POLLWRNORM         与 POLLOUT 相同
- POLLWRBAND          不阻塞地可写非 0 优先级波段数据


这三行用来测试可写性
revents：

- POLLERR                  已出错
- POLLHUP                  已挂断
- POLLNVAL                 描述符不引用一打开文件


当描述符被挂断后，就不能再写向该描述符，但仍可能从该描述符中读到数据。应该区分文件结束和挂断之间的区别：如果正从终端输入数据，并键入了文件结束字符，POLLIN 被打开，于是就可读文件结束标志（read 返回 0）。如果测试的正在读调制解调器，并且电话线已挂断，则在 revents 中将街道 POLLHUP 通知（之前 POLLHUP 在 revents 中没有打开）。


### 4.ssize_t readv(int fd, const struct iovec *iov, int iovcnt)



### ssize_t writev(int fd, const struct iovec *iov, int iovcnt)

这两个函数在一次函数调用中读，写多个_非连续缓冲区_。被称为散布读和聚集写。声明在 sys/uio.h 文件中。第一个函数将 fd 文件中的数据读到 iov 数组中，第二个函数将 iov 数组中的数据聚集起来写入 fd 文件中。struct iovec 结构如下：

```c
struct iovec {
    void  *iov_base;    /* Starting address */
    size_t iov_len;     /* Number of bytes to transfer */
};
```

iov_base 成员指向缓冲区的首地址，iov_len 成员代表该缓冲区大小。writev 以顺序 iov[0]，iov[1] 至 iov[iovcnt-1]从缓冲区中聚集输出数据。函数返回输出的字节总数，通常，它应该等于所有缓冲区长度之和。readv 将读到的数据按照上述的顺序散步到缓冲区中。readv 函数总是填满一个缓冲区，再填写下一个缓冲区。readv 返回读到的字节数，遇到文件结尾无数据可读返回 0。![](imgclip_2.png)


### 5.void _mmap(void _addr, size_t length, int prot, int flags, int fd, off_t offset)

该函数将一个给定文件映射到一个存储区域中。声明在 sys/mman.h 文件中。该函数返回值为映射区的起始地址。addr 用于指定映射存储区的起始地址，通常将其设置为 0，表示由系统选择该映射区的起始地址（系统一般选择堆区和栈区中间的共享区）。fd 参数指定要被映射文件的描述符，在映射之前，先要打开该文件。length 参数指示映射区的长度（字节为单位）。offset 参数指示要映射字节在文件中的起始偏移量。prot 参数说明对映射存储区的保护要求：

- PROT_READ       映射区可读
- PROT_WRITE     映射区可写
- PROT_EXEC       映射区可执行
- PROT_NONE      映射区不可访问


对映射存储区的保护要求不能超过文件 open 模式访问权限。例如，若该文件是只读打开的，那么对映射存储区就不能指定 PROT_WRITE。
flags 参数影响映射存储区的多种属性：

- MAP_FIXED         返回值必须等于 addr。不推荐使用该标志，因为不利于可移植性。如果未指定此标志，而且 addr 非 0，则内核只把 addr 视为一种建议值，并不保证会采用该地址。将 addr 指定为 0 可获得最大可移植性。
- MAP_SHARED     该标志说明本进程对该映射区做的修改对其他进程是可见的。说白了，就是本进程对该映射区做的修改都会真正修改磁盘上的该文件。这样，其它使用该标志映射该文件的进程就会和本进程共享该文件，对该区域的修改对于这些进程都是可见的，最终也会修改磁盘上的文件。
- MAP_PRIVATE     使用了该标志，只是将文件的一个副本映射到存储区中，进程对存储区的修改，只是在副本上进行修改，不会影响到磁盘上的文件，当进程终止后，磁盘上的该文件没有变化。当然，对映射区的修改对于多个进程都是不可见的。


还有许多 MAP_类标志，详情参考 man 手册。
需要注意的是，一般映射区的长度会是内存页长的整数倍，那么需要映射的文件如果比内存的页长小，那么系统会把映射区内除了文件以外的多余部分设置为 0，对这部分的操作不会影响到文件。例如，文件长度为 12 字节，系统页长为 512 字节，因此多出的 500 字节系统会设置为 0，并且操作这 500 字节对文件无影响（如果想让文件映射满一页，可以先对打开的文件进行 lseek 操作，将文件指针移动到距离文件开头一页大小的位置，再随意向文件中写一个字符即可，实际上该操作将文件变大了再映射）。此外，与映射区相关信号有两个：**SIGSEGV 和 SIGBUS**。第一个信号通常用于指示进程试图访问对它不可用的存储区，如果进程企图存数据到 mmap 指定为只读的映射区，那么也会产生该信号。如果访问映射区的一部分，该部分已经不存在了，则会产生第二个信号。例如进程映射了一个文件后，另外一个进程又将该文件截短了，那么该进程如果访问被截去的部分时就会接收到第二个信号。最后，用 fork 产生子进程后，子进程会继承存储映射区（因为存储映射区也是父进程地址空间的组成部分）。子进程调用 exec 执行新程序后，不再继承存储映射区。


### 6.int mprotect(void *addr, size_t len, int prot)

该函数用来更改一个已映射存储区的权限。声明在 sys/mman.h 文件中。prot 许可值与 mmap 函数的该参数一样。addr 地址必须是系统页边界对齐。


### 7.int msync(void *addr, size_t length, int flags)

如果修改了共享映射存储区中的页，就可以使用该函数将页冲洗到被映射的文件中。该函数类似于 fsync，但作用于映射存储区。如果映射的存储区是私有的，就不会修改被映射的文件。addr 地址也必须和页边界对其。flags 参数用来控制冲洗的方式，必须要指定 MS_ASYNC 和 MS_SYNC 二者中的一个。MS_ASYNC 表示不等待冲洗完成，MS_SYNC 表示等待冲洗完成后函数才返回（冲洗完成之前进程一直阻塞）。MS_INVALIDATE 标志是可选标志，用来丢弃与磁盘没有同步的任何页（会将函数参数表示的地址范围的所有页丢弃），这一般不是期望的操作。声明在 sys/mman.h 文件中。


### 8.int munmap(void *addr, size_t length)

当进程终止时，或者调用了该函数后，映射的存储区会被自动解除。关闭文件描述符不会解除映射。声明在 sys/mman.h 文件中。该函数调用时，不会将映射区的内容写到磁盘文件中。对于 MAP_SHARED 标志后的映射区，内容同步的工作由内核虚存算法自动进程，和该函数是没有关系的。对于 MAP_PRIVATE 标志的映射区，调用该函数解除后将被丢弃。


## 第 15 章 进程间通信



### 1.int pipe(int pipefd[2])

该函数用来创建管道。声明在 unistd.h 文件中。pipefd 数组将返回两个文件描述符：pipefd[0] 为读而打开，pipefd[1] 为写而打开。向 pipefd[1] 描述符中写入的数据，可以从 pipefd[0] 描述符中读出。一般不在同一个进程中使用管道，因为没有意义，而是在父进程和子进程之间使用管道。当用 fork 创建子进程后，子进程会继承父进程已打开的文件描述符，因此也就继承了管道。这时候父进程和子进程中总共有四个管道端口，它们之间都是通的，也就是说，向父进程的 pipefd[1] 写入的数据既可以从父进程的 pipefd[0] 中读出也可以从子进程的 pipefd[0] 中读出；同理，向子进程的 pipefd[1] 写入的数据既可以从子进程的 pipefd[0] 中读出也可以从父进程的 pipefd[0] 中读出。但是管道中的数据被读一次后，就会被清除，不会保留。向管道中连续写入的数据，第二次的数据回追加到第一次数据后面，而不会覆盖第一次的数据。多个进程对同一个管道进行写操作时，如果写入的数据长度大于管道的尺寸 PIPE_BUF 时，可入的数据可能会穿插，如果写入的数据小于 PIPE_BUF，则没有穿插问题（可用 pathconf/fpathconf 函数获取 PIPE_BUF 值的大小）。一般在父子进程间使用管道，会关闭父进程的读（或写）端描述符，同时关闭子进程的写（或读）端描述符，使父子进程半双工通信。当子进程执行 exec 后，文件描述符会丢失，因此一般将管道端口描述符先重定向到标准输入输出描述符中，再使子进程执行 exec，这样在子进程中就可继续使用管道。当管道的一端被关闭后，下面两个规则起作用：

- 当读一个写端已被关闭的管道时，在所有数据被读取后，read 返回 0，表示到达了文件末尾。
- 如果写一个读端已被关闭的管道时，会产生 SIGPIPE 信号。如果忽略该信号或者捕捉该信号并从信号处理函数返回后，write 函数返回 - 1，errno 设置为 EPIPE。


最后，a. 历史上的管道是半双工的，b. 管道只能在具有公共祖先的进程之间使用。


### 2.FILE _popen(const char _command, const char *type)



### int pclose(FILE *stream)

第一个函数用来创建一个子进程，同时，创建父子进程之间的管道（这和 fork/system 是不同的），返回管道的一个端口至父进程。该子进程执行 shell 程序，然后再创建子进程执行 commad 命令（在这点上类似于 system 函数）。type 参数可以是 “r” 或者“w”，如果是 r，管道的写端将连接到 commad 进程的标准输出，此时函数返回管道的读端；如果 w，管道的读端将连接到 commad 进程的标准输入，此时函数返回管道的写端。
pclose 函数用来关闭标准 I/O 流，等待命令执行结束，然后返回 shell 的终止状态。函数的返回值类似于 system 函数的返回值。由于 pclose 中会调用 waitpid 等待它所创建的所有子进程，因此如果在程序中，用户自行设置了 SIGCHILD 的处理函数，并且函数中调用了 wait 等函数，则可能会影响 pclose 中的 waitpid 函数。他们均声明在 stdio.h 文件中。


### 3. 协同进程（coprocess）

当一个进程产生某个过滤进程的输入，同时又读取该过滤进程的输出，则称该过滤进程为协同进程。实际上协同进程就是帮其它进程处理数据，当然要从其它进程中获取要处理的数据，然后再将处理以后的数据返回给其它进程。


### 4.int mkfifo(const char *pathname, mode_t mode)

该函数用来创建一个 FIFO 文件。声明在 sys/stat.h 文件中。创建一个 FIFO 类似与创建一个文件，因此该函数类似于 open/creat 函数。mode 参数取值和 open/creat 的 mode 参数相同。创建 FIFO 文件的用户和组权限规则同创建普通文件是相同的。创建好的 FIFO 文件可以使用一般的 I/O 函数（open，close，read，write，unlink 等）来操作它。
当打开一个 FIFO 时：

- 如果没有指定 O_NONBLOCK，那么只读 open 要阻塞到某个其它进程为写而打开它；类似的，只写 open 要阻塞到某个其它进程为读而打开它。
- 如果指定了 O_NONBLOCK，只读 open 函数会立即返回（如果没有其它进程写 FIFO）；只写 open 将返回 - 1，并设置 errno 为 ENXIO（如果没有其它进程为读而打开 FIFO）。


和管道类似，若用 write 写一个尚无进程为读而打开的 FIFO，则产生信号 SIGPIPE；若某个 FIFO 的最后一个写进程关闭了该 FIFO，则将为该 FIFO 的读进程产生一个文件结束标志（如果以读写打开 FIFO，最后一个写进程关闭 FIFO 时，不会产生文件结束标志）。如果有多个进程写同一个 FIFO，如果不希望数据穿插，则需要考虑原子操作。和管道类似，常量 PIPE_BUF 说明了可被原子写入 FIFO 的最大数据量。
FIFO 有两种用途：

- FIFO 由 shell 命令使用以便将数据从一条管道线传送到另一条，为此无需创建中间临时文件。
- FIFO 用于客户进程 --- 服务器进程应用程序中，以便在客户进程和服务器进程之间传递数据。




### 5.key_t ftok(const char *pathname, int proj_id)

该函数用来为 XSI IPC 创建一个键（key）。声明在 sys/msg.h 文件中。**所谓 XSI IPC，指的是消息队列，信号量，共享存储这三种进程间通信方式**。不包括前边提到的管道和 FIFO。XSI IPC 创建的 IPC 结构，不会因为进程的终止而消失，不像管道和 FIFO，进程终止后管道就不存在了，FIFO 虽然存在，但其数据已被清空。使用该函数创建键时，pathname 参数所指定的文件必须存在，该函数将使用指定文件的 stat 结构中 st_dev 和 st_ino 字段和 proj_id 参数的后 8 位组合起来生成键。生成的键可供下边的函数来创建 XSI IPC 结构。创建消息队列，信号量，共享存储所用的函数分别为 msgget，semget，shmget，这三个函数都有两个类似的参数：一个 key（就是由 ftok 生成的键）和一个整型 msgflag。可以用这三个函数创建新的 IPC 结构或者打开已存在的 IPC 结构。如若要创建新结构，需满足下面两个条件之一：

- key 值为 IPC_PRIVATE
- key 当前未与特定类型的 IPC 结构相结合，并且 msgflag 终止定了 IPC_CREAT 位


如若需要打开已存在的 IPC 结构，key 值必须等于创建 IPC 结构时所指定的键，并且 msgflag 中不应指定 IPC_CREAT。为了打开已存在的 IPC 结构，key 值决不能为 IPC_PRIVATE，该值比较特殊，它总是用来创建一个新 IPC 结构。如果需要访问用 IPC_PRIVATE 键创建的 IPC 结构，一定要知道该结构的标识符，然后用其他 IPC 调用（msgsnd 和 msgrcv）使用该标识符，而不能用 msgget，semget，shmget 三个函数来打开。如果需要创建一个新的 IPC 结构，而且要确保不是引用具有同一标识符的一个已存在 IPC 结构，需要在 msgflag 中同时指定 IPC_CREAT 和 IPC_EXCL 位，这样的话，如果 IPC 结构已经存在就会造成出错，返回 EEXIST（这与指定了 O_CREAT 和 O_EXCL 标志的 open 函数类似）。消息队列，信号量，共享存储三者有各自的 IPC 结构，后边会列出。这些 IPC 结构中包含了一个相同结构 struct ipc_perm 结构，装有该 IPC 结构的权限和所有者。如下所示：

```c
struct ipc_perm
  {
    __key_t __key;            /* Key.  */
    __uid_t uid;            /* Owner's user ID.  */
    __gid_t gid;            /* Owner's group ID.  */
    __uid_t cuid;            /* Creator's user ID.  */
    __gid_t cgid;            /* Creator's group ID.  */
    unsigned short int mode;        /* Read/write permission.  */
   ............

   ...........
  };
```

uid，gid，cuid，cgid 分别指定了拥有者 ID，拥有者组 ID，创建者 ID，创建者组 ID。mode 字段指定了访问者权限，如下所示：（IPC 结构不需要执行，因此没有执行权限）

- 用户读                    0400
- 用户写（更改）      0200
- 组读                        0040
- 组写（更改）          0020
- 其他读                    0004
- 其他写（更改）      0002




### 6.int msgget(key_t key, int msgflg)

该函数用来创建一个新的消息队列或者打开一个现存消息队列，返回消息队列标识符。后面对消息队列进行操作的函数可以使用该标识符。该队列 ID 可以声明在 sys/msg.h 文件中。该函数会创建如下所示的结构体，并初始化一部分成员：msg_perm 中的各个值；msg_qnum，msg_lspid，msg_lrpid，msg_stime，msg_ctime 都设置为 0；msg_ctime 设置为当前时间；msg_qbytes 设置为系统限制值。

```c
struct msqid_ds
{
  struct ipc_perm msg_perm;    /* structure describing operation permission */
  __time_t msg_stime;        /* time of last msgsnd command */
  .......

  __time_t msg_rtime;

  .......

  __time_t msg_ctime;

  .......

  __syscall_ulong_t __msg_cbytes;
  msgqnum_t msg_qnum;        /* number of messages currently on queue */
  msglen_t msg_qbytes;        /* max number of bytes allowed on queue */
  __pid_t msg_lspid;        /* pid of last msgsnd() */
  __pid_t msg_lrpid;        /* pid of last msgrcv() */
  .......

  .......
};
```

函数执行成功，返回一个非负的队列 ID。该队列可被用于其他三个消息队列函数。


### 7.int msgctl(int msqid, int cmd, struct msqid_ds *buf)

该函数可以对消息队列进行各种控制操作。声明在 sys/msg.h 文件中。该函数是 XSI IPC 中类似于 ioctl 的函数（垃圾桶函数）。msqid 是消息队列 ID（就是 msgget 函数返回的队列描述符），cmd 参数指定了队列要执行的命令：

- IPC_STAT         取此队列的 msqid_ds 结构，并将它存放在 buf 指向的结构中
- IPC_SET           按由 buf 指向结构中的值，设置与此队列相关结构中的下列四个字段：msg_perm.uid，msg_perm.gid，msg_perm.mode 和 msg_qbytes。此命令只能由下面两种进程执行：一种是其有效用户 ID 等于 msg_perm.cuid 或 msg_perm.uid，另一种是具有超级用户权限的进程。此外，只有超级用户才能增加msg_qbytes 的值。
- IPC_RMID         从系统中删除该消息队列以及仍在该对列中的所有数据。这种删除立即生效。仍在使用这一消息队列的其他进程在它们下一次试图对此队列进行操作时，将出错返回 EIDRM。此命令只能由下列两种进程执行：一种是其有效用户 ID 等于 msg_perm.cuid 或 msg_perm.uid，另一种是具有超级用户权限的进程。




### 8.int msgsnd(int msqid, const void *msgp, size_t msgsz, int msgflg)

调用该函数将数据放到消息队列中。声明在 sys/msg.h 文件中。每个消息都有三部分组成，它们是：正长整型类型字段，非负长度（msgsz）以及实际数据字节（对应于长度）。消息总是放在队列尾端。msgp 参数指向一个长整型数，它包含了症的整型消息类型，紧跟其后的是消息数据。（若 msgsz 是 0，则无消息数据）若发送的最长消息是 512 字节，则可定义下列结构：

```c
struct mymesg {
　　long mtype;
　　char mtext[512];

};
```

然后，ptr 就是一个指向该结构的指针。接收者可以使用消息类型以非先进先出的次序取消息。参数 msgflg 的值可以指定为 IPC_NOWAIT，这类似于文件 I/O 的非阻塞标志。若消息队列已满（或者是队列中的消息总数等于系统限制值，或对列中的字节总数等于系统限制值），则指定 IPC_NOWAIT 的话，msgsnd 立即出错并返回 EAGAIN；没有指定 IPC_NOWAIT 的话，进程将阻塞到下述情况出现为止：有空间可以容纳要发送的消息；从系统中删除了此队列，或捕捉到一个信号，并从信号处理程序返回。第二种情况下，返回 EIDRM（“标识符被删除”）。第三种情况则返回 EINTR。
需要注意的是，对删除消息队列的处理不是很完善，因为对每个消息队列并没有设置一个引用计数器（对打开文件则有这种计数器），所以删除一个队列会造成仍在使用这一队列的进程在下次对队列进行操作时出错返回。信号量机制也是如此。而文件删除就比较完善，当使用某个文件的最后一个进程关闭了它的文件描述符后，才能删除文件内容。
当 msgsnd 成功返回后，与消息队列相关的 struct msqid_ds 结构将得到更新，以标明发出该消息的进程 ID（msg_lspid），进行该调用的时间（msg_stime），并指示队列中增加了一条消息（msg_qnum 增加 1）。


### 9.ssize_t msgrcv(int msqid, void *msgp, size_t msgsz, long msgtyp, int msgflg)

该函数从队列中取出消息。声明在 sys/msg.h 文件中。该函数的 msqid 参数和 msgp 参数和 9 中的函数类似。msgsz 参数表示要接收的长度。如果返回的消息长度大于 msgsz，并且 msgflg 中设置了 MSG_NOERROR，则接收到的消息会被截短。（这种情况下，不会通知我们消息被截短了，消息截去的那一部分被丢弃）如果没有设置 MSG_NOERROR 标志，而消息有太长，则函数出错返回 E2BIG（消息仍留在队列中）。type 参数用于指定想要取哪一种消息：

- type == 0     返回队列中的第一个消息
- type > 0       返回队列中消息类型为 type 的第一个消息
- type < 0      返回队列中消息类型值小于或等于| type | 的消息，如果这种消息有若干个，则取类型值最小的消息。type 值非 0 用于以非先进先出次序读消息。例如，若应用程序对消息赋优先权，那么 type 就可以是优先权值。如果一个消息队列由若干个客户进程和一个服务器进程使用，那么 type 字段可以用来包含客户进程的进程 ID（只要进程 ID 可以存放在长整型中）。


可以指定 flag 值为 IPC_NOWAIT，使操作不阻塞。这使得如果没有指定类型的消息，则 msgrcv 函数返回 - 1，errno 设置为 ENOMSG；如果没有指定 IPC_NOWAIT，则进程阻塞到如下情况出现才终止：有了指定类型的消息；从系统中删除了此队列（出错则返回 - 1 且 errno 置为 EIDRM）；或捕捉到一个信号并从信号处理程序返回（msgrcv 函数返回 - 1，errno 设置为 EINTR）。
msgrcv 函数成功执行时，内核更新与该消息队列相关联的 struct msqid_ds 结构，已指示调用者的进程 ID（msg_lrpid）和调用时间（msg_rtime），并将队列中的消息数减 1（msg_qnum）。
注意：当初实施消息队列时，是因为其他形式 IPC 都是半双工管道。而现在有了全双工管道。因此，在新的应用程序中不应该再使用消息队列。


### 10.int shmget(key_t key, size_t size, int shmflg)

该函数用来创建或者打开一个共享存储区，返回存储区标识符。声明在 sys/shm.h 文件中。后边对共享区操作的函数就可以使用该标识符。该函数的创建或者打开规则和 msgget 函数相同。当创建一个新段时，会创建一个相关的结构体 struct shmid_ds，并初始化下列成员：shm_perm 中的各个值；shm_lpid，shm_nattch，shm_atime，shm_dtime 成员都设置为 0；shm_ctime 成员设置为当前时间；shm_segsz 设置为请求长度 size。

```c
struct shmid_ds
  {
    struct ipc_perm shm_perm;        /* operation permission struct */
    size_t shm_segsz;            /* size of segment in bytes */
    __time_t shm_atime;            /* time of last shmat() */
　........
    __time_t shm_dtime;            /* time of last shmdt() */
   ........
    __time_t shm_ctime;            /* time of last change by shmctl() */
   ........
    __pid_t shm_cpid;            /* pid of creator */
    __pid_t shm_lpid;            /* pid of last shmop */
    shmatt_t shm_nattch;        /* number of current attaches */
   .........
  };
```

size 参数用来指定该共享存储段的长度，实现通常将其向上取为系统页长的整数倍。但是，如果应用指定的 size 值并非系统页长的整数倍，则最后一页的余下部分不可使用。如果正在创建一个新段，那么必须指定 size 值；如果正在引用一个现存段，则将 size 指定为 0。当创建一个新段时，段内的内容初始化为 0。


### 11.int shmctl(int shmid, int cmd, struct shmid_ds *buf)

该函数用来对指定的共享存储段进行控制。声明在 sys/shm.h 文件中。cmd 指定了下面 5 中命令中的一种：

- IPC_STAT          取此段的 shmid_ds 结构，并将它存放在由 buf 指向的结构中。
- IPC_SET            按 buf 指向结构中的值设置与此段相关结构中的下列三个字段：shm_perm.uid，shm_perm.gid 和 shm_perm.mode。此命令只能由下面两种进程执行：一种是其有效用户 ID 等于 shm_perm.cuid 或 shm_perm.uid，另一种是具有超级用户权限的进程。
- IPC_RMID          从系统中删除共享存储段，因为每个共享存储段有一个连接计数（shmid_ds 结构中的 shm_nattch 字段），所以除非该段最后一个进程终止或与该段分离，否则不会实际上删除该存储段。不管此段是否仍在使用，该段标识符立即被删除，所以不能再用 shmat 函数与该段连接。此命令只能由下面两种进程执行：一种是其有效用户ID 等于 shm_perm.cuid 或 shm_perm.uid，另一种是具有超级用户权限的进程。
- IPC_LOCK         将共享存储段锁定在内存中，此命令只能由超级用户执行。
- IPC_UNLOCK    解锁共享存储段。此命令只能由超级用户执行。




### 12.void _shmat(int shmid, const void _shmaddr, int shmflg)

一旦创建了共享存储段，就可以使用该函数将其连接到进程的地址空间中。声明在 sys/shm.h 文件中。共享存储段连接到调用进程的哪个地址上与 shmaddr 参数以及在 flag 中是否指定 SHM_RND 位有关：
a. 如果 shmaddr 为 0，则此段连接到由内核选择的第一个可用地址上。推荐使用这种方式。（一般位于堆区和栈区中间的共享区）
b. 如果 shmaddr 非 0，并且没有指定 SHM_RND，则此段连接到 shmaddr 所指定的地址上。
c. 如果 shmaddr 非 0，并且指定了 SHM_RND，则此段连接到 shmaddr mod ulus SHMLBA 所表示的地址上。
如果在 shmflag 中指定了 SHM_RDONLY 位，则以只读方式连接此段，否则以读写方式连接此段。shmat 函数的返回值是该段所连接的实际地址，如果函数出错返回 - 1。如果执行成功那么内核将使该共享存储段 shmid_ds 结构中的 shm_nattch 计数器值加 1。


### 13.int shmdt(const void *shmaddr)

当今成使用完存储共享区后，调用该函数是共享区与进程分离。声明在 sys/shm.h 文件中。该操作并不从系统中删除共享区标识符及其数据结构。该标志符让然存在，直至某个进程调用 shmctl（带命令 IPC_RMID）删除它。shmaddr 参数是共享区在进程中的起始地址。如果函数执行成功，shmid_ds 结构中的 shm_nattch 计数器值减 1。
**小结：掌握管道和 FIFO，因为在大量应用程序中仍可有效地使用这两种基本技术。在新的应用程序中，尽量避免使用消息队列以及信号量，而用全双工管道和记录锁取代。共享存储段有其应用场合，而 mmap 函数也能提供同样的功能。**


## 第 16 章 网络 IPC：套接字



### 1.int socket(int domain, int type, int protocol)

该函数用来创建一个套接字。声明在 sys/socket.h 文件中。参数 domain 用来指定套接字使用的地址族，取值如下：

- AF_INET            ipv4 因特网域
- AF_INET6          ipv6 因特网域
- AF_UNIX            unix 域
- AF_UNSPEC      未指定


参数 type 指定套接字的类型，取值如下：

- SOCK_DGRAM                长度固定的，无连接的不可靠报文传递（UDP）
- SOCK_RAW                      IP 协议的数据包接口
- SOCK_SEQPACKET         长度固定的，有序，可靠的面向连接报文传递
- SOCK_STREAM                有序，可靠，双向的面向连接字节流（TCP）


参数 protocol 通常为零，表示按给定的域和套接字类型选择默认的协议。当对同一域和套接字类型支持多个协议时，可以使用 protocol 参数选择一个特定协议。在 AF_INET 通信域中套接字类型 SOCK_STREAM 的默认协议是 TCP，在 AF_INET 通信域中套接字类型 SOCK_DGRAM 的默认协议是 UDP。
socket 函数创建的套接字描述符可用普通文件操作函数来操作，但并适用所有的文件操作函数。


### 2.int shutdown(int sockfd, int how)

该函数用来关闭套接字的输入 / 输出端口。声明在 sys/socket.h 文件中。如果参数 how 是 SHUT_RD（关闭读端），那么无法从套接字读取数据（但仍可以读）；如果参数 how 是 SHUT_WR（关闭写端），那么无法使用套接字发送数据（但仍可以写）；使用 SHUT_RDWR 则将无法同时读取和发送数据。需要注意的是，close 函数虽然可以关闭套接字，但是如果使用 dup2 复制了套接字描述符，那么直到最后一个套接字描述符被关闭后，套接字才会被释放；而 shutdown 函数是一个套接字处于不活动状态，无论它的描述符有多少。


### 3.uint32_t htonl(uint32_t hostlong)



### uint16_t htons(uint16_t hostshort)



### uint32_t ntohl(uint32_t netlong)



### uint16_t ntohs(uint16_t netshort)

这四个函数用来完成本地字节序和网络字节序之间的转换。声明在 arpa/inet.h 文件中。n 代表 network，h 代表 host，l 代表 long，s 代表 short。奔腾处理器上的 Linux 系统采用小端字节序。


### 4. 地址格式

为了是不同格式地址能够被传入到套接字函数，地址会被强制转换成通用的地址结构 struct sockaddr。
**Linux 对该通用地址结构的实现如下（/usr/include/i386/bits/socket.h）：**

```c
struct sockaddr
  {
    __SOCKADDR_COMMON (sa_);    /* Common data: address family and length.  */   /* #define __SOCKADDR_COMMON(sa_)  \ sa_family_t sa_prefix##family */
    char sa_data[14];        /* Address data.  */
  };
```

**Linux 中 IPV4 的套接字地址 sockaddr_in 格式如下：**

```c
struct sockaddr_in
  {
    __SOCKADDR_COMMON (sin_);
    in_port_t sin_port;            /* Port number.  */        /* typedef uint16_t in_port_t; */
    struct in_addr sin_addr;        /* Internet address.  */        /* typedef uint32_t in_addr_t; */
    /* Pad to size of `struct sockaddr'.  */
    unsigned char sin_zero[sizeof (struct sockaddr) -
               __SOCKADDR_COMMON_SIZE -
               sizeof (in_port_t) -
               sizeof (struct in_addr)];
  };

struct in_addr
  {
    in_addr_t s_addr;
  };
```

**Linux 中 IPV6 的套接字地址 sockaddr_in6 格式如下：**

```c
struct sockaddr_in6
  {
    __SOCKADDR_COMMON (sin6_);
    in_port_t sin6_port;    /* Transport layer port # */
    uint32_t sin6_flowinfo;    /* IPv6 flow information */
    struct in6_addr sin6_addr;    /* IPv6 address */
    uint32_t sin6_scope_id;    /* IPv6 scope-id */
  };

struct in6_addr
  {
    union
      {
    uint8_t    __u6_addr8[16];
#if defined __USE_MISC || defined __USE_GNU
    uint16_t __u6_addr16[8];
    uint32_t __u6_addr32[4];
#endif
      } __in6_u;
#define s6_addr            __in6_u.__u6_addr8
#if defined __USE_MISC || defined __USE_GNU
# define s6_addr16        __in6_u.__u6_addr16
# define s6_addr32        __in6_u.__u6_addr32
#endif
  };
```

在使用时，无论是 ipv4 还是 ipv6，均要被强制转换成 struct sockaddr_in 结构传入到套接字例程中。


### 5.in_addr_t inet_addr(const char *cp)



### char *inet_ntoa(struct in_addr in)



### const char _inet_ntop(int af, const void _src, char *dst, socklen_t size)



### int inet_pton(int af, const char _src, void _dst)

这些函数用于二进制地址格式和点分十进制字符串格式之间的相互转换。它们均声明在 arpa/inet.h 文件中。前两个函数只能用于 IPV4 地址。后两个新函数支持 IPV4 和 IPV6 地址。第一和第三个函数将点分十进制字符串转换成二进制地址，第二和第四个函数刚好相反。后两个函数的 af 参数取值为：AF_INET 和 AF_INET6，分别代表 ipv4 域和 ipv6 域。第三个函数的 size 参数指定了保存文本字符串缓冲区（dst）的大小，有两个常数用于简化工作：INET_ADDRSTRLEN 和 INET6_ADDRSTRLEN，前者可放 ipv4 地址，后者可放 ipv6 地址。对于第四个函数，dst 参数指向的地址空间必须足够大，以便能够存入 32 位地址（当 af 指定为 AF_INET）和 128 位地址（当 af 指定为 AF_INET6）。


### 6.struct hostent *gethostent(void)



### void sethostent(int stayopen)



### void endhostent(void)

调用这些函数可以获取计算机的主机信息。均声明在 netdb.h 文件中。第一个函数会打开数据库文件（如果该文件没有打开），如果文件是打开的，则该函数返回文件的下一条目；第二个函数也可以打开数据库文件，如果文件是打开的，则将其回绕。第三个函数关闭数据库文件。第一个函数返回的 struct hostent 结构如下：

```c
struct hostent {
               char  *h_name;            /* official name of host */
               char ### h_aliases;         /* alias list */
               int    h_addrtype;        /* host address type */
               int    h_length;          /* length of address */
               char ### h_addr_list;       /* list of addresses */      
           }
```



### 7.struct netent *getnetbyaddr(uint32_t net, int type)



### struct netent _getnetbyname(const char _name)



### struct netent *getnetent(void)



### void setnetent(int stayopen)



### void endnetent(void)

这几个函数用来获取网络名字和网络号。均声明在 netdb.h 文件中。将网络名字和网络号包含在 struct netent 结构体中，该结构体如下：

```c
struct netent {
               char      *n_name;     /* official network name */
               char     ### n_aliases;  /* alias list */
               int        n_addrtype; /* net address type */
               uint32_t   n_net;      /* network number */        // 返回的地址采用网络字节序。
}
```



### 8.struct protoent _getprotobyname(const char _name)



### struct protoent *getprotobynumber(int proto)



### struct protoent *getprotoent(void)



### void setprotoent(int stayopen)



### void endprotoent(void)

这几个函数用来获取协议名字和协议号。均声明在 netdb.h 文件中。将协议名字和协议号包含在 struct protoent 结构体中，该结构体如下：

```c
struct protoent {
               char  *p_name;       /* official protocol name */
               char ### p_aliases;    /* alias list */
               int    p_proto;      /* protocol number */
           }
```



### 9.struct servent _getservbyname(const char _name, const char *proto)



### struct servent _getservbyport(int port, const char _proto)



### struct servent *getservent(void)



### void setservent(int stayopen)



### void endservent(void)

这几个函数用来获取服务名字和服务的端口号。均声明在 netdb.h 文件中。返回的 struct servent 结构如下：

```c
struct servent {
               char  *s_name;       /* official service name */
               char ** s_aliases;    /* alias list */
               int    s_port;       /* port number */
               char  *s_proto;      /* protocol to use */
           }
```



### 10.int getaddrinfo(const char _node, const char _service, const struct addrinfo *hints, struct addrinfo** res)

const char *gai_strerror(int errcode)


### void freeaddrinfo(struct addrinfo *res)

第一个函数通过主机名字或服务名字来获取主机的地址信息。声明在 netdb.h 文件中。主机可能有多个地址，因此返回一个节点类型为 struct addrinfo 的链表 res。struct addrinfo 结构如下：

```c
struct addrinfo {
               int              ai_flags;
               int              ai_family;
               int              ai_socktype;
               int              ai_protocol;
               socklen_t        ai_addrlen;
               struct sockaddr *ai_addr;
               char            *ai_canonname;
               struct addrinfo *ai_next;
           };
```

hints 参数是一个过滤地址的模板，用来选择特定的地址。它仅使用 ai_family，ai_flags，ai_protocol 和 ai_socktype 字段。剩余的整数字段必须为 0，指针字段必须为空。ai_flags 的使用的标志参考《apue》。如果该函数失败，不能使用 perror 或 strerror 函数来生成错误信息，而要使用第二个的函数。第二个函数用来释放这些链表节点。


### 11.int getnameinfo(const struct sockaddr _sa, socklen_t salen, char _host, size_t hostlen, char *serv, size_t servlen, int flags)

该函数将套接字地址转换成主机名或服务器名。声明在 netdb.h 文件中。如果 host 非空，它指向一个长度为 hostlen 字节的缓冲区用于存储返回的主机名；如果 serv 非空，它指向一个长度为 servlen 字节的缓冲区用于存储返回的服务名。flags 参数指定一些转换的控制方式，取值参考《apue》。


### 12. int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen)

该函数将地址绑定到一个套接字上。声明在 sys/socket.h 文件中。参数 addr 有以下规则：

- 在进程所运行的机器上，指定的地址必须有效，不能指定一个其他机器上的地址。
- 地址必须和创建套接字时的地址族所支持的格式相匹配。
- 端口号必须不小于 1024，除非该进程具有超级用户特权。


对于因特网域（AF_INET/AF_INET6），如果指定 IP 地址为 INADDR_ANY，套接字端点可以被绑定到系统的所有网络接口上，这意味着可以收到这个系统所安装的所有网卡的数据。一般情况下，与客户端套接字关联的地址没有太大意义，可以让系统选择一个默认地址；服务器端需要绑定地址一个众所周知的地址，使得客户端可以进行连接。另外，如果没有使用 bind 绑定地址，那么当调用 connect 或 listen 时，系统会自动将一个地址绑定到套接字上。


### 13.int getsockname(int sockfd, struct sockaddr _addr, socklen_t _addrlen)

调用该函数来获得绑定到套接字上的一个地址。声明在 sys/socket.h 文件中。调用该函数时，addrlen 参数指定了 addr 缓冲区的大小；函数返回时，该参数被设置成缓冲区实际被使用的大小。如果提供的缓冲区和地址的实际大小不匹配，则将其截断而不报错。如果套接字当前没有绑定地址，函数结果没有意义


### 14.int getpeername(int sockfd, struct sockaddr _addr, socklen_t _addrlen)

如果套接字已经和对方连接，调用该函数可以获得对方的地址。声明在 sys/socket.h 文件中。该函数和 13 中的函数类似。


### 15.int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen)

该函数在 sockfd 套接字和 addr 指定的服务器之间建立一个连接。声明在 sys/socket.h 文件中。使用该函数建立连接时，可能会失败，因此应用程序必须能够处理该函数返回的错误，这些错误可能由一些瞬时变化条件引起（这在一些负载很重的服务器上很可能发生）。关于函数返回的错误值类型和原因，参考 man 手册。另外，对于 UDP 套接字调用该函数的话，所有发送报文的目标地址被设置为该函数中指定的地址，这样每次传送报文的时候就不许要再提供地址，此外，也仅能接收来自指定地址的报文。


### 16.int listen(int sockfd, int backlog)

服务器端调用该函数监听套接字，来宣告可以接受客户端的连接请求。声明在 sys/socket.h 文件中。backlog 参数指定了该套接字能连接的最大数量。一旦超过了最大值，服务器会拒绝多余连接请求。


### 17.int accept(int sockfd, struct sockaddr _addr, socklen_t _addrlen)

一旦服务器调用了 listen，就可调用该函数获得连接请求并建立连接。声明在 sys/socket.h 文件中。该函数返回一个套接字描述符，该套接字连接到了客户端。返回的套接字和原始套接字 sockfd 具有相同的套接字类型和地址族。而原始套接字 sockfd 并没有关联到客户端的套接字，而是继续保持可用状态并接受其他连接请求。如果不关心客户端的地址，则可将 addr 参数和 addrlen 置为 null。否则，addr 设为足够大的缓冲区来存放客户端的地址信息，len 指定了该缓冲区的大小。返回时，函数会在缓冲区填充客户端地址信息并更新 len 的大小。
如果没有连接请求，accept 函数会阻塞到直到一个请求到来。如果 sockfd 设置为非阻塞模式，accept 函数会返回 - 1 并将 errno 设置为 EAGAIN 或 EWOULDBLOCK。此外，服务器端可以使用 poll 或者 select 函数来等待一个请求的到来，这时，一个请求套接字会以可读方式出现。


### 18.ssize_t send(int sockfd, const void *buf, size_t len, int flags)



### ssize_t sendto(int sockfd, const void _buf, size_t len, int flags, const struct sockaddr _dest_addr, socklen_t addrlen)



### ssize_t sendmsg(int sockfd, const struct msghdr *msg, int flags)

这三个函数用来发送数据。声明在 sys/socket.h 文件中。第一个函数的用法和 write 类似，用于 TCP 套接字发送数据（UDP 套接字不能使用它来发送），flags 标志取值如下：

- MSG_DONTROUTE        勿将数据路由出本地网络
- MSG_DONTWAIT            允许非阻塞操作（等价于使用了 O_NONBLOCK）
- MSG_EOR                       如果协议支持，此为记录结束
- MSG_OOB                       如果协议支持，发送带外数据


如果 send 成功返回，并不代表连接的另一端接收到了数据，所保证的仅是当 send 成功返回时，数据已经无错误的发送到网络上。
UDP 套接字可以使用第二个函数来发送数据，dest_addr 参数指定了目标地址。TCP 套接字如果使用第二个函数来发送数据的话，会忽略目标地址。
第三个函数用来发送由 struct msghdr 结构体指定的多重缓冲区的数据，该函数和 writev 函数类似。struct msghdr 结构体如下：

```c
struct msghdr {
               void         *msg_name;       /* optional address */
               socklen_t     msg_namelen;    /* size of address */
               struct iovec *msg_iov;        /* scatter/gather array */
               size_t        msg_iovlen;     /* # elements in msg_iov */
               void         *msg_control;    /* ancillary data, see below */
               size_t        msg_controllen; /* ancillary data buffer len */
               int           msg_flags;      /* flags on received message */
           };
```



### 19.ssize_t recv(int sockfd, void *buf, size_t len, int flags)



### ssize_t recvfrom(int sockfd, void _buf, size_t len, int flags, struct sockaddr _src_addr, socklen_t *addrlen)



### ssize_t recvmsg(int sockfd, struct msghdr *msg, int flags)

这三个函数用来接收数据。声明在 sys/socket.h 文件中。第一个函数的用法和 read 函数类似，flags 标志取值如下：

- MSG_OOB               如果协议支持，接收带外数据
- MSG_PEEK              返回报文内容而不真正取走报文
- MSG_TRUNC           即使报文被截断，要求返回的是报文的实际长度
- MSG_WAITALL         等待直到接收完 len 指定的数据长度（仅 SOCK_STREAM）


当指定为 MSG_PEEK 标志时，可以查看下一个要读的数据但不会真正取走，当再次调用 read 或 recv 函数时会返回刚才查看到的数据。
对于 SOCK_STREAM 套接字（TCP 套接字），默认情况下接收到数据可以比请求的少，但是如果设置了 MSG_WAITALL 标志，则直到接收完 len 指定的数据长度，recv 函数才返回。对于 SOCK_DRAM 和 SOCK_SEQPACKET 套接字（它们是基于报文而不是字节流的），MSG_WAITALL 标志没有改变什么行为，因为这些套接字本身就是一次取走整个报文。如果发送者调用 shudown 来结束传输，或者发送端已经关闭，那么当所有数据接收完毕后，recv 返回 0。
第二个函数可以获得发送者的套接字端点地址（src_addr 和 addrlen 必须非空），该函数通常用于 UDP 套接字，也可以用于 TCP 套接字（这时等同于 recv）。
第三个函数将接收到的数据送入多个缓冲区（类似于 readv），也可以接收辅助数据。msg 参数指定了用于接收数据的缓冲区。可以设置参数 flags 来改变 recvmsg 的默认行为，返回时，msghdr 结构中的 msg_flags 字段被设置为所接收数据的各种特征（进入 recvmsg 时的 msg_flags 被忽略），这些返回的 msg_flags 标志如下：

- MSG_CTRUNC         控制数据被截断
- MSG_DONTWAIT     recvmsg 处于非阻塞模式
- MSG_EOR                接收到记录结束符
- MSG_OOB                接收到带外数据
- MSG_TRUNC            一般数据被截断




### 20. int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen)

该函数用来设置套接字选项。声明在 sys/socket.h 文件中。套接字有三个层次的选项（XSI 仅定义了前两个选项）：

- 通用选项，工作在所有套接字类型上。
- 在套接字层次管理的选项，但是依赖于下层协议的支持。
- 特定于某协议的选项，为每个协议所独有。


参数 level 标识了选项的层次，如果是通用层次的选项，level 设置成 SOL_SOCKET；否则，level 设置成套接字层次的选项，例如，对于 TCP 选项，level 是 IPPROTO_TCP，对于 IP 选项，level 是 IPPROTO_TCP。参数 optname 代表通用层次和套接字层次的选项，参考《apue》。参数 optval 根据选项的不同指向一个数据结构或者一个整数。一些选项是 on/off 开关，当 valopt 整数非 0，选项被启用，当 optval 整数为 0，选项被禁止。参数 optlen 指定了 optval 指向对象的大小。


### 21.int getsockopt(int sockfd, int level, int optname, void _optval, socklen_t _optlen)

该函数用来获取套接字当前的选项。声明在 sys/socket.h 文件中。该函数调用时，optlen 设置成 optval 缓冲区大小，函数返回时，optlen 被更新为实际尺寸。如果实际尺寸大于 optlen 初始设置值，选项会被截断而不报错。


### 22. 带外数据

带外数据又叫做紧急数据，TCP 协议支持一个字节的带外数据，UDP 不支持带外数据。在三个 send 函数中指定了标志 MSG_OOB，就可以传输待外数据，传送的若干个字节中最后一个字节就是带外数据。此外，接收进程可以调用 fcntl(sockfd，F_SETOWN，pid) 函数来设置 pid 进程对套接字信号进行处理。当接收进程接收到带外数据时，pid 指定的进程就会收到一个套接字信号。调用 owner = fcntl(sockfd，F_GETOWN，0) 可以获得处理套接字信号的进程 ID。


### 23.int sockatmark(int sockfd)

TCP 协议支持紧急标记（在普通数据流中紧急数据所处的位置），如果采用套接字选项 SO_OOBINLINE，那么可以在普通数据中接收紧急数据。当系一个要读的字节是紧急数据的话，该函数返回 1。
此外，当带外数据出现在套接字读取队列时，select 函数会返回一个文件描述符并且拥有一个异常状态。可以在普通数据流上接受紧急数据，或者在某个 recv 函数中采用 MSG_OOB 标志在其他队列数据之前接收紧急数据。TCP 队列仅有一字节的紧急数据，如果在接收当前的紧急数据之前又有新的紧急数据到来，那么当前的字节就成了普通字节（新的最后一个字节是紧急数据）。


### 24. 套接字非阻塞和基于套接字的异步 I/O

通常，recv 函数没有数据可用时会阻塞等待。同样地，当套接字输出队列没有足够空间来发送消息 send 函数也会阻塞。在套接字非阻塞模式下，这些函数不会阻塞而是失败，设置 errno 为 EWOULDBLOCK 或者 EAGAIN。非阻塞模式下，可以使用 poll 或 select 来判断套接字何时能接收或者传输数据。
在基于套接字的异步 I/O 中，当能够从套接字中读取数据，或者套接字写队列中的空间变得可用时，可以安排发送信号 SIGIO。通过两个步骤来使用异步 I/O：

- 建立套接字拥有者关系，信号可以被传送到合适的进程
- 设置套接字选项（使用 fcntl），使得套接字可用时（可读或可写）发信号告知


有三种方式完成步骤 a：

- 在 fcntl 中使用 F_SETOWN 命令
- 在 ioctl 中使用 FIOSETOWN 命令
- 在 ioctl 中使用 SIOCSPGRP 命令


有两种方式完成步骤 b：

- 在 fcntl 中使用 F_SETFL 命令并且启用文件标志 O_ASYNC。
- 在 ioctl 中使用 FIOASYNC。
