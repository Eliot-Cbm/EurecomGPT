# Phase 2 — Reflection: where does parallelism stop helping?

Copy this file to `submission/phase2_reflection.md`, fill in every answer slot, and commit it:

```bash
cp phase-2-tiny-gpt/reflection_template.md submission/phase2_reflection.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or your threading plot freely.

Numbers in section 1 must **match `submission/phase2_report.json`** — copy them across, do not
retype from memory; they are checked against it.

---

## 1. Your measurements

| | Value |
|---|---|
| Naive attention, ms (`attention.naive_ms`) | <!--answer:attn_naive_ms-->265.237<!--/answer--> |
| Vectorized attention, ms (`attention.vectorized_ms`) | <!--answer:attn_vectorized_ms-->1.232<!--/answer--> |
| CPU training, ms for 100 steps (`devices.cpu_ms`) | <!--answer:cpu_ms-->524100.1<!--/answer--> |
| GPU training, ms for 100 steps (`devices.gpu_ms`) | <!--answer:gpu_ms-->12949.9<!--/answer--> |
| Final training loss (`training.final_loss`) | <!--answer:final_loss-->1.4578<!--/answer--> |

**The thread sweep.** Give your four timings from `threads` — 1, 2, 4 and 8 threads — and say
which count was fastest. (verbatim: the numbers, in order)

<!--answer:thread_sweep-->
1 thread: 2618.3 ms; 2 threads: 2254.063 ms; 4 threads: 2413.618 ms; 8 threads: 2406.892 ms. The fastest configuration was 2 threads.
<!--/answer-->

---

## 2. Intra-node parallelism, three ways

**Why vectorizing wins.** Both attention implementations do the same arithmetic and produce the
same result (the unit test checks that). Explain where the naive version's time actually goes,
and what the vectorized one does differently — in terms of the machine, not the Python.
(~70 words)

<!--answer:vectorization_why-->
The naive implementation performs many small operations, so the machine repeatedly starts work on small pieces of data instead of processing large arrays efficiently. This limits the use of SIMD instructions and efficient memory access. The vectorized implementation processes whole arrays using optimized numerical routines, allowing the CPU to perform many operations in parallel and reducing overhead. Both versions produce the same result, but vectorization makes much better use of the hardware.
<!--/answer-->

**Where the thread curve flattens, and why.** Using your sweep: more threads stopped helping —
or started hurting — at some point. Say where, and give the reason. Your Colab runtime's vCPU
count is part of the answer; so is what the threads have to coordinate on. (~80 words)

<!--answer:threading_plateau-->
The fastest configuration was 2 threads at 2254.063 ms. Performance stopped improving beyond 2 threads: 4 threads took 2413.618 ms and 8 threads took 2406.892 ms. My Colab runtime reports only 2 CPU cores, so additional threads cannot provide more physical CPU capacity. They also need to coordinate and share resources such as memory bandwidth and cache. This coordination creates overhead, which can outweigh the benefit of adding threads.
<!--/answer-->

**What made the GPU faster.** The model, the data and the step count were identical; only the
device changed. Say what about a transformer training step suits a GPU, and name one workload
that would *not* see this speed-up. (~70 words)

<!--answer:gpu_why-->
Transformer training involves large matrix multiplications and tensor operations that can be divided into many parallel calculations. GPUs contain many processing units designed to execute this kind of work efficiently. In my measurements, 100 training steps took 524100.1 ms on the CPU and 12949.9 ms on the GPU. A small, sequential workload with little parallel work, such as a simple calculation with a few values, would not benefit nearly as much from a GPU.
<!--/answer-->

---

## 3. The question

**Where does parallelism stop helping, and why?** Pull the three experiments together. Your
answer should distinguish the *kinds* of limit you met — work that cannot be divided, hardware
you do not have, coordination that costs more than it saves, data that has to move — and say
which one bound each experiment. One concrete number from section 1 per claim. (~150 words)

<!--answer:parallelism_limit-->
The attention experiment shows that execution efficiency matters: vectorization reduced the time from 265.237 ms to 1.232 ms by using hardware more efficiently. The thread experiment shows a hardware and coordination limit: my runtime has 2 CPU cores, and 2 threads were fastest at 2254.063 ms; 4 threads took longer at 2413.618 ms. Adding threads cannot help when the hardware is limited and coordination costs more than it saves. The GPU experiment shows that highly parallel workloads benefit from specialized hardware: training took 524100.1 ms on the CPU versus 12949.9 ms on the GPU. However, data movement and work that cannot be parallelized still limit speed-ups. Parallelism stops helping when there is not enough independent work, hardware capacity, or when coordination and data movement outweigh the saved computation.
<!--/answer-->

**What you would measure next.** One experiment this phase did not run that would sharpen the
answer, and what result would change your mind. (~50 words)

<!--answer:next_experiment-->
I would measure GPU training time with several batch sizes while keeping the model and number of training steps fixed. I would also measure data transfer time separately. If larger batches improved throughput significantly, it would show that the GPU was not fully utilized at the original batch size. If transfer time became dominant, it would show that moving data limits performance.
<!--/answer-->
