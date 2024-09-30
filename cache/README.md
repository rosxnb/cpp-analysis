# CACHE

Cache is SRAM (static RAM) and can be present in chip itself (L1 cache) or near chip (L3 chip).
They hold contents of recently accessed memory locations. Depending on cache level, caches are 
used per thread (L1 cache), shared between threads of a core (L2 cache), or shared with all cores 
(L3 cache). Also, instructions and data cache may be exclusive especially for L1 cache while some 
cache may hold both instructions and data like L2 and L3 caches.

Three common types of caches are:
- Data Cache
- Instruction Cache
- Translation Lookaside Buffer (caches the result of translating virtual page address to real page address)


**Caches are organized into cache lines**. If you ask for a byte, you get a cache line. Main memory is read/written
in terms of cache lines. Cache line is typically of size 64 bytes. For example: 16 `int32_t` gets loaded when asked for one.

**Cache Associativity** is about slot assignment to cache lines based on the address of the cache line. For a cache associativity 
of 4, every individual address the cache line go to one of four slots.


Small is fast:
- No time/space trade-off at hardware level contradicting algorithmic perspective.
- Compact, well-localized code that fits in cache is fastest.
- Compact data structures that fit in cache is fastest.
- Data structure traversals touching only the cached data are fastest.
- Locality counts: read/write at address A caches contents near A. Hardware may prefetch nearby cache line for predictable linear traversal (ex: constant stride jump).
- For small N, linear array search can beat O( log_2 N ) searches of heap-based Binary Search Tree.
- Big-Oh wins for large N, but hardware caching takes early lead.


To get info about CPU on MacOS:
```zsh
> system_profiler SPHardwareDataType
> sysctl -a machdep.cpu
```

**Notes**:
- [Cache Oblivious Algorithms](https://en.wikipedia.org/wiki/Cache-oblivious_algorithm) is subarea that takes advantages of cache when designing algorithms.

**Examples list:**

- [Cache Line](#example-cache-line)
- [False Sharing](#example-false-sharing)
- [Instruction Branching](#example-instruction-branching)



## Example: Cache Line

**Example Case 1**: Matrix traversal as an example of cache locality

For row major traversal, when you first load data, corresponding cache line blob gets loaded 
closer to CPU so it is faster. For column major, the loaded blob is essentially useless each 
time hence it becomes slower to load new data into cache after flushing current cache data.

**Cache line prefetching:** hardware can detect pattern of access and speculatively prefetch 
cache lines if it thinks you might need. So even backward linear traversal is cache friendly.

**Example Case 2**:

```c++
struct Object {
    bool isAlive;   // can be bit field
    ...
}; // assume size is ≥ 64 bytes

std::vector<Object> objects;
for( auto& o: objects ) {
    if( !o.isAlive ) break;
    // do stuff if object alive
}
```

In above example program, we are checking only 1 bit of 64 bytes cache line and essentially throwing
it is the object is not alive. Such design pattern can be avoided by utilizing Data Oriented Designs 
which are design patters that are hardware cache friendly.


## Example: False Sharing

Hardware tries to get rid of "corrupt" cache line when two cores frequently tries to modify data on same 
cache line they cause false sharing.

False Sharing problem arises when all following holds:
- Independent data falls on one cache line (example: A address for core 1 and A+1 address for core 2).
- Different cores concurrently and frequently acccess that line.
- At least one is writer.

**Example Case**:
    - Problem statement: Count odd numbers in an array.
    - Solution with false sharing:
        - Instantiate an array [Arr] to hold result for each thread.
        - Divide array into strides of sub arrays for each thread.
        - Directly modify [Arr] corresponding thread slot to increment count.
        - After thread work completes, traverse [Arr] and sum total count.
    - Reducing false sharing:
        - Introduce thread local variable counter to count odd number.
        - Only update [Arr] corresponding thread slot onece that is after computing count.


## Example: Instruction Branching

**Example Case**:
    - Array of pointers to Base Class (to load respective derived class methods on condition).
    - Iterate over array of pointers to load derived class method dynamically.
    - Cache is filled with instructions of respective derived class method.
    - Same instructions may need to be flushed and loaded multiple times into the cache.
    - SOLUTION: sort array by the type.

**Profile Guided Optimization** (PGO) is mode in compiler where the compiler keeps track of runtime 
of generated executable. You run that executable on use cases where you want to optimize. You then 
recompile the program along with data collected by compiler for previous executable to produce optimized
executable this time around. *This may not always be effective buy is worth trying*.

**Whole Program Optimization** (WPO) attempts to complement PGO.


# REFERENCES

- [Scott Meyers: CPU Caches and Why You Care](https://www.youtube.com/watch?v=WDIkqP4JbkE&t=138s)
