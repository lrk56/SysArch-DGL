# HW2 — Submission Template

This is the only file you need to fill out and submit for HW2. Copy it,
fill in your answers directly below each question, and turn in the
completed copy. You don't need to touch or resubmit `HW2.md`.

For multiple-choice items, mark your choice by changing `[ ]` to `[x]`.
For the Feedback section, several items just ask you to enter a single
number, see the instructions there.

---

# Part A: Multiple Choice (5 pts each, 20 pts total)

**MC1.** Why can column-major traversal of a row-major-stored array run much slower even when it performs the same number of arithmetic operations and logical loads?

- [ ] A. Column-major traversal uses a slower addition instruction
- [x] B. Its access pattern makes much poorer use of data brought into the cache, causing more costly interaction with the memory hierarchy
- [ ] C. The CPU has to execute every column-major instruction twice
- [ ] D. Row-major traversal skips some array elements

**MC2.** What is a cache line?

- [ ] A. A single variable stored in a CPU register
- [x] B. A contiguous block of memory transferred and cached together
- [ ] C. A dedicated wire connecting the CPU directly to RAM
- [ ] D. A compiler-generated marker identifying loops

**MC3.** What does **spatial locality** describe?

- [ ] A. A program repeatedly executing the same arithmetic instruction
- [x] B. A program being likely to access memory locations near one it recently accessed
- [ ] C. A processor keeping frequently used variables only in registers
- [ ] D. Two processors accessing memory at the same time

**MC4.** Why can't a compiler safely reorder every column-major loop into a row-major loop?

- [ ] A. Compilers aren't allowed to change loops
- [x] B. Changing iteration order can change program behavior when dependencies, aliasing, data layout, or other constraints are present
- [ ] C. Compilers don't know that caches exist
- [ ] D. Row-major traversal is only valid in assembly language

---

# Part B: Show What You Learned

### N1 — The corrected model (20 pts)

In 2–3 sentences, correct this statement: "If two programs perform the same operations the same number of times, they should take roughly the same amount of time."

> Your answer:
> This statement ignores the fact that operations do not all cost the same amount and that memory access patterns do matter. Two programs can execute the same instructions the same number of times, but differ massively in runtime if one has good locality, meaning theres mostly cache hits, and the other keeps missing the cache and waiting on main memory, which can be substantially slower than a cache hit. Other factors such as branch prediction, instruction-level parallelism, and data layout can also cause large differences in execution time between two programs.

### N2 — Put real numbers on it (15 pts)

- **Processor / processor family:** AMD Ryzen 9 7950X (Zen 4)
- **Fastest cache level you investigated:** L1 data cache (L1D)
- **Approximate access cost:** About 4 CPU cycles, or 0.7 nanoseconds at a 5.7 GHz clock. AMD specifies a 4-cycle integer load-to-use latency, and independent testing measured a similar result.
- **Slower cache level or main memory:** Main memory (DRAM) after the requested data misses in the caches.
- **Approximate access cost:** About 73.35 nanoseconds in a test using a Ryzen 9 7950X with DDR5-6000 memory and a 1 GB test size. AMD's optimization guide does not provide a DRAM latency figure.
- **What surprised you about the comparison:** I was surprised that the measured DRAM access took about 100 times as long as an L1 cache hit. This helped me understand why a program with the same number of loads and additions can run much more slowly when its memory access pattern causes more accesses to reach main memory. The exact ratio varies by workload and hardware.
- **Sources:**
- AMD, Software Optimization Guide for the AMD Zen 4 Microarchitecture, publication 57647, §§2.6.2–2.6.4: https://www.amd.com/content/dam/amd/en/documents/processor-tech-docs/software-optimization-guides/57647.zip

- Chester Lam, “AMD’s Zen 4, Part 2: Memory Subsystem and Conclusion,” Chips and Cheese, November 8, 2022: https://chipsandcheese.com/p/amds-zen-4-part-2-memory-subsystem-and-conclusion

### N3 — Catch the AI (20 pts)

- **What the AI claimed:** The AI claimed that column-major traversal of a large array misses on nearly every access, while row-major traversal misses approximately once per 16 accesses.
- **Why you questioned it or wanted more precision:** I wanted to know whether or not this was always true or whether the cache's design, prefetching, and the arrays size could change the actual miss rate. The original explanation made it sound as if a large array automatically causes nearly every column-major access to miss, but I decided to dig much deeper into this claim. 
- **What you did to check it:** I pushed back and asked the AI to reconsider its claim and identify which parts it oversimplified. I asked for specific references to support its claims, and I then examined its references, including Cornell's Virtual Workshop material on cache associativity and AMD's Zen 4 optimization guide. Cornell's explanation showed how certain memory strides can cause cache conflicts even when the data could theoretically fit in the cache.
- **What you concluded:** The original claim that Claude made was far too broad. Column-major traversal has worse spatial locality because consecutive accesses are separated by an entire row, but the actual miss rate depends heavily on many factors such as cache capacity, associativity, array dimensions, and prefetching. Also, TLB misses can effect performance, so total array size by itself does not determine the result.

### N4 — Your best follow-up (10 pts)

**Your follow-up question:**

When the CPU requests data that isn't in L1 cache, what exactly happens next? Does it always go to L2, then L3, then RAM, or can it skip levels? Also, does the CPU have to stop and wait for the data before it can continue executing instructions?

**Why that question was useful, or how it changed/refined your understanding (1–2 sentences):**

This question helped me to understand that a cache miss does not necessarily mean that the CPU needs to stop all execution or wait for the full memory latency. The CPU can continue to execute independent instructions and overlap multiple memory requests, although instructions that depend on missing data need to wait. This helped me understand why cache misses affect performance without translating directly into an equal amount of extra runtime. I often struggle to understand these concepts in practice and how they truly function, so this was an incredibly helpful question for me to deal with this weakness. 

### N5 — Connect back (10 pts)

Check the **two** architecture concepts from lecture that most directly explain the performance difference you investigated, and explain each in one sentence.

- [x] Cache lines / block-based memory transfers — explanation:
- [x] Spatial locality — explanation:
- [ ] Temporal locality — explanation:
- [ ] Cache hierarchy (L1/L2/L3) — explanation:
- [ ] Cache-miss penalty / memory latency — explanation:

---

# Appendix — AI Conversation

Paste your full AI chat export below. No cleanup required.

**I understand that row-major and column-major traversal perform the same number of loads and additions, but column-major traversal can be much slower. I dont understand why changing the order of memory accesses makes such a big difference if the program is doing the same amount of work.
Can you explain what is happening by starting with the basic idea and helping me understand it step by step rather than just explaining everything at once?**

Let’s build this up one idea at a time. I’ll stop at the end of each step so you can check it makes sense before we add the next layer.

Step 1: Memory is slow, and the CPU knows it
Your premise is right: both traversals do the same loads and additions. What differs is how long each load takes, and a load is not a fixed-cost operation.

Fetching a value from main memory (RAM) takes on the order of 100+ CPU cycles. An addition takes about 1 cycle. So if the CPU had to go all the way to RAM for every element, it would spend nearly all its time waiting.

To fix this, CPUs have a small, very fast memory called a cache, sitting between the CPU and RAM. Reading from cache takes a few cycles instead of hundreds.

So “a load” isn’t one fixed-cost operation. It might cost ~4 cycles (cache hit) or ~200 cycles (go to RAM). The order of accesses changes how many loads hit the cache, and that is the entire difference.

Step 2: The cache doesn’t fetch single values, it fetches chunks
When you ask for one int, the hardware doesn’t bring in just those 4 bytes. It brings in a whole cache line, typically 64 bytes, which is the 4 bytes you asked for plus its neighbors.

So if you load a[0], the cache line holding a[0] through a[15] (for 4-byte ints) comes along for free. The next 15 loads of a[1]...a[15] are then already in cache.

This is a bet the hardware makes: if you touched this address, you’ll probably touch nearby addresses soon. This is called spatial locality.

Step 3: How a 2D array is laid out in memory
Memory is a flat sequence of bytes, so a 2D array has to be flattened. In C/C++ (and most languages), arrays are row-major: row 0’s elements come first, then row 1’s, and so on.

Logical:           In memory (flat):
a[0][0] a[0][1]    a[0][0] a[0][1] a[0][2] a[0][3] a[1][0] a[1][1] ...
a[0][2] a[0][3]    └──── row 0 ────────────────────┘└──── row 1 ──...
a[1][0] a[1][1]
...
Elements in the same row are adjacent in memory. Elements in the same column are separated by an entire row’s worth of bytes (the stride).

Step 4: Putting steps 2 and 3 together
Row-major traversal (for i { for j { sum += a[i][j]; } }) walks through memory in the order it’s stored: address after address, no gaps.

Access a[0][0]: cache miss, fetch 64 bytes (16 ints).
Accesses a[0][1] to a[0][15]: 15 cache hits.
Access a[0][16]: miss, fetch the next line. And so on.
That is roughly 1 miss per 16 accesses.

Column-major traversal (for j { for i { sum += a[i][j]; } }) jumps down the column, so consecutive accesses are a full row apart. If a row is 4096 ints (16 KB), each access lands on a different cache line.

Access a[0][0]: miss, fetch a line containing a[0][0..15].
Access a[1][0]: 16 KB away, so a different line. Miss.
Access a[2][0]: another line. Miss.
You use one element out of each 64-byte line you fetched. The other 15 elements are wasted, at least for now.

Step 5: “But won’t those other elements be useful when we come back?”
Good question, and this is why column-major is worse than just 16× more misses for large arrays.

After you finish column 0, you go back and start column 1. a[0][1] lives in the same cache line as a[0][0], which you fetched earlier. If that line is still in the cache, it’s a hit.

But the cache is small (tens of KB for L1, a few MB for L2/L3). To walk down one column of a big array, you touched thousands of distinct cache lines. By the time you finish the column, the early lines have been evicted to make room. When you return to a[0][1], its line is gone, and you pay for the miss again.

So for a large array, column-major misses on nearly every access, versus 1 in 16 for row-major. Same work, but roughly an order of magnitude more trips to slower memory.

Step 6: The hardware makes it worse (a bonus effect)
CPUs also have a prefetcher that notices sequential access patterns and starts fetching the next cache lines before you ask for them, hiding the latency. Row-major’s perfectly sequential pattern is ideal for it.

Large strides are harder for it to exploit well, so column-major loses this help too.

The takeaway
	Row-major	Column-major
Loads / adds	N²	N²
Cache lines used per fetch	all 16 elements	~1 element
Miss rate	~1/16	~1 (for large arrays)
Prefetcher	works great	works poorly

The program does the same arithmetic, but the cost of each load depends on whether the data is already close to the CPU. Access order determines that.

If you’d like, I can next show a small C program you can run to see the timing gap yourself, or walk through what changes when the array is small enough to fit entirely in cache (the gap mostly disappears). Which would help more?

**yes, please walk through what changes when the array is small enough to fit within the cache.**
Step 1: What “fits in cache” means

Take a small array, say 32×32 ints. That’s 1,024 ints × 4 bytes = 4 KB. A typical L1 data cache is 32 to 48 KB, so the whole array fits with room to spare.

Compare that with the earlier example: 4096×4096 ints is 64 MB, far bigger than any cache level.

Step 2: Column-major’s first pass still misses

Walk down column 0 of the small array. Each a[i][0] is in a different cache line (a row is 32 ints = 128 bytes, so consecutive rows are 2 lines apart). Every access is a miss, just like before. That’s 32 misses, and the cache is filling up with lines as it goes.

Nothing is different yet. The difference shows up on the next column.

Step 3: The fetched lines survive

Here is the key change. In the big array, the lines fetched during column 0 were evicted before you came back. Here, the entire array is only 4 KB, so all 64 lines it occupies stay in cache while you work.

Now move to column 1. a[0][1] lives in the same line as a[0][0], which you fetched during column 0. It’s still there, so it’s a hit. Same for a[1][1], a[2][1], and so on down the column.

The 15 “wasted” elements per line from the earlier discussion turn out not to be wasted. They were just waiting in cache until you needed them.

Step 4: Counting the misses

For a 32×32 array (64 cache lines total):

Row-major: each line is fetched once and fully used before moving on. 64 misses total.
Column-major: each line is fetched once, during the first column that touches it, and stays resident. Also 64 misses total.

Both traversals miss on exactly the same set of lines, since every line must be loaded from RAM at least once. The order only changes when the misses happen, not how many. Out of 1,024 accesses, about 94% are hits either way.

Step 5: What the gap looks like now

Almost all the large-array penalty disappears, because it came from eviction, and eviction no longer happens. Small differences can remain:

The prefetcher still favors sequential access, so row-major may be slightly ahead on the initial misses.
Loop overhead and address calculation differ a little (a stride multiply vs. a simple increment), which can matter once memory stops being the bottleneck.
Vectorization: compilers can often turn a row-major inner loop into SIMD instructions that add several adjacent elements at once. A strided column loop usually can’t be vectorized as well, so row-major can keep a modest edge even when everything is in cache.

But the gap shrinks from roughly an order of magnitude to something much smaller, often a small constant factor.

Step 6: The in-between regime

The effect isn’t an on/off switch. It depends on which cache level the array fits in:

Array size	Fits in	Column-major penalty
~4 KB	L1	Negligible
~256 KB	L2	Small
~8 MB	L3	Moderate (misses go to L3, not RAM)
64 MB+	Nothing	Severe (misses go to RAM)

As the array outgrows each level, the column-major re-fetches fall through to the next, slower level. That’s why you often see timing curves with distinct steps as array size grows.

The takeaway

Access order matters because of reuse distance: how much other data you touch between two uses of the same cache line. Column-major makes that distance enormous (a whole column’s worth of lines). If the cache is big enough to hold that much data, the line is still there when you return, and the order stops mattering much. If it isn’t, every return trip is a fresh miss.

If you want to see this directly, I can write a short C program that times both traversals across a range of array sizes so you can watch the gap open up as the array crosses each cache boundary. Want that next?

**Great, I think that I understand why row-major traversal benefits from spatial locality, but I want to make sure I understand the actual cost of a cache miss as I am still cloudy in that area. When the CPU requests data that isn't in L1 cache, what exactly happens next? Does it always go to L2, then L3, then RAM, or can it skip levels? Also, does the CPU have to stop and wait for the data before it can continue executing instructions? Explain the process step by step. Distinguish between what generally happens and what depends on the specific processor.**

Step 1: The lookup happens level by level

Generally true: when a load misses in L1, the request goes to L2. If L2 misses, it goes to L3. If L3 misses, it goes to the memory controller and then DRAM.

The CPU can’t skip ahead because it doesn’t know where the line is. The only way to find out whether L2 has it is to look in L2. Each level you check adds latency, roughly:

Level	Typical latency (varies a lot by chip)
L1	~4-5 cycles
L2	~12-16 cycles
L3	~40-70 cycles
DRAM	~200-400 cycles (about 70-100 ns)

So a miss that goes all the way to RAM pays the cost of all the failed lookups plus the DRAM access, though DRAM dominates.

Step 2: What the hardware does when L1 misses

Generally true, though the details differ by design:

The load finds no matching line in L1.
The hardware records the miss in a small table of outstanding misses (called MSHRs or line fill buffers) so it can track the request and avoid sending a duplicate if another load wants the same line.
The request is sent to L2, and on a miss there, onward.
When the line arrives, it’s installed in L1 (evicting something else if needed), and the waiting load receives its value. Many designs forward the requested word right away rather than waiting for the whole 64-byte line.
Step 3: Can levels be skipped?

For the lookup, mostly no. But several things modify this simple picture, and these depend on the processor:

Fill path: many modern L3 caches are “victim” or non-inclusive caches. A line fetched from DRAM may be placed directly into L1/L2 without being copied into L3 at all. The lookup still checked L3, but the data skipped it on the way in.
Speculative memory access: some designs start the DRAM request in parallel with the L3 lookup, cancelling it if L3 hits. This hides some latency.
Other cores: the line might live in another core’s L1 or L2. Coherence hardware (snoop filters or directories) can supply the data from there instead of from RAM.
Prefetchers: these fetch lines into L2 or L3 before you ask, so when the load happens, it hits at a closer level than it otherwise would.
Cache-bypassing accesses: non-temporal loads and stores, and uncacheable memory like device registers, deliberately avoid some or all levels.
Step 4: Does the CPU stop and wait?

Generally true for modern high-performance cores: not immediately. They are out-of-order, so the sequence looks like this:

The load misses and is parked, waiting for data.
Instructions that don’t depend on the loaded value keep executing.
Instructions that do depend on it wait in a queue.
Instructions still retire (commit) in program order, so the missed load blocks retirement of everything after it.
Step 5: Where the stall actually comes from

The core can only run ahead as far as its reorder buffer (ROB) allows, which holds a few hundred instructions on modern cores (this size varies by chip). A DRAM miss takes ~300 cycles, which is enough time to fill the ROB. Once it’s full, the core can’t take in new instructions and stalls until the load completes.

So the CPU hides a portion of the latency with useful work, but a full DRAM miss is almost always too long to hide completely.

Step 6: Memory-level parallelism (this connects back to your question)

Misses don’t have to be handled one at a time. A core can have several outstanding misses at once (roughly 10-20 line fill buffers per core, depending on the chip).

In sum += a[i][j], the addresses don’t depend on loaded values, so the CPU can issue many loads ahead of time and overlap their misses. The only dependency chain is the additions, which are cheap.

That means the effective cost per miss is closer to latency ÷ number of overlapping misses than the full latency. Column-major still loses badly because it generates far more misses than row-major, and the hardware can only overlap so many.

Compare pointer chasing, like walking a linked list, where each load’s address comes from the previous load. There’s no overlap, so every miss costs its full latency. This is why linked lists can be dramatically slower than arrays.

Summary
	Generally true	Depends on the processor
Lookup order	L1 → L2 → L3 → RAM	Whether L3 is inclusive/victim, where lines get filled
Latencies	Each level is several times slower	Exact cycle counts
Waiting	Independent work continues, dependent work waits	ROB size, how much latency is hidden
Overlap	Multiple misses in flight	Number of fill buffers, prefetcher quality
Line size	Usually 64 bytes	Some designs (e.g., Apple’s M-series) use 128

If it would help, we can next work through a concrete example: estimating how many cycles the 4096×4096 column-major loop spends per element, using these numbers and the overlap idea from Step 6.

**Ok, so I understand what happens during a cache miss. Now I want to investigate real latency numbers for a specific modern processor for my homework.Can you choose a specific processor or processor family and help me find credible sources for approximate L1, L2, L3, and main memory access latencies?For each level, can you tell me: The approximate access time, whether the number is measured, specified by the manufacturer, or estimated from another technical source, and the original source where the number comes from. Also, distinguish between cache-hit latency and the total time to access main memory after cache misses. Explain why these numbers will vary by processor and workload, and give me links to the sources so that I can verify them myself. Do NOT invent these numbers if reliable sources aren't available.**

I’ll pick AMD Zen 4 (Ryzen 7000 series), since it has both a vendor optimization guide and detailed independent measurements, and look up the sources now.

I used AMD Zen 4 (Ryzen 7000 desktop, e.g. the Ryzen 9 7950X). It has a manufacturer document that specifies latencies in cycles, and an independent measurement on a real 7950X in nanoseconds. AMD publishes no DRAM latency figure, so that one comes only from the independent measurement.

Source A: AMD’s own numbers (specified, in cycles)

The source is Software Optimization Guide for the AMD Zen4 Microarchitecture, AMD publication 57647, Rev. 1.01, April 2023.

Level	Figure	Where in the guide
L1D	4-cycle integer load-to-use latency (7 for FP), or 5 (8 for FP) with a complex addressing mode	§2.6.2 and §2.12
L2	variable load-to-use latency of no less than 14 cycles	§2.6.3
L3	average load-to-use latency of 50 cycles	§2.6.4
DRAM	Not given	none 
Advanced Micro Devices Software Optimization Guide for the AMD Zen4
 +2

Appendix A of the guide repeats these numbers as add-ons to instruction latency and marks the L2 and L3 values as possibly varying by product. The guide doesn’t say how AMD obtained the numbers, so I can’t tell you whether they come from silicon measurement or design simulation. “No less than” and “average” also mean they are not guarantees. 
numberworld

Source B: independent measurement (in nanoseconds)

The source is Chips and Cheese, AMD’s Zen 4, Part 2: Memory Subsystem and Conclusion, Chester Lam, Nov 2022. The testbed was a Ryzen 9 7950X with DDR5-6000.

Level	Measured	Notes
L1D	4 cycles, about 0.7 ns at a 5.7 GHz clock	2 MB pages, so TLB effects are excluded
L2	14 cycles, 2.44 ns	same test
L3	in the 8-9 ns range with 2 MB pages; 9.47 ns with 4 KB pages (below L2 TLB capacity)	the article’s text gives no L3 cycle count, only the chart does
DRAM	73.35 ns at a 1 GB test size	the authors say it depends heavily on the DDR setup 
amds zen 4 part 2 memory subsystem and conclusion
 +3

I didn’t find a full description of the test method in the article text, so check the Chips and Cheese site before describing their method in your homework.

Consistency check (my own arithmetic)

At 5.7 GHz, AMD’s cycle counts convert to 4 cycles ≈ 0.70 ns, 14 ≈ 2.46 ns, and 50 ≈ 8.8 ns. That matches Source B’s measurements, so the two sources agree. The DRAM figure converts to roughly 400 cycles (73.35 ns × 5.7 GHz ≈ 418), but the real clock during the test may differ, so treat that as order-of-magnitude only.

Cache-hit latency vs. main-memory latency
L1, L2, L3 numbers are hit latencies. Each is the time for a load whose data is found at that level.
The DRAM number is a total. It is the end-to-end time for a load that misses L1, L2 and L3, so it includes those failed lookups. It also includes the trip through the fabric and memory controller, the DRAM access itself, and sometimes TLB misses. At a 1 GB test size, essentially every access misses all caches, which is why that size is used for the DRAM figure.
Why the numbers vary

By processor:

Clock speed decides how many nanoseconds a cycle is, so cycle counts don’t compare directly across chips.
Cache design matters too. Chips and Cheese reports that doubling Zen 4’s L2 cost 2 cycles of latency, and that Intel’s Golden Cove takes almost 20 more cycles to reach its L3 than Zen 3 and Zen 4. 
chipsandcheese
chipsandcheese
DRAM latency depends on memory type, speed, timings and platform (the article itself warns about this).

By workload:

Page size and TLB reach change results. At a 12 MB test size with 4 KB pages, Zen 3’s L3 latency rises above 12 ns while Zen 4 stays just below 10 ns. 
chipsandcheese
Heavy memory traffic changes latency. PC Games Hardware’s summary of a Chips and Cheese test says Zen 4 latency can spike when high bandwidth demand hits the other chiplet. I only saw the German summary, not the original article. 
pcgameshardware
Access pattern also matters. Dependent loads (pointer chasing) expose the full latency, while independent loads overlap, as we discussed earlier.
What I could not verify
DRAM latency from AMD: I found none.
AMD’s official hosting of the guide: I read it from a third-party mirror (numberworld.org). A GitHub index of AMD guides notes that AMD’s own portal link for 57647 wasn’t working when checked, so search AMD’s documentation hub for “57647” to confirm against the original. 
github
Secondary sources: WikiChip, Guru3D and Wikipedia repeat the 14 and 50 cycle figures, but they are not originals, so don’t cite them as such.
Newer chips: Zen 5 exists, and Wikipedia says its L3 latency is 3.5 cycles lower. I haven’t gathered primary Zen 5 numbers. 
Wikipedia
Sources
Software Optimization Guide for the AMD Zen4 Microarchitecture (57647 Rev 1.01), mirror copy
Chips and Cheese: AMD’s Zen 4, Part 2: Memory Subsystem and Conclusion
PC Games Hardware summary of Chips and Cheese Zen 1–5 latency analysis (secondary, German)
GitHub index of AMD optimization guides (for locating the official 57647)

For your write-up, cite AMD’s guide as the specified source and Chips and Cheese as the measured one, and state the testbed (7950X, DDR5-6000, page size). If you’d like, I can turn this into a table formatted for your homework, or repeat the search for an Intel chip so you can compare the two.

**You said "For a large array, column-major misses on nearly every access.” I want to challenge this and get some more information. Is this always true, or does it depend on the cache architecture and other factors? Could the cache’s design, prefetching, or array size change how often cache misses happen? Please explain which parts of your original claim are reliable and which are simplifications. Then, give me a credible  source that I can use to verify your explanation by myself. Do not just defend your original answer.**

You’re right to push on this. Before I answer, I’m going to check a couple of the specifics against sources, since some of my earlier claims were simplifications.

Finding a workable source after a blocked reference.

No, it isn’t always true. As I stated it, the claim was an oversimplification, and a few of my supporting statements were wrong or unverified. Here is which parts hold up.

What is reliable
Column-major uses one element per cache line in each pass. Within a single column pass, the other 15 elements of each line go unused.
Reuse is decided by reuse distance. The line holding a[0][0] is reused when you reach a[0][1], one column later. Whether it is still cached depends on what you touched in between.
At L1, “misses nearly every access” is reliable for a long column. Zen 4’s L1D is 32 KB, which is 512 lines (AMD guide §2.6.2), so any column longer than that cannot stay in L1.
What I got wrong or oversimplified
What I said	Verdict
“For a large array, misses nearly every access”	Too strong. It holds at L1 and sometimes at deeper levels, depending on column length, associativity and page size.
My table keying the penalty to total array size (64 MB+ means “misses go to RAM”)	Wrong framing. The relevant quantity is the column’s working set, not the array’s size.
“Large strides are harder for the prefetcher, so column-major loses this help”	Overstated. It depends on the stride and the prefetcher’s limits.
(omitted)	I never mentioned conflict misses, TLB misses, or that overlapped misses cost less than a full latency each.
1. The working set is the number of rows, not the array size

One column pass touches N distinct lines. For 4096 rows that is 4096 × 64 B = 256 KB (my arithmetic). That is far more than Zen 4’s 32 KB L1D but well under its 1 MB L2. Capacity alone would let those lines survive in L2. Each L1 miss would then cost an L2 hit (about 14 cycles) rather than a DRAM access (about 73 ns).

The Algorithmica matrix-multiplication case study makes this point. After the first column read, the next 15 reads hit the same cache lines, unless the matrix is so large that n × cache_line_size bytes won’t fit in the cache, which it says isn’t the case for practical sizes. That is the opposite of my “nearly every access” claim, and it is the better default. 
algorithmica

2. Associativity can make my claim true for a different reason

Caches are set-associative, and a stride that is a multiple of a large power of two maps every access to the same few sets. Cornell’s Virtual Workshop works this out for Skylake’s 32 KB L1d. A stride that is a multiple of 4096 bytes shrinks the effective L1d from 32 KB to just 8 lines (512 bytes), which it calls cache thrashing. 
cornell

Zen 4’s L1D has the same geometry: 32 KB and 8-way (AMD guide §2.6.2), indexed by address bits [11:6] (§2.6.2.3). By my application of that geometry, my 4096-int example has a 16 KB row stride, so it hits this case. At L1 it would miss nearly every access even with plenty of capacity. A non-power-of-two row length, such as 4100 ints, would largely avoid it. The Algorithmica page says the same: performance deteriorates only on a few specific matrix sizes, those where n is a multiple of a large power of two. For L2 and L3, whether conflicts occur depends on how addresses map into sets, and I found no source on Zen 4’s L2/L3 indexing. I’m not claiming either way. 
algorithmica

The peer-reviewed literature backs this up. A PLDI 2016 paper notes that even with high associativity, enough accessed elements can map to one set to cause conflict misses, even when the working set is much smaller than cache capacity. Its fix is array padding. 
sigplan

3. TLB misses are a separate cost I left out

With 4 KB pages, a 16 KB stride puts every row in a different page. A 4096-row column touches 4096 pages, but Zen 4’s L2 data TLB holds 3072 entries and the L1 DTLB only 72 (AMD guide §2.7). By my arithmetic, that column overflows the TLB, so you’d expect page walks on top of any data-cache misses. The Chips and Cheese article from earlier showed this effect: with 4 KB pages, Zen 3’s L3 latency rose above 12 ns at a 12 MB test size, while Zen 4 stayed just under 10 ns because of its larger TLB.

4. Prefetching: my claim was unverified

The AMD guide lists an L1 stride prefetcher. It uses the access history of individual instructions to fetch additional lines when each access is a constant distance from the previous. A fixed-stride column walk is exactly that pattern, so my claim that it “works poorly” was too confident. 
numberworld

The limits are design-specific. For Intel, a forum post quoting the Intel Optimization Manual says the IP-based stride prefetcher detects strides of up to 2K bytes, and prefetched data must be within the same 4K page. A patent description likewise says conventional hardware prefetchers stop issuing prefetches at a 4 KB page boundary. A 16 KB stride would defeat prefetchers with those limits, but a 256-byte stride might not. I can’t tell you Zen 4’s limits, because the guide sections I read don’t state them. Prefetching can turn some demand misses into hits, so it changes the miss count, not only the cost. 
intel
google

5. Miss count is not miss cost

In sum += a[i][j] the load addresses don’t depend on earlier loaded values, so misses can overlap. Zen 4’s load-store unit tracks up to 24 outstanding cache misses (AMD guide §2.12). Even when nearly every access misses, the time per miss can be well below a full latency.

A more defensible version of the claim

Walking down a column of a row-major array touches one line per row. Whether those lines are still cached at the next column depends on (a) whether N lines fit in each cache level, (b) whether power-of-two strides cause set conflicts, and (c) whether the pages fit in the TLB. Prefetching can convert some demand misses to hits, and out-of-order overlap reduces the cost of those that remain.

Sources, with their limits
Algorithmica: Matrix Multiplication is the clearest source for the “first column misses, next 15 hit” point. It is a well-known online book but not peer-reviewed.
Cornell Virtual Workshop: Cache Associativity gives the 4096-byte stride calculation for Skylake.
Hong et al., “Effective Padding of Multi-Dimensional Arrays to Avoid Cache Conflict Misses,” PLDI 2016 (doi:10.1145/2908080.2908123) is the peer-reviewed source.
AMD Zen 4 Software Optimization Guide, 57647 is the mirror copy used before; it is the source for the Zen 4 parameters above.
Intel community thread quoting the Optimization Manual is a secondhand quote about older Intel cores, so go to Intel’s manual for anything you cite. Google Patents US9430392 supports the page-boundary point.
Drepper, “What Every Programmer Should Know About Memory” (PDF) is widely recommended and covers matrix access patterns, but the site blocked my automated access. I haven’t read it, so I can’t vouch for specific sections.

You can also test this yourself: time a column-major sum with N = 4096 versus N = 4100 and compare L1 miss counts with perf stat -e L1-dcache-load-misses. Associativity thrashing predicts a large gap even though the arrays are nearly the same size. I can write that program for you if you’d like.

**Ok great, can you summarize what you learned from correcting your original claim in 2 to 3 sentences? Explain what you originally got wrong or simplified too much, what factors actually affect the cache miss rate, and what a more accurate claim would be. Keep it simple enough for a homework assignment.**

I originally claimed that column-major traversal of a large array misses on nearly every access, but I treated total array size as the deciding factor when what matters is whether the lines in one column (one per row) stay in cache until the next column reuses them. That depends on how many rows there are compared with each cache level’s capacity, on associativity (power-of-two row lengths map accesses to the same few cache sets and cause conflict misses), and on TLB reach, while prefetching can turn some misses into hits and out-of-order execution overlaps the misses that remain. A more accurate claim is that column-major traversal wastes spatial locality within each pass and can miss on nearly every access at L1, but how often it misses at deeper levels, and how much those misses cost, depends on column length, cache associativity, page size, and prefetching.

**Ok, so how do programmers use this information in real world coding? Explain a specific technique for improving cache locality, and how it improves performance. Also, can a compiler automatically change column-major traversal into row-major traversal, or are there situations where it cant  do that?**

Before explaining what compilers can and can’t do, I’ll test it on a real compiler instead of relying on documentation alone.

Quick status: I’m testing real GCC and clang builds on this machine to see when loop interchange happens, and I’m running a cache-blocking experiment next. Results are coming shortly.

Programmers use this by choosing loop order and data layout so the innermost loop walks memory contiguously. When that isn’t possible, as in a matrix transpose, they use cache blocking (tiling). Compilers can sometimes reorder loops for you, but only under conditions they can prove safe, and I tested where that breaks down.

How this shows up in real code
Put the last index innermost in C (a[i][j] with j innermost). Fortran is the reverse.
Avoid power-of-two row lengths or pad them (see the associativity discussion above).
Block loops that reuse data (below).
Measure instead of guessing, for example with perf stat miss counters.
Use tuned libraries such as BLAS for matrix kernels instead of hand-writing them.
The technique: cache blocking (tiling)

Transposing a matrix reads rows from one array and writes columns of the other, so one array is always walked with a large stride. Reordering the loops can’t fix that. Tiling can. It processes the matrix in small B×B tiles that fit in cache, so the lines touched in the tile stay cached while you finish with them.

c
for (int ii = 0; ii < N; ii += B)
    for (int jj = 0; jj < N; jj += B)
        for (int i = ii; i < min(ii+B, N); i++)
            for (int j = jj; j < min(jj+B, N); j++)
                dst[j*N + i] = src[i*N + j];

This works because it shrinks the reuse distance. Instead of N lines between reuses, as in a full column pass, there are only about B. Cornell’s materials describe blocking as a way to cut the miss rate by improving spatial and temporal locality. They also note that the best block size depends on the cache capacity and other hardware details.

I measured it on this sandbox (4096×4096 ints, best of 3, GCC 13 -O2):

N	Naive	Best blocked	Speedup
4096	198.7 ms	69.1 ms (B=32)	about 2.9×
4100	105.7 ms	34.0 ms (B=32)	about 3.1×

Two details stand out:

Very large tiles got slower again (B=128 took 111 ms at N=4096), because the tile stopped fitting in cache.
The naive version was about 1.9× slower at N=4096 than at N=4100, consistent with the power-of-two conflict effect we discussed.
Can the compiler turn column-order into row-order?

Sometimes. The transformation is called loop interchange. GCC’s interchange pass is enabled by default at -O3. LLVM has an interchange pass that aims for cache-friendlier access and tests legality using dependence analysis. I tested GCC 13.3 and clang 18.1.3 on a 4096×4096 array (single runs; the gaps are large, but treat the numbers as indicative):

Case	GCC -O2	GCC -O3	clang -O3	clang -O3 + interchange enabled
Int sum, column order	143.6 ms	8.7 ms	214.1 ms	8.3 ms
Same sum, row order (baseline)	6.4	8.9	8.9	8.4
Float sum, column order	178.7	170.0	171.4	151.9
a += b via pointers, column order	326.0	350.6	371.4	9.3
Loop-carried dependence	205.2	185.5	220.8	204.0

What I found:

It happens only with optimization on. GCC needed -O3. At -O2 nothing changed.
Clang 18 did nothing by default. I had to enable its interchange pass with -mllvm -enable-loopinterchange. A recent LLVM change (Aug 2026) proposes turning it on by default at -O3, but I don’t know whether it has landed.
Floating-point sums were not interchanged. Neither compiler interchanged the float sum, and with -ffast-math both did (GCC dropped from 170 ms to 7.1 ms). Interchange reorders the additions, and float addition isn’t associative, so the compiler needs permission to change the result.
Possible aliasing blocked GCC. With two pointer arguments that might overlap, GCC left the loop alone (350.6 ms). Adding restrict, a promise they don’t overlap, made GCC interchange it: 332 ms became 9.9 ms in a separate test. Clang interchanged the unannotated version when its pass was enabled. I didn’t investigate why the two compilers differ there.
Real dependences block it. For a[i][j] = a[i-1][j+1] + 1, clang reported that it cannot interchange due to dependences, and no build interchanged it. By my analysis, swapping the loops would change which values get read before they’re overwritten.

Other situations I didn’t test, which the pass descriptions suggest limit it, include loops that aren’t cleanly nested (work between the loops) and complicated array indexing. Clang’s own remarks flagged restrictions on the loop shapes it handles. Interchange also can’t help a transpose at all, since both loop orders walk one array by column. That’s why tiling matters.

Caveats
This is a virtualized Intel Xeon sandbox with 2 vCPUs (48 KB 12-way L1d, 2 MB L2), so timings are noisy and won’t match your machine.
Compiler behavior is version-specific. Rerun with your own toolchain, using gcc -fopt-info-loop-optimized or clang -Rpass=loop-interchange -Rpass-missed=loop-interchange.
Sources
GCC patch introducing the loop interchange pass, enabled at -O3
GCC docs change noting -floop-interchange is enabled at -O3
LLVM LoopInterchange.cpp source
LLVM change to default loop interchange at -O3 (Aug 2026)
Cornell Virtual Workshop: Array Blocking
Cornell CS 3410: Cache Optimization assignment

I can send you the test programs if you’d like to run them yourself.

---

# Feedback on the Assignment (5 free points)

Your opportunity to tell us about this assignment. There are no right or wrong answers, every item is optional, and leaving any item blank is fine. All items are graded on completion only, never on the response given. For the numeric items below, entering 6 or leaving the answer blank both count as "Prefer not to answer."

### B1 — Before

Before this assignment, how well could you explain why two programs that perform the same number of operations can have very different runtimes because of their memory access patterns?

1. I could not explain this topic at all; it was new to me.
2. I could name some related terms, but not explain the concepts behind them.
3. I could explain some of the assignment's key concepts at a high level, but not how they fit together.
4. I had some previous knowledge of this topic and would have felt comfortable explaining most of the key concepts.
5. I had a working background in this topic and was confident I could explain the performance effect to someone else.
6. (or blank) = Prefer not to answer.

**Your answer (1–6):** _3_

### P1 — After

After this assignment, how well could you explain why two programs that perform the same number of operations can have very different runtimes because of their memory access patterns?

1. I still don't feel I could explain this comfortably yet; I need more practice and exposure to the topic.
2. I could explain some of the assignment's key concepts at a high level, but not in detail.
3. I could explain the individual concepts but not fully how they connect to the observed performance difference.
4. I am confident I could explain the effect well enough to help a fellow classmate understand it.
5. I could confidently explain and defend the relationship between access pattern, locality, and performance in a technical discussion.
6. (or blank) = Prefer not to answer.

**Your answer (1–6):** _4_

### INV — What drove your investigation?

Pick the one that fits best.

- [ ] I mainly wanted to complete the required assignment efficiently.
- [ ] I mainly wanted to understand enough to answer the assignment questions correctly.
- [x] I mainly wanted to resolve why the two traversals behaved differently even though they appeared to perform the same work.
- [ ] I mainly wanted to understand memory locality well enough that I could recognize or apply it in other programs.
- [ ] Something else. *(Optional: tell us what.)*
- [ ] Prefer not to answer

### FAV — What helped most?

Pick one.

- [ ] The in-class demonstration / prediction
- [ ] Reading the Concept section
- [ ] The AI investigation
- [x] Challenging or verifying something the AI told me
- [ ] Answering the Confirm questions
- [ ] Prefer not to answer

### SCA — What should we do with this format?

- [x] Continue as is. *Tell us more (optional): any suggestions to make it even better?*
- [ ] Adjust something. *Tell us more (optional): what would you change?*
- [ ] Stop, and go back to a traditional homework model. *Tell us more (optional): what didn't work for you?*
- [ ] Prefer not to answer

---

# Grading Rubric

| # | What's being evaluated | Points |
|---|---|---:|
| MC1–MC4 | Correctly identifies the core memory-hierarchy and locality concepts | 20 |
| N1 | States the corrected model: explains why memory access pattern and spatial locality can make operation count alone a poor predictor of runtime | 20 |
| N2 | Provides a specific memory-hierarchy latency comparison for a named processor/family with a credible source | 15 |
| N3 | Identifies a specific AI claim, gives a genuine reason for checking it, describes a verification action, and reaches a supported conclusion | 20 |
| N4 | Identifies a specific follow-up question and explains how it advanced or refined understanding | 10 |
| N5 | Selects two defensible architecture concepts and correctly explains their connection to the observed performance difference | 10 |
| Transcript | Included and shows the AI investigation | Required |
| Feedback | Completes the required feedback items; "Prefer not to answer" (or leaving an item blank) counts as complete | 5 |

**Total: 100 points.**
