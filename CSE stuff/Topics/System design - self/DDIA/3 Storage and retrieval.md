
Think about whats more important - analytics or transactions? difference in storage engine optimized for either of them

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

