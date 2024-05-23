# Hash Hash Hash

The goal of this lab is to utilize the POSIX thread library in order to implement a multi-threading safe hash-table.
There are two version of the hash tables implementation, in the first version only one mutex is used in order to demostrate
how you can guarantee atomicity of hash table insertion. In the second version, multiple mutexes are used in order to provide
more concurency while still providing thread safety and improving performance from the single thread implementation.

## Building
```shell
make
```

## Running
```shell
./hash-table-tester -t 8 -s 50000
```
-t changes the number of threads to use (default 4)
-s changes the number of hash table entries to add per thread (default 25,000)

## First Implementation

First, I added a mutex variable `v1_lock` as a field to the `hash_table_v1` struct in order to initialize it upon creation. This is where I called 
`pthread_mutex_init` after it was asserted that the created hash table was not null within the `hash_table_v1_create` function. I then called `pthread_mutex_lock` 
and `pthread_mutex_unlock` at the beginning and end of the `hash_table_v1_add_entry` respectively in order to ensure the retrieval of the hash_table/list entries
and their updating were done atomically within each thread without any interruption at intermediate stages. I also made sure to unlock the mutex 
in the case of the entry already existed and an early return occuring.


### Performance
```shell
Hash table base: 1,037,564 usec
  - 0 missing
Hash table v1: 1,499,786 usec
  - 0 missing
```
Version 1 is a little slower than the base version. The reasoning for this implementations slower performance than the base is because using multiple threads with a single lock can be slower than using a single thread with no locks due to the overhead of acquiring and releasing the lock introuducing significant delays, especially when contention is high. When multiple threads compete for the same lock, they spend time waiting to acquire the lock, causing context switching and synchronization overhead, which can outweigh the benefits of parallelism, resulting in poorer performance compared to a single-threaded approach without locking.


## Second Implementation

Within the second version I removed the global lock on the entire hash table to decrease the contention rates of competing threads. 
In order to support a multi-thread safe table I added a mutex to each bucket of the hash table. For this I added `v2_bucket_lock`
as a field to the `hash_table_entry` struct. Within the creation of the hash_table I had to initialize the lock for each entry upon
a proper list head being created. The calls `pthread_mutex_lock` and `pthread_mutex_unlock` were made on whichever hash_table_entry was returned by the helper function `get_has_table_entry` and after the value was inserted into its entry respectively. While freeing the memory allocated for this hash table, I made sure to call `pthread_mutex_destroy` on each entry after all elements in the entry were freed. 

### Performance
```shell
Hash table base: 1,132,411 usec
  - 0 missing
Hash table v1: 1,630,872 usec
  - 0 missing
Hash table v2: 357,667 usec
  - 0 missing
```

This big performance boost is realized as now upon each call to adding an entry into the table, only the specific entry in the hash table is locked. This allows for other entries to be manipulated in parallel for a much higher throughput while still maintaining a race condition free hash table as can be seen with 0 missing entries resulting.


## Cleaning up
```shell
make clean
```