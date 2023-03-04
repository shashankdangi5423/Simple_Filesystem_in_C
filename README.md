
The layout of Filesystem on a disk is composed of four consecutive logical parts:

- The *Superblock* is the very first block of the disk and contains information
  about the file system (number of blocks, size of the FAT, etc.)
- The *File Allocation Table* is located on one or more blocks, and keeps track
  of both the free data blocks and the mapping between files and the data blocks
  holding their content.
- The *Root directory* is in the following block and contains an entry for each
  file of the file system, defining its name, size and the location of the first
  data block for this file.
- Finally, all the remaining blocks are *Data blocks* and are used by the
  content of files.

The size of virtual disk blocks is **4096 bytes**.

Unless specified otherwise, all values in this specification are *unsigned* and
the order of multi-bytes values is *little-endian*.

### Superblock

The superblock is the first block of the file system. Its internal format is:

| Offset | Length (bytes) | Description                             |
|--------|----------------|-----------------------------------------|
| 0x00   |  8             | Signature (must be equal to "ECS150FS") |
| 0x08   |  2             | Total amount of blocks of virtual disk  |
| 0x0A   |  2             | Root directory block index              |
| 0x0C   |  2             | Data block start index                  |
| 0x0E   |  2             | Amount of data blocks                   |
| 0x10   |  1             | Number of blocks for FAT                |
| 0x11   |  4079          | Unused/Padding                          |

If one creates a file system with 8192 data blocks, the size of the FAT will be
8192 x 2 = 16384 bytes long, thus spanning 16384 / 4096 = 4 blocks. The root
directory block index will therefore be 5, because before it there are the
superblock (block index #0) and the FAT (starting at block index #1 and spanning
4 blocks). The data block start index will be 6, because it's located right
after the root directory block. The total amount of blocks for such a file
system would then be 1 + 4 + 1 + 8192 = 8198.

### FAT

The FAT is a flat array, possibly spanning several blocks, which entries are
composed of 16-bit unsigned words. There are as many entries as *data blocks* in
the disk.

The first entry of the FAT (entry #0) is always invalid and contains the special
EOC (*End-of-Chain*) value which is `0xFFFF`. Entries marked as 0 correspond to
free data blocks. Entries containing a positive value are part of a chainmap and
represent a link to the next block in the chainmap.

Note that although the numbering in the FAT starts at 0, entry contents must be
added to the data block start index in order to find the real block number on
disk.

The following table shows an example of a FAT containing two files:

- The first file is of length 18,000 bytes (thus spanning 5 data blocks) and is
  contained in consecutive data blocks (DB#2, DB#3, DB#4, DB#5, DB#6 and DB#7).
- The second file is of length 5,000 bytes (this spanning 2 data blocks) and its
  content is fragmented in two non-consecutive data blocks (DB#1 and DB#8).

Each entry in the FAT is 16-bit wide.

| FAT index: |      0 | 1 | 2 | 3 | 4 | 5 | 6 |      7 |      8 | 9 | 10 | ... |
|------------|--------|---|---|---|---|---|---|--------|--------|---|----|-----|
|   Content: | 0xFFFF | 8 | 3 | 4 | 5 | 6 | 7 | 0xFFFF | 0xFFFF | 0 | 0  | ... |

### Root directory

The root directory is an array of 128 entries stored in the block following the
FAT. Each entry is 32-byte wide and describes a file, according to the following
format:

| Offset | Length (bytes) | Description                             |
|--------|----------------|-----------------------------------------|
| 0x00   |  16            | Filename (including NULL character)     |
| 0x10   |  4             | Size of the file (in bytes)             |
| 0x14   |  2             | Index of the first data block           |
| 0x16   |  10            | Unused/Padding                          |

An empty entry is defined by the first character of the entry's filename being
equal to 0.

The entry for an empty file, which doesn't have any data blocks, would have its
size be 0, and the index of the first data block be `FAT_EOC`.

Continuing the previous example, let's assume that the first file is named
"test1" and the second file "test2". Let's also assume that there is an empty
file named "test3". The content of the root directory would be:

| Filename (16 bytes) | Size (4 bytes) | Index (2 bytes) | Padding (10 bytes) |
|---------------------|----------------|-----------------|--------------------|
| test1\0             | 18000          | 2               | xxx                |
| test2\0             | 5000           | 1               | xxx                |
| test3\0             | 0              | `FAT_EOC`       | xxx                |
| \0                  | xxx            | xxx             | xxx                |
| ...                 | ...            | ...             | ...                |

### Reference program and testing


- Create a new virtual disk containing an empty file system:
    `$ filesystem.x make <diskname> <data block count>`
- Get some information about a virtual disk:
    `$ filesystem.x info <diskname>`
- List all the files contained in a virtual disk:
    `$ filesystem.x ls <diskname>`
- Etc. in order to have the list of commands:
    `$ filesystem.x`




