# Hash Hash Hash
Addition of mutex locking to create a thread-safe program which protects a shared hash table in which the entries are linked lists. 

## Building
```shell
make
```

## Running
```shell
./hash-table-tester -t 8 -s 50000

results:
Generation: 73,181 usec
Hash table base: 343,638 usec
  - 0 missing
Hash table v1: 680,642 usec
  - 0 missing
Hash table v2: 104,169 usec
  - 0 missing
```

## First Implementation
In the `hash_table_v1_add_entry` function, I added one global mutex that would be used by at most one thread at the same time to lock down the entire hash table when adding an entry. I first initialized the mutex using the macro PTHREAD_INITIALIZER, then at the beginning of the add_entry function, I locked the mutex with a call to pthread_mutex_lock(&mutex), similarly at the end of the add_entry function I unlocked the entire hash-table with a call to pthread_mutex_unlock(&mutex);

### Performance
```shell
make
./hash-table-tester -t 8 -s 50000

results:
Generation: 73,181 usec
Hash table base: 343,638 usec
  - 0 missing
Hash table v1: 680,642 usec
  - 0 missing
Hash table v2: 104,169 usec
  - 0 missing
```
Version 1 is a little slower/faster than the base version. This is due to the fact that there is not true concurrency as the threads can only act one at a time, furthermore the pthread lock and unlock calls issue syscalls in order to place and remove threads from the waiting queue which adds a significant amount of time making the v1 slower than the base version. 

## Second Implementation
In the `hash_table_v2_add_entry` function, I created and initialized an array of mutexes with a size equal to the amount of entries in the hash table. Instead of locking the whole hash table at once for each thread, I locked individual entries pointing to the linked list so that other threads that wanted to access other entries could do so in parallel. This array was indexed by the hashing function just as the hash table was indexed. 

### Performance
```shell
make
./hash-table-tester -t 8 -s 50000

result:
Generation: 74,608 usec
Hash table base: 339,525 usec
  - 0 missing
Hash table v1: 669,924 usec
  - 0 missing
Hash table v2: 101,749 usec
  - 0 missing
```

V2 is about 3.4 times as fast as the base and about 6.7 times as fast as v1. This is due to the fact that I am only locking a certain entry at one time, due to this, Other threads can simultaneously access the hash table at different entries and add to those linked lists concurrently, speeding up the overall runtime of the program. Ideally my speedup would be by a factor of the number of cores my machine has... However, due to main overhead being from syscalls when threads do access the same entry. Because of this shared variable and dependency, there cannot be a fully maximized speedup and threads have to wait sometimes. 
## Cleaning up
```shell
make clean
```
