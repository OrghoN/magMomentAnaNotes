# How the training's memory needs were reduced

*A conceptual explanation. For file names, commits and test results, see
`LOADER_MEMORY_FIX_LOG.md`.*

## The problem in one paragraph

The training data is about 19 million events (3.3 GB compressed, 12.7 GB as text). The model
is tiny, and the GPU had memory to spare. The problem was the ordinary computer memory (RAM)
used to hold the data while feeding it to the GPU. The original code needed about **30 GB just
to load the file** and about **65–70 GB by the end of the first pass over the data**. The
EAF session allows 20 GB. The new code is expected to need **about 6–8 GB** for the whole run.

The data itself was never the issue. The useful numbers in this dataset fit in about 4.5 GB.
Almost all of the memory went into extra copies of the data, unused data, and convenient but
wasteful ways of storing it. Each fix below removes one of these.

---

## Where the memory was going

Think of loading the data as unpacking a delivery into a workshop. The old code did these
things:

1. **It unpacked everything onto the floor at once.** It read the whole compressed file into
   memory (3.3 GB), then decompressed all of it into a second block of memory (12.7 GB), and
   only then started to make sense of it. For a moment, both the box and its full contents
   took up space, about 16 GB before any real work.

2. **It kept every item, including the ones never used.** The file has 106 columns per event.
   Training uses 54. The other 52 (diagnostic flags, truth information, empty placeholder
   columns) were loaded and kept for the whole run anyway.

3. **It stored the per-particle data inefficiently.** Each event has a list of reconstructed
   particles ("prongs"), with 27 numbers each. The old format kept a separate list for each
   of the 27 quantities in every event. Each list carries its own bookkeeping, and most events
   have only one or two prongs, so the bookkeeping took more space than the numbers
   themselves.

4. **It made a full copy to shuffle.** To split the data randomly into training, validation and
   test sets, it built a complete shuffled copy of the whole table. For a while there were two
   full copies (about 26 GB).

5. **It kept a "ready-made" copy of every event, in every helper process.** During training,
   several helper processes prepare batches for the GPU. Each event, once prepared, was stored
   in a cache so it wouldn't have to be prepared again. Storing all 15 million prepared training
   events costs about 48 GB. Worse, the cache sat inside the helper processes, which are
   restarted at every pass over the data. So the memory was spent every pass, and the cache
   never saved any work.

---

## What was changed

### 1. Unpack a little at a time (streaming)
The file is now decompressed and read in **small pieces of 64 MB**. Each piece is turned into
numbers, and then the piece of text is thrown away before the next one is read. The full text
of the file never sits in memory. The cost of reading is now about 0.3 GB, whether the file is
100 MB or 13 GB.

### 2. Keep only what training uses
While reading, only the **54 columns the configuration asks for** are kept. These are the
inputs, the targets, and the columns needed to evaluate the results afterwards. Everything
else is skipped as it goes by.

### 3. Store the numbers compactly
All prongs of all events now go into **one big table**: one row per prong, 27 columns. A
second, small list records where each event's prongs start and end, as a pair of numbers per
event. The per-event information (11 inputs, 2 targets, evaluation columns) goes into similar
plain tables. Everything is stored in 32-bit floating point, the precision the GPU uses
anyway. Laid out like this, the whole dataset takes **about 4.5 GB**, and there is very little
bookkeeping overhead.

### 4. Shuffle the index, not the data
To make the random train/validation/test split, the code now shuffles a **list of event
numbers** and gives each set its own list of numbers. The data itself is never copied or
moved. Shuffling 19 million numbers costs about 150 MB, compared with about 13 GB for copying
the table.

### 5. Normalise in place
The inputs are rescaled (mean 0, standard deviation 1) using statistics from the training
set. This used to create new copies of the columns. Now the numbers are rescaled where they
already are, piece by piece.

### 6. No cache: build each event when it's needed
The prepared-event cache was removed. With the compact layout, preparing an event is just
cutting out a few slices of the tables and padding them to a fixed size. That takes
microseconds, so storing the results is pointless. This removed the largest cost of all
(about 48 GB), and it also makes data loading faster than before.

### 7. Let helper processes share the data
The helper processes are created by "forking" the main process. On Linux, a forked process
shares its parent's memory until either side changes it. The helpers only read the data, so
all of them share the same single 4.5 GB copy, rather than each getting its own. The code now
explicitly asks for this way of starting the helpers. It also caps their number at the number
of CPUs actually available.

---

## The result, side by side

| | Old | New |
|---|---|---|
| Text of the file in memory | 16 GB (compressed + decompressed) | ~0.3 GB (one piece at a time) |
| Columns kept | 106 | 54 |
| Data table | ~13 GB → ~16 GB after conversion | ~4.5 GB |
| Shuffling | a second full copy (~13 GB) | a list of event numbers (~0.15 GB) |
| Cache of prepared events | ~48 GB, rebuilt every pass | none |
| **Peak while loading** | **~30 GB** | **~6–7 GB** (expected) |
| **During training** | **~65–70 GB** | **~6–8 GB** (expected) |

The 4.5 GB data size and the 0.3 GB reading cost were measured on a sample of real files and
scaled up. The totals include an estimate for PyTorch, the GPU driver and the helper
processes, which could not be measured on the GPVM. The real figure will come from the first
full run on the EAF, which now records its peak memory at the end of `nohup.out`.

## What was given up

Very little:
- The random split is different from the one the old code would have produced. Results are
  not row-for-row comparable with older runs, but there were none to compare with.
- The rescaling is done in 32-bit rather than 64-bit arithmetic. The differences are around one
  part in a million, far below the 20% noise deliberately added to the inputs during training.
- The first load still has to decompress and read the whole file, which takes a few minutes.
  An optional on-disk cache (`"array_cache"` in the config) saves the compact tables after
  the first read, so later runs can skip this step.

