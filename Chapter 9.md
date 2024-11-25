
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
![[Pasted image 20241123185239.png]]
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

### **Superblock**

The **superblock** stores metadata about the file system itself. Each file system has one superblock, located at its beginning on disk.

**Contents of the Superblock**:

- Total size (in blocks) of the file system.
- Size of the inode list (in blocks).
- Number of free blocks and inodes.
- Lists of free blocks and free inodes.

**Free Block Management**:

- The file system maintains a complete list of free blocks on the disk.
- The superblock contains the first part of this list and dynamically adds/removes blocks from its tail.
- A linked structure: the first element of the list points to the block containing the next part of the list, continuing in this manner.
![[{CC7BB409-11D4-4AC2-B992-093D7A4E7C5F}.png]]

---

### **Incore Inode**

An **incore inode** extends the fields of an on-disk inode with additional fields for memory-based operations.

**Additional Fields in Incore Inode**:

- The vnode structure.
- Device ID of the partition containing the file.
- Inode number of the file.
- Flags for synchronization and cache management.
- Pointers to manage the inode in the free list and hash queue.
- Block number of the last block read.

**Handling Disk Block Array**:

- In the **on-disk inode**, the `di_addr[]` array uses 3-byte entries.
- In the **incore inode**, 4-byte entries are used for better efficiency.

---

### **Inode Lookup**

The inode lookup process involves:

1. **Pathname Parsing**:
    
    - Function: `lookuppn()`
    - Parses one component of the pathname at a time, invoking the `VOP_LOOKUP` operation.
2. **Directory Lookup**:
    
    - File system-specific operation, e.g., `S5lookup()`.
    - First checks the **directory lookup cache**. If a cache miss occurs, it reads the directory block by block, searching for the specified file name.
3. **Finding or Creating Inodes**:
    
    - If the directory contains the file entry, `S5lookup()` retrieves its inode number and invokes `iget()` to locate the inode.
        - If the inode is not in memory:
            - `iget()` allocates an incore inode and initializes it by reading the on-disk inode.
            - Expands the `di_addr[]` array to 4 bytes per entry.
            - Adds the inode to the appropriate hash queue and initializes the vnode.
    - For a new file, `s5create()` allocates an unused inode number, and `iget()` loads it into memory.
![[{DBF1B3C8-BC92-46C7-970D-F50755365249}.png]]
---

### **File I/O**

Both **read** and **write** operations use a file descriptor, a user buffer, and a byte count.

#### **Key Process**:

1. The **file offset** is obtained from the open file object associated with the descriptor.
2. After the operation, the offset advances by the number of bytes transferred.
3. Sequential operations begin where the previous one left off; random access requires a call to `lseek()` to set the desired file offset.

#### **File System-Independent Operations**:

- The kernel uses the file descriptor as an index into the descriptor table to obtain the open file object.
- It verifies the file's open mode and retrieves the vnode pointer.
- The kernel invokes `VOP_RWLOCK` to serialize access to the file, ensuring consistency and single-threaded writes.

---

### **S5FS-Specific Read and Write**

#### **Read Operation**:

- `s5read()`:
    
    1. Translates the starting offset into a **logical block number** and offset within the block.
    2. Reads data **page by page**, mapping the block into kernel virtual memory.
    3. Copies data to user space using `uiomove()`, which calls `copyout()`.
    4. The `s5getpage()` function invokes `bmap()` to map logical block numbers to physical block numbers.
- **Caching**:
    
    - Before reading from the disk, `s5read()` checks if the required page is already in the vnode's page list.
- **Synchronization**:
    
    - The calling process sleeps during the I/O operation.
    - The disk driver wakes the process upon I/O completion, allowing the process to resume.
- The operation concludes when all data has been read or an error occurs.
    

#### **Write Operation**:

- Modified blocks are not immediately written to disk; instead, they are cached in memory.
- For partial block writes:
    1. The kernel reads the full block from the disk.
    2. Modifies the relevant portion in memory.
    3. Writes the updated block back to disk when necessary.

---

### **Efficiency in File I/O**

- By caching frequently accessed blocks and deferring writes, the file system minimizes disk I/O, improving performance.
- The linked structure of free block lists and the expanded `di_addr[]` array in incore inodes ensure fast access and updates to file system metadata.

## File I/O Operations

### Basic Characteristics
- Read and write operations accept:
  - File descriptor (index returned by open)
  - User buffer address (destination for read, source for write)
  - Count of bytes to be transferred

- File offset is obtained from the open file object associated with the descriptor
- After I/O operations, the offset advances by the number of bytes transferred
- For random I/O, use `lseek()` to set the file offset to the desired location

### Kernel File Handling
- File system independent code uses descriptor as an index into the descriptor table
- Verifies file is opened in the correct mode
- Obtains vnode pointer from file structure
- Invokes `VOP_RWLOCK` to serialize access to the file

#### S5FS Implementation
- Acquires exclusive lock on inode
- Ensures data read or written in a single system call is consistent
- Writes are single-threaded
- Previously used block buffer cache for file system blocks

## Read Operation (s5read())

### Process
- Translates starting offset to logical block number and block-specific offset
- Reads data one page at a time
- Maps block into kernel virtual address space
- Uses `uiomove()` to copy data into user space via `copyout()`

### S5FS Specific Implementation
- Uses `s5getpage()` to convert logical block number to physical block number
- Searches vnode's page list to check if page is in memory
- Calling process sleeps until I/O is complete
- Disk driver wakes up the process to resume data copy

### Write Operation
- Modified blocks are not immediately written to disk
- For partial block writes:
  - Kernel reads entire block from disk
  - Modifies relevant part
  - Writes back to disk

## Inode Management

### Allocation and Reclamation
- Inode remains active while vnode has non-zero reference count
- When reference count drops to zero:
  - Invokes `VOP_INACTIVE` operation
  - Frees the inode
  - Puts inode into free list without invalidating it

### Inode Caching
- Each file system has a fixed-size inode table
- Actively used file's inode is pinned in the table
- File pages are cached in memory
- When file becomes inactive, some pages may remain in memory

### Inode Reuse Policy
- When vnode reference count reaches zero:
  - Kernel releases vnode and private data object
  - Checks vnode's page list
  - Releases inode to front of free list if no pages cached
  - Releases inode to back of free list if pages remain in memory

## Performance and Functional Limitations

### Performance Issues
- Inodes grouped at file system start
- Requires long disk seeks between inode and data block reading
- Random inode and block allocation
- Suboptimal disk block allocation
- Fixed disk block size limits flexibility

### Functional Limitations
- File name character limit of 14 characters
- Maximum of 65,535 inodes per file system

## Analysis
- Simple design
- Simplicity creates challenges in:
  - Reliability
  - Performance
  - Functionality

### Critical Concern: Superblock
- Contains vital file system information
- Single copy per file system
- Corruption renders entire file system unusable




# BERKLEY FAST FILE SYSTEM

Major difference than s5fs include disk layout, on disk structure and free block allocation methods.
# Old Hard Disk Architecture

## Physical Structure
- Composed of multiple platters, each with an associated disk head
- Platters contain concentric tracks
- Tracks divided into sequentially numbered sectors
- Typical sector size: 512 bytes (defines disk I/O granularity)
![[{DA0165C1-E6EF-4591-9A5B-0C954683A722}.png]]

## Disk Access Characteristics
### Disk I/O Performance Factors
- **Head Seek**: Most time-consuming component of disk I/O
  - Seek latency depends on head movement distance
- **Rotational Delay**: 
  - Wait for heads to position correctly
  - Additional wait for correct sector to pass under disk head

## Unix Disk Representation
- Views disk as a linear array of blocks
- Block size: Small power of 2
- Device driver translates block number to:
  - Logical sector number
  - Physical track
  - Head number
  - Sector number

## Disk Partition Organization
### Partition Structure
- Comprises consecutive cylinders
- Formatted partition contains a self-contained file system
- Fast File System (FFS) divides partition into cylinder groups
  - Stores related data in same cylinder group
  - Minimizes disk head movements

## Superblock Characteristics
### Critical File System Information
- Contains details about:
  - Number, sizes, and locations of cylinder groups
  - Block sizes
  - Total number of blocks and inodes
- Superblock data remains static unless file system is rebuilt
- Critical information requiring protection

### Superblock Redundancy
- Duplicate copies in different cylinder group locations
- Protects against potential disk errors

## Block and Fragment Management
### Block Size Considerations
- Larger block size improves I/O performance
- Allows more data transfer in single operation
- Potential space wastage

### Fragment System (FFS)
- Divides blocks into smaller fragments
- Block size characteristics:
  - Power of two (minimum 4096)
  - Typical maximum: 8192 bytes
  - Different file systems can have different block sizes

### Fragment Details
- Fixed size set during file system creation
- Possible fragment sizes: 1, 2, 4, 8
- Lower bound: 512 bytes
- Requires bitmap to track fragments

### Fragment Allocation Rules
- File composed of complete disk blocks
- Last block may use partial fragment
- Adjacent free fragments cannot be combined
- Requires occasional file data recopying

### Performance Recommendation
- For optimal performance, write full blocks when possible

## Additional Notes
- No support for triple indirect blocks in some implementations
- 4k block size can waste significant space for small files
- File block must be entirely contained within single disk block


# Allocation Policies in File Systems

## S5FS Allocation Approach
- Superblock contains single lists for:
  - Free blocks
  - Free inodes
- Elements added/removed from list ends
- List order becomes random over time

## Fast File System (FFS) Allocation Strategies

### Key Policy Objectives
- Colocate related information
- Optimize sequential access
- Provide greater control over disk block and inode allocation

### Specific Allocation Rules

#### Inode Placement
- Place inodes of files in same directory within same cylinder group
- Supports rapid `ls -l` access
- Leverages user access locality patterns

#### Directory Distribution
- Create new directories in different cylinder groups from parent
- Select cylinder groups with:
  - Above-average free inode count
  - Fewest existing directories

#### Data Block Placement
- Position file data blocks in same cylinder group as corresponding inode
- Optimizes combined inode and data access

#### Cylinder Group Transition
- Change cylinder group at specific file size milestones:
  - Initial transition at 48 kilobytes
  - Subsequent transitions every megabyte
- 48 kilobyte mark chosen to match direct block entries for 4096-byte blocks

### Implementation Considerations
- Balance localization with data distribution
- Effectiveness diminishes as disk approaches capacity
- Performance degrades significantly after 90% disk utilization

## Functionality Enhancements

### Long Filename Support
- Expanded directory structure in FFS
- Directory entry components:
  - Inode number
  - Allocation size
  - Filename length
- Filename characteristics:
  - Maximum 255 characters
  - Null-terminated
  - Padded to 4-byte boundary
- Space optimization through entry merging on filename deletion
![[{79A6A45A-890A-4275-BAD0-F0EC56026B7A}.png]]

# File System Enhancements

## Symbolic Links
### Characteristics
- Points to target file via pathname
- Pathname can be absolute or relative
- Inode type field identifies as symbolic link
- File data contains target file's pathname

### Key Operations
- `lstat`: Does not translate final symbolic link
- `readlink`: Returns link content
- Created via `symlink` system call

## Performance and Space Improvements
- Atomic file/directory renaming
- Quota mechanism for user resource limitation
- Performance gains
- Reduced disk space wastage

## Directory Lookup Optimizations (4.3 BSD)
1. Hint-based name lookup cache
   - 70% name translation hit rate
2. Process-level directory offset caching
   - Speeds up subsequent file translations

## Temporary File Systems

### RAM Disk Approach
- Limitations:
  - Large RAM dedication
  - Inefficient implementation

### Memory File System (MFS)
- Built on process virtual address
- Kernel-based I/O handling
- Uses mount process as I/O server
- Supports paging

### Tmpfs File System
- Kernel-based temporary file system
- No separate server process
- Metadata in non-paged kernel memory
- Data blocks in paged memory
- Uses VM subsystem's anonymous page facility

## Special Purpose File Systems

### Specfs File System
- Uniform device file interface
- Intercepts device I/O calls
- Manages multiple device file references
- Ensures device access consistency

### /proc File System
#### Purpose
- Process information and control interface
- Originally for debugging
- Expanded to process management

#### Structure
- Process ID-named directories
- Contains files/subdirectories:
  - `status`: Process state
  - `psinfo`: Process details
  - `ctl`: Process control
  - `map`: Virtual memory description
  - `as`: Address space mapping
  - `sigact`: Signal handling
  - `cred`: User credentials
  - `object`: Mapped objects
  - `lwp`: Lightweight process details