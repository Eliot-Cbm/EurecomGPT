# Phase 1 — Comparison Report

Copy this file to `submission/phase1_report.md`, fill in every answer slot, and commit it:

```bash
cp phase-1-echo-bot/report_template.md submission/phase1_report.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or images freely.

Numbers must **match `submission/phase1_report.json`** exactly — that file is produced by
`measure.py` and is what your figures are checked against. Copy them across; do not retype from
memory.

---

## 1. Setup facts

**Your image size**, in MB, from `gcloud artifacts docker images list` (number only, e.g. `142`):

<!--answer:image_size_mb-->
49.99
<!--/answer-->

**The custom VPC and subnet you created** (names only):

<!--answer:vpc_name-->
echo-net
<!--/answer-->

<!--answer:subnet_name-->
echo-subnet
<!--/answer-->

---

## 2. Measurements

Copy these from `submission/phase1_report.json`. Cold is `cold_ms`; warm median is
`stats.median`. Numbers only, in milliseconds (e.g. `812.4`).

| Platform | Cold (ms) | Warm median (ms) |
|---|---|---|
| IaaS VM | <!--answer:vm_cold_ms-->232.8<!--/answer--> | <!--answer:vm_warm_median_ms-->211.6<!--/answer--> |
| Cloud Run | <!--answer:cloudrun_cold_ms-->4179.1<!--/answer--> | <!--answer:cloudrun_warm_median_ms-->644.4<!--/answer--> |
| Cloud Function | <!--answer:function_cold_ms-->1312.3<!--/answer--> | <!--answer:function_warm_median_ms-->636.0<!--/answer--> |

Include your histogram or plot of the warm distribution below (an image, or the ASCII chart
`measure.py` prints). This is read by a human, not the grader:

<!--answer:latency_plot-->
Phase 1 — deployment measurements

VM        cold=  232.8ms  warm_median=  211.6ms
#############

Cloud Run cold= 4179.1ms  warm_median=  644.4ms
########################################

Function  cold= 1312.3ms  warm_median=  636.0ms
#######################################
<!--/answer-->

---

## 3. Per-platform analysis

**Scaling behaviour.** Imagine requests arriving one at a time, then ten at once, then a
hundred. For each of the three platforms, answer concretely:

- Does anything get created automatically to absorb the load — and if so, *what* exactly?
  (One platform here creates nothing at all.)
- What is the hard limit, and where does it come from — a flag you passed yourself, or the
  size of the machine you picked?

(~80 words. Your `--machine-type=e2-micro` and `--max-instances=3` are both concrete answers
to the second question.)

<!--answer:scaling_behaviour-->
The VM uses one `e2-micro` machine and does not create new VMs automatically when traffic increases. Its capacity depends on its CPU and memory. Cloud Run can create more container instances to handle extra requests, but it is limited to 3 instances by `--max-instances=3`. Cloud Functions Gen 2 can also scale automatically by creating more instances when needed, depending on its configuration and platform quotas.
<!--/answer-->

**Deployment effort.** What did you have to build, configure and maintain to get each of the
three running? (~100 words)

<!--answer:deployment_effort-->
Deploying the VM took the most work because I had to configure the network, firewall rules and machine, then build and run the Docker container. For Cloud Run, I built the Docker image, pushed it to Artifact Registry and deployed the service. For Cloud Functions, I prepared the Python code and deployed it as an HTTP function. Google manages most of the infrastructure for Cloud Run and Cloud Functions, so they need less maintenance than the VM. Overall, the VM gives more control, but the managed platforms are easier to maintain.
<!--/answer-->

---

## 4. Why do two of your three numbers look alike?

With Cloud Run and your Function both deployed, run `gcloud run services list` and paste the
**complete output** here, unedited:

<!--answer:run_services_list-->
✔
SERVICE: echo
REGION: us-central1
URL: https://echo-798783231347.us-central1.run.app
LAST DEPLOYED BY: service-798783231347@gcf-admin-robot.iam.gserviceaccount.com
LAST DEPLOYED AT: 2026-10-16T15:45:07.620385Z

✔
SERVICE: echo-bot
REGION: us-central1
URL: https://echo-bot-798783231347.us-central1.run.app
LAST DEPLOYED BY: eliot.corbehem@gmail.com
LAST DEPLOYED AT: 2026-10-16T15:39:48.077938Z
<!--/answer-->

Now explain what that output tells you about how your Function is actually being run, why its
cold start resembles Cloud Run's so closely, and what — if anything — genuinely differs between
the two deployment paths. (~120 words)

<!--answer:convergence_explanation-->
The output shows two services called `echo` and `echo-bot`. The `echo` service was deployed using Cloud Functions Gen 2, while `echo-bot` was deployed directly to Cloud Run. Cloud Functions Gen 2 uses Cloud Run infrastructure, which explains why their cold start times can be similar. Both services may take longer to respond when a new instance needs to start. The main difference is how they are deployed. For Cloud Run, I built and uploaded a Docker image. For Cloud Functions, I deployed my Python code and selected a runtime and an entry point. Google manages most of the infrastructure for both services.
<!--/answer-->
