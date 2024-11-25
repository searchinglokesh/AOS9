
This chapter delves into the organization and management of file systems in UNIX, including **local file systems** and **distributed file systems**.

- **Local File Systems**: Handle storage and data management on devices directly connected to the system.
- **Distributed File Systems**: Enable users to access files stored on remote machines seamlessly.

The chapter places special emphasis on **local file systems**, highlighting the most common general-purpose systems found in modern UNIX environments.

---

### Common UNIX File Systems

1. **System V File System (S5FS)**
2. **Berkeley Fast File System (FFS)**

The **Vnode interface** allows multiple file system types to coexist on a single machine, providing abstraction and uniformity across different file systems.

---

### File Abstraction in UNIX

UNIX's file abstraction extends beyond traditional files and directories, encompassing various I/O objects such as:

- **Network connections** through sockets or streams.
- **Interprocess communication** via pipes and FIFOs.
- **Block and character devices**.

This architecture is supported by the **Vnode/VFS framework**.

---

### UNIX Block Buffer Cache

The **buffer cache** is another critical topic in this chapter, focusing on the UNIX block buffer cache and its role in file system performance optimization.

---

### System V File System Architecture

The System V file system resides on a single logical disk or partition. Key features include:
![System V File System](Pasted image 20241113003844.png)

1. **Partitions**:
    
    - Viewed as a linear array of blocks, where each block is 512 bytes multiplied by a power of 2.
    - The disk driver translates physical block numbers into **cylinder, track, and sector numbers**.
2. **Boot Area**:
    
    - Found at the beginning of the partition, containing the bootstrap code for the operating system.
3. **Superblock**:
    
    - Holds the attributes and metadata of the file system.
4. **Inode List**:
    
    - A linear array of inodes, with one inode per file.
    - Each inode, identified by its inode number, contains metadata about the file.
    - **Size**: 64 bytes.
    - The inode list determines the maximum number of files the partition can contain.
5. **Data Area**:
    
    - Stores data blocks for files and directories.
    - Includes **indirect blocks**, which point to file data blocks.

---

### Directories

Directories are special files containing lists of files and subdirectories. Characteristics:

- Records are fixed in size (16 bytes).
- **Structure**:
    - First 2 bytes: Inode number.
    - Next 14 bytes: File name.

This structure limits a directory to **65,535 files**, leading to inefficiencies when directories grow large.

---

### Inodes

Each file has an associated **inode** that stores its metadata.

1. **In-Memory Inode (`struct inode`)**:
    
    - Temporary representation used when a file is open.
2. **On-Disk Data Inode (`struct dinode`)**:
    
    - Persistent storage of inode data.

**Key Fields**:

- **File Type**:
    - `IFREG` for regular files.
    - `IFDIR` for directories.
    - `IFBLK` for block devices.
    - `IFCHR` for character devices.
- **Permissions**: Lower 9 bits specify read, write, and execute permissions for the owner, group, and others.
- **`di_addr` Array**: Manages file growth by allocating new blocks as needed, ensuring flexibility without significant fragmentation.
  ![[Pasted image 20241124052658.png]]

---

### Efficient File Storage

UNIX handles small and large files efficiently by leveraging the **13-element block array** in each inode:![[{3ADBB264-7C68-46EE-BFB3-B4D28306A4E0}.png]]

- **First 10 entries**: Direct block pointers.
- **11th entry**: Points to a single indirect block.
- **12th entry**: Points to a double indirect block.
- **13th entry**: Points to a triple indirect block.

This system is optimized for small files but can also accommodate large files with additional block references.

---

### Sparse Files and Holes

UNIX files can have "holes," or unallocated blocks, to avoid wasting disk space:

- **Holes** span entire blocks but consume no actual storage.
- The kernel sets corresponding `di_addr` entries to zero.
- Reading these blocks results in zero-filled data.
- However, issues may arise during file copying, potentially exhausting disk space.

---


Directories:
- special file containing list of files and subdirectories. 
- Contains fixed size records of 16 bytes each.
- First 2 bytes contain inode number .
- Next 14 contain file name.(Places limit of 65535 files)
- Can cause inefficiencies if directory grows large.

Inode:
- each file has an inode assosiated with it.
- contains meta data of the file
- in memory inode(struct inode)
- on disk data inode(struct dinode) - when file is open kernel stores data from disk copy of inode into inmemory data structure.
	- Divided into several bit fields.
	- IFREG - Regular file
	- IFDIR - directory
	- IFBLK - Block devicec
	- IFCHR - character device
	- the 9 lower indicate specify read,write execute,members of owner's group
 - di_addr - When file grows kernel allocates new blocks from any convinenent location on the disk. Easy to grow and shrink without the disk fragmentation ingeritent.



Unix solution is to store small list in the inode itself and use extra blocks for large files.
- Efficient for small files
- Flexible enough to handle large files.

The 13 element array each storing 3 byte physical block number.
- 10 - block number of indirect block
- 11 - double indirect block
- 12 - Triple indirect block

Unix may contain holes. 
- can be large ,spanning entire blocks , wasteful to allocate space for such block.
- kernel sets corresponding elements of di-addr array of these to zero.
- Zero filled block is read when trying to read
- Can cause issues when copying files, run out of disk space

SUPER BLOCK
- contains meta data about file system itself.
- One superblock for each file system.
- resides at beginning of file system on disk
- Contains:
	- Size in blocks of the file system
	- Size in blocks of inode list
	- Number of free blocks and inodes.
	- Free Block list
	- Free Inode list
- File system must maintain a complete list of free blocks in the disk
![[{CC7BB409-11D4-4AC2-B992-093D7A4E7C5F}.png]]
- Contains first part of list and adds and removes blocks from its tail. First element in this list points to the block containing the next part of the list and so forth

Incore inode - all fields of on disk inodes + additional fields
 - The vnode
 - Device ID of the partition containing the files.
 - Inode number of the file.
 - Flags for synchronization and cache management
 - Pointers to keep inode on free list
 - Pointers to keep inode on hash queue.
 - Kernel hashes by their inode numbers , to locate equickly when needed
 - Block number of the last block read.

Disk block array is handled differently. While di_addr[] array in the on disk inode used three bytes for each block number ,, incore inode uses four - byte elements.

Inode lookup
lookuppn() - does pathname parsing. Parses one component at a time, Invoking VOP_LOOKIP operation.
When searching uses translates to call S5lookup() first checks the directory lookup cache. In case of cache miss reads the directory one block at a time, searching entries for the specified file name.
![[{DBF1B3C8-BC92-46C7-970D-F50755365249}.png]]

- If directory contains valid entry of file s5lookup() obtains inode number of the entry
- Calls iget() to locate the inode , tries hashing to find the appropriate hash queue
	- if not in table iget() allocates an inode and initializes it by reading on-disk inode.
	- while copying ondisk inode to incore inode expands di-addr to 4 bytes each.
	- Then puts it into appropriate hash queue.
	- Also initialises the vnode.
	- Finally returns a pointer to the inode to s5lookup().
- Simply put when creating new file
	- s5create() - allocates unused inode number
	- iget() - bring that inode into memory

FILE I/O
- read and write both accept file descriptor (index returned by open), user buffer address that is the destination(for read) or source(for write) + count(number of bytes to be transferred)
- Offset of the file is obtained from the open file object assosiated with the descriptor.
- At end of I/O operations offset is advanced by the number of bytes transferred.
- Read or write begin where previous one completed. For random i/o user must first call lseek to set file offset to the desired location.
File system independent code uses descriptor as index into descriptor table
- Obtain the pointer to open file object and verifies that the file is opened in correct mode.
- If kernel obtains the vnode pointer from file structure.

- Kernel invokes VOP_RWLOCK to serialize access to file.
- S5fs implements this by acquiring exclusive lock on inode. All data read or written in single system call is consistent and all write calls to the file are single threaded. 
	- In earlier file I/O routines used the block buffer cache an area of memory reserved for file system blocks.
s5read() - translates starting offset for the I/O operation to the logical block number in the file and the offset from the beginning of the block.
- Reads the data one page at a time. by mapping block into kernel virtual address space and calling uiomove() to copy data into user space which calls copyout() routine.
- In s5fs this opertion is implemented by s5getpage() which calls bmap() to convert logival block number to physical block number on the disk. searches vnoce's page list to see if page is already in memory.

- Calling process sleeps until the I/O is complete. when block is read disk driver wakes up the process which resumes data copy in copyout().
- s5read() returns when all data has been read or if an erro has occured.
Write works similary:
- modified blocks are not written immediately to disk but remain inmemory.
- Only part of block is being written, kernel first read the entire block from disk modyfing the relevant part and write back to the disk

Allocating and reclaiming Inodes:
- Inode remains active as long as vode has non - zero refernce count
- When count drops to zero invokes VOP_INACTIVE operation, which frees the inode
- When inode becomes inactive kernel puts it into free list and does not invalidate it.
Each file system has fixed size inode table that limits the number of active inodes in the kernel.

When file is actively used , its inode is pinned in the table. As file is accessed , its pages are cached in memory. When file becomes inactive some of its pages may be still in memory.
- If kernel reuses the inode after inactive the pages lose their identity. 
- If process needs the page, kernel must read it back from the disk ,even through the page is still in memory.
- Hence better to reuse those kernels that have no pages cached in memory.
- When vnode reference count reaches zero, kernel invokes its VOP_INACTIVE to release the vnode and it's private data object.
- When releasing inode to kernel checks the vnode's page list
	- releases the inode to the front of the free list if the page list is empty and to the back of the free list if any pages of the file are still in memory.
-BARK METHOD(SKIP??)
- new inode allocation and reclaim policy , allows the number of in-core inodes to adjust to load on the system.
- When Iget() cannot locate an inode on its hash queue, it removes the first inode from the free list. If this inode still has pages in memory - returns it to the back of the free list and calls the kernel memory allocator to allocate a new inode structure.

ANALYSIS:
- Distinguished by simple design .
- Simplicity creates problems in reliability,preformance and functionality.
- Major concern is SUPERBLOCK ,contians vital informationabout the entire file system .Each file system contains single copy of its superblock if that copy is corrupted the entire file system becomes unusable.

	PERFORMANCE ISSUES:
	- groups all inodes at the start of file system and remaining disk contains data blocks.
	- accessign file requires reading the inode and file data ,this causes long seek on the disk between 2 operations and hence increases I/O times. 
	- Inodes are allocated randomly
	- Disk block allocation is suboptimal . When files are created and deleted blocks are returned to the list in random order. Slows down sequential access to files
	- Disk block size is another concern that affects performance. Need flexible approach to allocating space to files.
	-
	FUNCTIONALITY LIMITATIONS:
	- restricting file characters to 14 charater have mattered today
	- Limit of 65535 inodes per file system is also too restricitive.

# BERKLEY FAST FILE SYSTEM

Major difference than s5fs include disk layout, on disk structure and free block allocation methods.

Hard Disk Structure
![[{DA0165C1-E6EF-4591-9A5B-0C954683A722}.png]]
Old harddisk study :
- Typical hard disk old ones.
	- Disk is composed of several platters each associated with disk head.
	- Each platter contains sever tracks that form concentric circles.
	- Each track is further divided into sectors, and sectors are also numbered sequentially.
	- Sector size typically 512 bytes define granulrity of disk I/O operations.

Unix views the disk as linear array of blocks. The number of sectors in a block is small power of 2 .
When UNIX wants to read particular block number , device driver translats that into a logical sector number and from it computes the physical track,head and sector number.
- Head seek is the most time consuming component of the disk I/O operation and the seek latency depends on how far the head moves.
- Must wait until heads move into position . Once the head moves into the position we must wait until the disk correct sector passes under the disk head. This delay is rotational delay.

### On Disk Organization
Disk partition comprises of a set of consecutive cylinders on the disk and formatted partition holds a self contained file system. FFS further divides the partition into one or more cylinder groups each containing small set of consecutive cylinders.
Allows UNIX to store related data in same cylinder group thus minimizing disk head movements.

BFFS contains information about entire file system:
- Number,sizes and locations of cylinder groupds,block sizes, total number of blocks and inodes and so forth. 
- Data in superblock does not change unless file system is rebuilt.
- Data in superblock is critical and must be protected from disk errors. Each cylinder group therefore contains duplicate copy of the superblock at different locations

#### Blocks and Fragments
Large block size improves performance by allowing more data to be transferred in single I/O operations.
- wastes more space
- FFS divides blocks into fragments
- Even though all blocks must be of same size
	- different file systems on the same machine have different block sizes.
	- Block size is power of two greater than or equal to minimum of 4096.
	- Most implementation have upper limit of 8192 bytes.
	- Much larger than 1024
- FFS does not use triple indirect block ,some use to support file size greater than 4 gb.

4k block wastes too much space for small size.
Converted into fragments ,the size is fixed for each file system and is set when the file system is created. May be set to 1,2,4,8 allowing lower bound of 512.
- Require bitmap that tasks each fragment.

- FFS file is composed entirely of complete disk blocks ,except for last block which may contain one or more consecutive fragment blocks.
- File block must completely contained within a single disk block.
- Even if adjacent disk blocks have enough consecutive free fragments to hold a dile block they must not be combined.
- This scheme reduces space wastage but requires occasional recopying of file data. Consider a file who last block occupies a single fragment.
- Example where file grows and fragment needs to find block where it can store this.

For best performance applications should write a full block at a time to files whenever possible.
Different file systems on same machine may have different block sizes.

Allocation Policies:
In s5fs the superblock contains one list of free blocks and another of free inodes , elements are simply added to or removed from the end of the list. Order is not strictly imposed, after sometime the oder becomes random.

FFS aims to colocate related information on disk and optimize sequential access . Provides greater degree of control on the allocation of disk blocks and inodes as well as directories.
The following rules summarize these policies.
- Attempt to place inodes of all files of a single directory in same cylinder group.
	- ls-l access all inodes of directory in rapid succcession . Users also tend to exhibit locality of access, working on many files in the same directory before moving to another.
- Create each new directory on different cylinder gorup from it's parent , so as to distribute data uniformly over the disk. Chooses new cylinder group form groups with above average free inode count. From these selects one with fewest directories.
- Try to place data blocks of file in the same cylinder group as the inode because typically the inode and data will be accessed together.
- To avoid fitting an entire cylinder group one last file, change the cylinder group when the file size reaches 48 kilobytes and again at every megabyte. 48 kilobyte mark was chosen because for 4096 byte block size, the inodes direct block entries describe the first 48 kilobytes.

The implementation must balance the localization efforts with need to distribute data throughout the disk. Too much localization causes all data to be crammed .
Implementation is highly effective when plenty of free space, but deterioratges rapidly once the disk is about 90% full. Becomes difficult to find free blocks in optimal locations.

Functionality Enhancement:
Since different on disk organisation and both not being compatible designers introduced other functional changes.
LONG FILE NAMES: ffs chagned the directory structure to allow file names greater than 14 character . The fixed part of entry consists of inode number, allocation size and size of the filename in the entry. Followed by null terminated filename padded to 4 byte boundary. The maximum size of the filename is currently 255 characters. but when deleting filename ffs merges the released space with previous entry. Hence the allocation size field revords total space consumed by variable part of the entry. 
![[{79A6A45A-890A-4275-BAD0-F0EC56026B7A}.png]]

Symbolic Links:
Address many limitations of hard links . Symbolic link is file that points to another file called target of the link. The type field in the inode identifies the file as a symbolic link and file data is simply the pathname of the target file. The pathnam may be absolute or relative, interpreted relative to the directory containing the link. 
- lstat - which does not translate the final symbolic link the the path name
- readlink 0 returns content of the link
- Created by symlink system call
Other enhancement:
- added a rename system call to allow atomic renames of files and directories, which previously required a link followed by an unlink. Added a quota mechanism to limit the file system resources available to any user. Quotas apply to both inoder

Analysis
- Huge performance gains
- Reduced the disk wastage space by half.
Modern new disk are no longer cylindrical.
FFS is oblivious to this.

FFS has several enhancements since it was first introduced . 4.3 bsd added two types of caches to speed up name lookups.
1) Uses hint based directory name lookup cache which shows hit rate of about 70%name translations.
2) Each process caches the directory offset of the last component of the most recently translated pathname .If next translation of file is of same directory search begins at this point rather than the top of the directory.

Temporary File System
- Temporary files are deleted when the application exits.
- short lived and need not be persistant.
- Should be fast but usually slowed by creation and deletion
1 Idea is to use RAM disks.
- local file system can be built in it using conventional tools like newfs.
- Main disadvantage is :
	- Dedicating large amount of RAM 
	- So need of better implementation

### Memory File system
- The entire file system is built on virtual address of process that handled the mount operation.
- Does not return from mount call but remains in kernal,waiting for I/O operations.
- Each mfsnode contains pid of mount process which now functions as I/O server /
- To perform I/O calling process places a request on a queue that is maintained in mfsdata structure ,wakes up the mount process and sleeps while the request is serviced.
- Mount process satisfies request of caller by copying data from or to the appropriate portion of its address space and awakens the caller.
- Since file system is in virtual memory of mount process can be paged like any other data

### Tmpfs file system
Memory based file system designed to efficiently handle temporary file s using advanced features of vnode/vfs interface 
- tmpfs operates entirely within kernel
- Not rely on seperate server process so no context switches
- Metadata is stored in non paged kernel memory dynamically allocated from the kernel heap
- Data Blocks are stored in paged memory which can be swapped out if need.
- File data is represented using the anonymous page facility of VM subsystem
- Overcome several shortcomings of mfs approach.
- Holding meta data in unipaged kernal memory eliminated memory to memory copies and some disk I/ O . Suport for memory mapping allows fast direct access to file data.


### Special Purpose file systems
RFS and NFS to coexist in one system
##### Specfs File System
Provides uniform interface to device files.Backend system for handling special files and connot be directly interacted with by users.
- Intercepts I/) calls for device files and forwards them to appropriate device drivers
- Handles situations where multiple device file refer to the same physical device by using shadow vnodes and snode
- Synchronises device access and ensures consistency, particularly with block device buffers

/proc file system
- provides interface to process information and control via file system . Intitially developed for debuggin it has expanded to general process manager.
	- Each process is represented by directory named after it's process ID.
	- Contain files and subdirectories like:
		- Status: Contains process's state,stack and heap information
		- psinfo: Process details like size ,controlling tty and memory use
		- ctl: Write only file allows user to perform control operations on the target process.
		- map: Describes virtual address memory of the process . It contains array of prmap structures.
		- as: Maps process's address space, allowing direct access via file system operations like lseek.
		- sigact: Signal handling information . File cotains array of sigaction structures
		- cred:user credentials
		- object: One file for each object mapped into the address space of the process.
		- lwp: One subdirectory for each LWP of the process. Each subdirectory contains 3 files
			- lwpstatus,lwpsinfo,lwpctl
