# Phase 3 — Comparison Writeup

Copy this file to `submission/phase3_comparison.md`, fill in every answer slot, and commit it:

```bash
cp phase-3-mapreduce-spark/comparison_template.md submission/phase3_comparison.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or images freely.

Numbers in section 1 must **match `submission/phase3_report.json`** — copy them across, do not
retype from memory; they are checked against it.

---

## 1. Facts from your runs

Numbers only (e.g. `12`, `20.3`, `0.041`).

| | Value |
|---|---|
| Map tasks in the cloud job (`mapreduce.num_splits`) | <!--answer:num_map_tasks-->12<!--/answer--> |
| Reduce tasks in the cloud job (`mapreduce.num_reducers`) | <!--answer:num_reduce_tasks-->4<!--/answer--> |
| Cloud MapReduce wall time, seconds (`mapreduce.cloud_elapsed_s`) | <!--answer:cloud_elapsed_s-->42.1<!--/answer--> |
| Local single-process wall time, seconds (`mapreduce.local_elapsed_s`) | <!--answer:local_elapsed_s-->0.01<!--/answer--> |
| Spark wall time for the `collect`, seconds (`tfidf.spark_elapsed_s`) | <!--answer:spark_elapsed_s-->4.4<!--/answer--> |
| Number of **stages** in the Spark job (read off notebook 2b / 3a) | <!--answer:num_stages-->4<!--/answer--> |
| Task **retries** you counted in your `--chaos` run (0 if none happened) | <!--answer:chaos_retries-->1<!--/answer--> |

**Explain the timings.** The three wall times above cover the same 60 documents, but not the
same job: the single-process run and the cloud MapReduce are *the same word count*; Spark's
time is for *TF-IDF*, which is more work (two `reduceByKey`s and a `join`). Rank the three and
explain what each is spending its time on. Say which comparison is like-for-like, and what the
Spark number — a heavier job, on one machine — tells you about where the cloud job's time goes.
(~80 words)

<!--answer:timing_explanation-->
The local WordCount was the fastest at 0.01 s because it ran in a single process on a small corpus of 60 documents. The cloud MapReduce job took 42.1 s due to distributed task orchestration, remote function calls and Cloud Storage I/O. Spark took 4.4 s to compute TF-IDF, a heavier workload involving multiple shuffles and a join. The local and cloud WordCount runs are a like-for-like comparison. Spark shows that the cloud job's overhead is significant compared with the actual computation.
<!--/answer-->

**What the counts tell you.** From your stage count: how many shuffles did the Spark job have,
and which operations in `spark_tfidf.py` caused them? From your retry count: roughly what
fraction of the 16 task attempts was retried, and is that what `--chaos 0.3` should produce?
(~50 words)

<!--answer:counts_interpretation-->
The Spark job had 4 stages and 3 shuffle boundaries, caused by the two reduceByKey operations and the join. One retry was observed among the 16 initial logical tasks (12 map tasks and 4 reduce tasks), corresponding to approximately 6.25%. The configured chaos probability was 0.3 (30%), but this does not guarantee that exactly 30% of tasks will fail: the actual number of failures depends on random outcomes during execution.
<!--/answer-->

---

## 2. Spark: transformations, actions, lineage

**Transformations vs actions.** Go through `spark_tfidf.py` and the notebook's Spark cells.
Name each RDD operation and say whether it is a transformation or an action. What happened
when you ran `build_tfidf_rdd` — and what happened when you ran `collect`? (~60 words)

<!--answer:transformations_vs_actions-->
Operations such as flatMap, map, reduceByKey and join are transformations: they create new RDDs and extend the lineage, but are evaluated lazily. build_tfidf_rdd therefore constructs the TF-IDF computation without executing the complete pipeline. collect is an action: it triggers execution of the required lineage and stages, then brings the resulting records back to the driver.
<!--/answer-->

**Lineage and recovery.** Using the lineage you printed in 2b and the timeline in 3a: if the
worker holding one output partition of the *first* `reduceByKey` died, what would Spark have
to recompute, and what could it reuse? Name the stage or shuffle boundary where the recovery
starts. (~50 words)

<!--answer:lineage_recovery-->
If a worker holding an output partition of the first reduceByKey died, Spark could recompute that partition from its lineage. Recovery would start from the shuffle boundary created by the first reduceByKey: Spark would rerun the map-side computation for the lost shuffle partition and then recompute the affected reduce partition. It could reuse unaffected partitions and completed shuffle outputs that were still available.


<!--/answer-->

---

## 3. Your cloud MapReduce, mapped onto Hadoop

Name the GCP service or file that played each role (one line each):

- **JobTracker** (schedules tasks, waits for the map phase, retries failures):
  <!--answer:role_jobtracker-->Google Cloud Workflows, through the workflow orchestration and retry mechanism.<!--/answer-->
- **Workers** (run your map and reduce code):
  <!--answer:role_workers-->The Cloud Run services or worker containers executing the map and reduce tasks.<!--/answer-->
- **HDFS** (holds inputs, shuffle files, outputs):
  <!--answer:role_hdfs-->Google Cloud Storage (GCS), which stores the input data, intermediate shuffle files and final output.<!--/answer-->
- **Partitioner** (decides which reducer owns a key):
  <!--answer:role_partitioner-->The partitioning function in the MapReduce implementation, which assigns each key to a reducer partition.<!--/answer-->

**The barrier.** There is no explicit "wait for all mappers" step in `workflow.yaml`. Where,
exactly, is the barrier between the map and reduce phases — which construct provides it?
(~30 words)

<!--answer:barrier_location-->
The barrier is provided by the workflow's dependency on the map-task results: the reduce phase is reached only after the map phase's parallel task calls have completed successfully. The workflow orchestration therefore acts as the barrier between map and reduce.
<!--/answer-->

**Idempotence.** The workflow retries a task that failed. Why is that safe with the worker as
written, and what would go wrong if `run_map` *appended* its pairs to the shuffle file instead
of overwriting it? (~60 words)

<!--answer:idempotence-->
Retrying a task is safe because the worker overwrites its output shuffle file rather than appending to it. Running the same map task again therefore replaces the previous output with the same deterministic result. If run_map appended pairs instead, a retry could duplicate intermediate key-value pairs in the shuffle file, causing reducers to count some words more than once and produce an incorrect word count.
<!--/answer-->

**The chaos run.** What did you see in the execution log and in Logs Explorer when you ran
with `--chaos 0.3`? Did the job still succeed, and was the final answer still identical to the
local reference? (~50 words)

<!--answer:chaos_observations-->
With --chaos 0.3, the execution logs showed simulated HTTP 503 errors in the call_map task. The workflow automatically retried the failed Map task, and the retry succeeded. The overall workflow finished with state SUCCEEDED. The cloud word count was 2,221, matching the local word count of 2,221, so the final result was identical despite the injected failure.
<!--/answer-->
