
Think about whats more important - analytics or transactions? there are differences in storage engine optimized for either of them

Log structured and Page structured (like B trees) storage engines

# Index
Datastructure to handle searching better.
its derived from the primary data.

indexes are an overhead, they are obviously slower than just writing to a file, but then again, if searching data is important, you need an index, especially if your expected data size is big

indexes are an overhead to writes, they slow writes down. every time data is written, indices are updated.

its a trade off
-> good indexes speed up reads, but slow down writes.

----
## Hash indexes

![[Pasted image 20251128122547.png]]

key value pair, each key is mapped to a position in the hashmap.

for value x, check memory offset, seek that offset, read. 
bitcask is essentially this^

its in memory, so we must ensure the hashmap is small enough to fit in the RAM. the data itself could be bigger than the RAM, because we can always make a disk read request.

usecase
- situations where value keeps updating. like view count of a video.

know that we were only appending to this hashmap. if the map grows big, we need to do compaction
![[Pasted image 20251128122939.png]]

---
## drawbacks of hash index
- if the the map grows too big, we may have to store it on the disk, but then it wont perform well, because we will need a lot of random access. 
- range queries will require iterating over all they keys, ie, keys from key0000 to key9999, ill need to iterate and look up each individual key

---
---

# SSTables and LSM Trees

Take what we were doing above in hash tables, but now sort by key.
But wont this break sequential writes?

Sorted string table, we will ensure that each key occurs only once (compaction ensures this already)

advantages over has indexes
- because segments are sorted by key, the compaction process becomes easier, we can implement a merge sort like thing, and just pick the most recent key
- ![[Pasted image 20251206004146.png]]
- we can have SPARSE INDEX
	- ![[Pasted image 20251206004217.png]]
	which means, if we know position of A and position of C, then we know B will be in between. so just jump to A, and start scanning till C. 
	we can have one key every few kilobytes, because KBs are scanned v quickly


How do we get our data sorted by key?

- ## RED BLACK TREES OR AVL TREES
(B Trees are maintained in disk, the above two are in memory ?)

---
_from perplexity_
## B-Trees for Disk

B-trees feature wide nodes with multiple children and keys, minimizing disk seeks by packing more data per block during reads or writes. This structure suits databases and file systems like SQLite, where data exceeds RAM and access times dominate performance. They reduce tree height and I/O operations compared to binary trees on disk.[](https://stackoverflow.com/questions/2734692/avl-tree-vs-b-tree)

## AVL Trees for Memory

AVL trees are self-balancing binary search trees that prioritize fast lookups, insertions, and deletions with strict O(log n) height via rotations, ideal for RAM-resident data. They appear in scenarios like database indexing or in-memory caches needing predictable performance, though they use extra space for balance factors. Red-black trees often replace them in libraries for slightly better balance flexibility.[](https://www.geeksforgeeks.org/dsa/introduction-to-avl-tree/)​

---

One catch - because we are using an in memory datastructure (AVL etc), if the system crashes, then that memory is lost. so we should write through to disk as well (does not need to be sorted etc, just log it because we just care about recovery)

# Making LSM tree out of SSTables
(log structured merge tree)

```
the basic idea of LSM-trees—keeping a cascade of SSTables that are merged in the background
```

2 ways of compaction
![[Pasted image 20251206011845.png]]
![[Pasted image 20251206011907.png]]
The LSM tree is fundamentally composed of two or more components that hold sorted data:

1. **Memtable (or C0 Component):**
    
    - This is an in-memory data structure (often a skip list or balanced tree) that holds the most recent writes.
        
    - **All new writes/updates go here first.**
        
    - Writes are very fast because they are to memory.
        
2. **SSTables (Sorted String Tables) or Levels (C1, C2, Ck...):**
    
    - When the Memtable reaches a certain size, it is flushed to disk as a new, immutable SSTable file.
        
    - These disk-based files are always **sorted by key** and are often compressed.
        
    - The disk-based part is usually organized into multiple **levels** (L0, L1, L2, etc.) where files are progressively merged into larger, more stable levels through a process called **Compaction**.

Here are the step-by-step notes for both compaction strategies, formatted for easy copying and pasting.

---

### **1. Leveled Compaction Strategy (LCS)**

Primary Goal: Optimize for Fast Reads.

Used By: LevelDB, RocksDB (default).

- **Step 1: Ingestion (Memtable Flush)**
    
    - Data is flushed from the Memtable to **Level 0 ($L_0$)**.
        
    - **Note:** $L_0$ is the _only_ level where files can contain overlapping key ranges (because they are direct dumps from memory).
        
- **Step 2: Triggering Compaction ($L_0 \to L_1$)**
    
    - When $L_0$ reaches a specific file count (e.g., 4 files), compaction is triggered.
        
    - The system picks files from $L_0$ and merges them into **Level 1 ($L_1$)**.
        
- **Step 3: Merging Levels ($L_i \to L_{i+1}$)**
    
    - When Level $i$ exceeds its size limit (e.g., 100MB), the system picks one file from $L_i$.
        
    - It identifies **all** files in the next level ($L_{i+1}$) that have an **overlapping key range**.
        
    - It performs a "Multi-way Merge Sort" on these files.
        
- **Step 4: Cleanup & Replacement**
    
    - A new set of SSTables is created in $L_{i+1}$.
        
    - The old files in $L_i$ and the old overlapping files in $L_{i+1}$ are deleted.
        
    - **Crucial Outcome:** All files in $L_{i+1}$ now have **disjoint (non-overlapping) key ranges**.
        
- **Step 5: Read Operation (The Payoff)**
    
    - To find a key, the system checks $L_0$ (all files).
        
    - For $L_1$ and deeper, it checks **only one file per level** (using the file's key range metadata).
        
    - This minimizes disk I/O, making reads very fast.
        

---

### **2. Size-Tiered Compaction Strategy (STCS)**

Primary Goal: Optimize for Fast Writes.

Used By: Cassandra, ScyllaDB (default).

- **Step 1: Ingestion (Tier Creation)**
    
    - Data is flushed from the Memtable to a new SSTable file.
        
    - This file is placed in a "Tier" (bucket) corresponding to its size (e.g., the **Small Tier**).
        
- **Step 2: Accumulation**
    
    - New flushes create more files in the Small Tier.
        
    - Existing files are **never** modified; they simply pile up.
        
    - **Note:** Key ranges **overlap** across all these files (the same key can exist in 5 different files in the same tier).
        
- **Step 3: Triggering Compaction (The Merge)**
    
    - When a specific tier accumulates a threshold number of files (e.g., 4 files of similar size), compaction triggers.
        
- **Step 4: Promotion**
    
    - The system takes those 4 files and merges them into **one single, larger file**.
        
    - This new large file is moved to the **Next Largest Tier** (e.g., 4 x 100MB files become 1 x 400MB file).
        
    - The 4 old small files are deleted.
        
- **Step 5: Read Operation (The Cost)**
    
    - To find a key, the system must potentially check **many files** in **every tier**.
        
    - It must read all versions of the key found to identify which one has the latest timestamp (Last Write Wins).
        
    - This results in "Read Amplification" (more work during reads).
        

---

### **Quick Comparison Cheat Sheet**

| **Feature**            | **Leveled (LCS)**              | **Size-Tiered (STCS)**                |
| ---------------------- | ------------------------------ | ------------------------------------- |
| **Optimized For**      | **Reads** (and Space)          | **Writes** (Insert Heavy)             |
| **Key Overlap**        | No overlap (except L0)         | Overlap everywhere                    |
| **Compaction Trigger** | Level Size Limit               | Number of Similar Sized Files         |
| **Read Cost**          | Low (Check 1 file/level)       | High (Check many files)               |
| **Write Cost**         | High (Frequent re-merging)     | Low (Files merge infrequently)        |
| **Space Wasted**       | Low (Old data deleted quickly) | High (Old data stays until big merge) |
google's big table paper
https://static.googleusercontent.com/media/research.google.com/en//archive/bigtable-osdi06.pdf

#note lucene, the search engine used in elasticsearch, uses something similar to LSM storage
- for every word, find all the document IDs for the documents that contain the given word. this mapping of word -> array of docIDs is kept in SSTable-like sorted files. merged in background when needed.

### performance optimisations
LSM tree algo can be slow when findings keys that dont exist
- check the memtable, then find all the segments, if not found, then go to L0 SSTable
- L0 will have the latest data that was flushed recently from memtable. search in L0, it will have duplicates as it is not compacted yet.
- search L1, L2... (no duplicates, but gotta search all).

Bloom filters are a probabilistic magical DS thing, that tells you if a key is not there (no false negatives), but it cant tell you for sure if a key is there (false positives are possible)

---

# B trees
the most common indexing structure

they also keep the keys sorted like sstables, but the page size is fixed. each block is of fixed size. (thats where the similarity ends)
![[Pasted image 20251230235946.png]]


btrees break database down into fixed size blocks or pages (usually 4KB)
read or write one page at a time
notice how this corresponds to the hardware, as disks are also in fixed size chunks

number of references at each level (at each page) is called the branching factor. in the above image, it is 6

if insertion of a key makes a page bigger than the fixed page size, then we split the page into two 
![[Pasted image 20251231000316.png]]

the tree remains balanced, height of O(LogN)

```
A four-level tree of 4 KB pages with a branching factor of 500 can store up to
256 TB.
```


![[Pasted image 20251231011912.png]]


pasted from gemini. read from book later
```
## OLTP vs. OLAP

As applications grow, we split database work into two categories:

- **OLTP (Online Transaction Processing):** High volume of small requests (e.g., "User 123 changed their password"). Focus: **Low Latency.**
    
- **OLAP (Online Analytical Processing):** Lower volume, but huge queries (e.g., "Total revenue in Q3 across all stores"). Focus: **Throughput.**
    

---

## 4. Data Warehousing & Schemas

A **Data Warehouse** is a separate database containing a read-only copy of all your OLTP data, optimized for analysis.

- **Star Schema:** A central **Fact Table** (huge, contains individual events like "a sale") surrounded by **Dimension Tables** (metadata like "who bought it," "which store").
    
- **Snowflake Schema:** A Star Schema where dimension tables are further broken down (normalized) into sub-dimensions (e.g., _Product_ → _Brand_).
    

---

## 5. Column-Oriented Storage (The OLAP Game Changer)

In OLTP, data is stored **row-by-row**. In OLAP, we use **Column-Oriented Storage**.

- **The Idea:** Store all values for Column A together, then all values for Column B, etc.
    
- **Why it works:**
    
    1. **I/O Efficiency:** If you only need to calculate `SUM(price)`, you only read the `price` column file from disk, skipping names, IDs, and emails.
        
    2. **Compression:** Because values in a column are similar (e.g., a column of country names), you can use **Bitmap Encoding** or **Run-length Encoding** to shrink the data massively.
        
    3. **Vectorized Processing:** CPUs can process "vectors" of data from these columns much faster than individual rows.
        

## 6. Pre-computing Results

To make massive queries even faster, databases use:

- **Materialized Views:** A query result that is physically saved to disk and updated as data comes in.
    
- **Data Cubes:** A multi-dimensional materialized view. It pre-calculates every possible total (e.g., Sales by Product AND Sales by Store AND Sales by Date). It makes dashboarding instant but makes data ingestion much slower.
```