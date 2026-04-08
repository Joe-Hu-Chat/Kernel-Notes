# VFS

**VFS** forms unified interfaces for `syscall`s. Necessary `file_operations` will be implemented in device drivers

## ksys_write

![image-20240604170631759](./.04_linux_vfs/image-20240604170631759.png)

### vfs_write

![image-20240604170743875](./.04_linux_vfs/image-20240604170743875.png)

## ksys_read

![image-20240604170850849](./.04_linux_vfs/image-20240604170850849.png)

### vfs_read

![image-20240605094435370](./.04_linux_vfs/image-20240605094435370.png)

## fdget_pos

![image-20240604150617306](./.04_linux_vfs/image-20240604150617306.png)



![image-20240604150652638](./.04_linux_vfs/image-20240604150652638.png)

### __to_fd

![image-20240604150718984](./.04_linux_vfs/image-20240604150718984.png)

### __fdget

![image-20240604150834743](./.04_linux_vfs/image-20240604150834743.png)

### __fget_light

![image-20240604150856209](./.04_linux_vfs/image-20240604150856209.png)



![image-20240604151006745](./.04_linux_vfs/image-20240604151006745.png)



![image-20240604151110189](./.04_linux_vfs/image-20240604151110189.png)

### READ_ONCE

![image-20240604151200167](./.04_linux_vfs/image-20240604151200167.png)



![image-20240604151236890](./.04_linux_vfs/image-20240604151236890.png)

#### Generic

This seems just to improve compile-time performance by using `_Generic`

![image-20240604160359427](./.04_linux_vfs/image-20240604160359427.png)

ref: https://en.cppreference.com/w/c/language/generic



### `struct` fd

![image-20240604162121456](./.04_linux_vfs/image-20240604162121456.png)

#### `struct` files_struct

![image-20240604162610409](./.04_linux_vfs/image-20240604162610409.png)

#### `struct` fdtable

![image-20240604162720039](./.04_linux_vfs/image-20240604162720039.png)

#### `struct` file

![image-20240604163025522](./.04_linux_vfs/image-20240604163025522.png)

#### `struct` file_operations

```c
struct file_operations {
	struct module *owner;
	loff_t (*llseek) (struct file *, loff_t, int);
	ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
	ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);
	ssize_t (*read_iter) (struct kiocb *, struct iov_iter *);
	ssize_t (*write_iter) (struct kiocb *, struct iov_iter *);
	int (*iopoll)(struct kiocb *kiocb, bool spin);
	int (*iterate) (struct file *, struct dir_context *);
	int (*iterate_shared) (struct file *, struct dir_context *);
	__poll_t (*poll) (struct file *, struct poll_table_struct *);
	long (*unlocked_ioctl) (struct file *, unsigned int, unsigned long);
	long (*compat_ioctl) (struct file *, unsigned int, unsigned long);
	int (*mmap) (struct file *, struct vm_area_struct *);
	unsigned long mmap_supported_flags;
	int (*open) (struct inode *, struct file *);
	int (*flush) (struct file *, fl_owner_t id);
	int (*release) (struct inode *, struct file *);
	int (*fsync) (struct file *, loff_t, loff_t, int datasync);
	int (*fasync) (int, struct file *, int);
	int (*lock) (struct file *, int, struct file_lock *);
	ssize_t (*sendpage) (struct file *, struct page *, int, size_t, loff_t *, int);
	unsigned long (*get_unmapped_area)(struct file *, unsigned long, unsigned long, unsigned long, unsigned long);
	int (*check_flags)(int);
	int (*flock) (struct file *, int, struct file_lock *);
	ssize_t (*splice_write)(struct pipe_inode_info *, struct file *, loff_t *, size_t, unsigned int);
	ssize_t (*splice_read)(struct file *, loff_t *, struct pipe_inode_info *, size_t, unsigned int);
	int (*setlease)(struct file *, long, struct file_lock **, void **);
	long (*fallocate)(struct file *file, int mode, loff_t offset,
			  loff_t len);
	void (*show_fdinfo)(struct seq_file *m, struct file *f);
#ifndef CONFIG_MMU
	unsigned (*mmap_capabilities)(struct file *);
#endif
	ssize_t (*copy_file_range)(struct file *, loff_t, struct file *,
			loff_t, size_t, unsigned int);
	loff_t (*remap_file_range)(struct file *file_in, loff_t pos_in,
				   struct file *file_out, loff_t pos_out,
				   loff_t len, unsigned int remap_flags);
	int (*fadvise)(struct file *, loff_t, loff_t, int);
} __randomize_layout;
```



## Unified Interface

It allows userspace applications to use standard system calls—such as `open()`, `read()`, and `write()`—regardless of the underlying storage medium or filesystem protocol.



## Key Objects

**Superblock**: Stores global information about a specific filesystem

**Inode**: Represent a specific **file object** and its metadata (permissions, size, etc.).

**Dentry**: Links a filename to an inode and is cached for fast path lookups.

**File object**: Represents an open file in a specific process's context, tracking the current offest.



# /dev



Types of Device Files:

Character Devices (`c`): Data is accessed as a stream of bytes (e.g., keyboards, mice)

Block Device (`b`): Data is accessed in fixed-size blocks (e.g., hard dirves, USB sticks)

Pseudo-Devices: Virtual interfaces that provide system services, such as `dev/null` (discard data), `/dev/zero` (provides null bytes), and `/dev/random` (generates entropy).



Major/minor numbers:

The major number tells the VFS which device driver handles a dev file.



Devtmpfs: Modern Linux systems often mount a special filesystem called `devtmpfs` at `/dev`, which automatically creates and destroys these device nodes as hardware is added or removed.



# procfs

System and Process status information. Mostly used for process-specific data

procfs是为了反映系统以及进程的状态信息**，**sysfs用于Linux设备驱动模型

历史最早，最初就是用来跟内核交互的唯一方式，用来获取处理器、内存、设备驱动、进程等各种信息。



# sysfs

**sysfs** is a virtual filesystem in the Linux kernel that provides a standardized interface to expose kernel objects, their attributes, and relationships to user space. It is mounted at `/sys` and allows users and applications to query and **configure kernel subsystems, devices, and drivers** through virtual files.



sysfs 跟 kobject 框架紧密联系，而 kobject 是为设备驱动模型而存在的，所以 sysfs 是为设备驱动服务的。

**sysfs** is tightly integrated with the `kobject` system in the kernel. Each registered `kobject` creates a corresponding directory in `sysfs`, reflecting its parent-child hierarchy.



/sys/devices/: Mirrors the internal kernel device tree, representing physical and logical devices.

/sys/class/: Groups devices by types (e.g., network interfaces, block devices).

/sys/bus/: Contains directories for each bus type (e.g., PCI, USB), with subdirectories for devices and drivers.

/sys/module/: Information about loaded kernel module

/sys/power/: Power management settings





# debugfs

`sys/kernel/debug`

Debugfs exists as a simple way for kernel developers to make information available to user space.

debugfs也是一种用来调试内核的内存文件系统，内核开发者可以通过debugfs和用户空间交换数据



debugfs 从名字来看就是为debug而生，所以更加灵活。
