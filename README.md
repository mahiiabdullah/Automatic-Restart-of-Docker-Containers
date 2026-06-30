# Automatic Restart of Docker Containers using Puku CLI

## Introduction

Docker containers provide isolated environments for deploying applications consistently across systems. In production environments, containers may fail due to application crashes, configuration errors, or resource exhaustion. Service reliability depends on recovering from such failures safely and efficiently.

Docker provides restart policies to automatically restart failed containers. To prevent restart storms during repeated failures, Docker applies exponential backoff, increasing the delay between restart attempts.

This lab explores Docker restart behavior using Puku CLI, an MCP-powered AI terminal assistant capable of executing infrastructure operations through natural language instructions.

---

## Learning Objectives

By the end of this lab, you will be able to:

1. Configure Docker containers with different restart policies
2. Analyze container behavior under successful and failed exits
3. Observe Docker exponential backoff during crash loops
4. Diagnose container restart issues using metadata and logs
5. Use Puku CLI for AI-assisted DevOps workflows

**Prerequisites:** Basic Docker knowledge and familiarity with container concepts.

---

# Prologue: The Challenge

You join the reliability engineering team at a company running multiple containerized services. One service repeatedly crashes after deployment. Engineers observe:

* Frequent container restarts
* Increased CPU usage
* Excessive log generation
* Reduced system stability

Your task is to investigate:

* Should the container restart automatically?
* Which restart policy is appropriate?
* How does Docker avoid restart storms?
* How can AI-assisted tooling improve debugging?

You will use Puku CLI to investigate the system.

---

# Environment Setup

Ensure the following are installed:

* Docker
* Puku CLI
* Running Docker daemon

Start Puku CLI:

```text
puku-cli
```

Login and then Verify Docker access.

Prompt to Puku:

```text
Check whether Docker is installed and whether the Docker daemon is running.
```

Expected output:

* Docker version information
* Docker daemon active

Add your screenshot here:

![Terminal capture: Puku verifying Docker installation and daemon status](image/CLI%20Commands%20and%20reply/setup.png)

> Note: Output may vary depending on Docker version.

---

# Chapter 1: Understanding Restart Policies

Restart policies define how Docker responds when containers stop running.

---

## 1.1 What You Will Build

You will explore these restart strategies:

* no
* on-failure
* on-failure:N
* always

---

## 1.2 Think First: Exit Codes

A container exits with status code `0`.

**Question:** Should Docker interpret this as a failure?

<details>
<summary>Click to review</summary>

No.

Exit code `0` indicates successful completion. Non-zero exit codes indicate failure.

</details>

---

## 1.3 Implementation

Prompt to Puku:

```text
Explain Docker restart policies with practical examples.
```

Record important observations.

---

## 1.4 Assessment

Match each policy to behavior.

| Policy         | Behavior |
| -------------- | -------- |
| no             | ___      |
| on-failure     | ___      |
| always         | ___      |
| unless-stopped | ___      |

Options:

* A: Restart only if the container exits with a non-zero code.
* B: Always restart unless the container was explicitly stopped.
* C: Never restart
* D: Restart regardless of exit reason

<details>
<summary>Show Answer</summary>

* no → C
* on-failure → A
* always → D
* unless-stopped → B

</details>

---

## 1.5 Checkpoint

**Self-Assessment**

* [ ] You understand all restart policies
* [ ] You understand exit codes
* [ ] You can predict restart behavior

---

# Chapter 2: Container Without Restart Policy

Understanding default behavior provides the baseline for comparison.

---

## 2.1 What You Will Build

Create a container that runs once and exits permanently.

---

## 2.2 Think First

If a process finishes normally and no restart policy exists, what happens?

<details>
<summary>Click to review</summary>

The container stops permanently.

</details>

---

## 2.3 Implementation

Prompt to Puku:

```text
Create a Docker container named no-restart using BusyBox. It should print the current UTC time once and exit. Do not apply any restart policy. Then show the container status.
```

Output:

![Terminal capture: Puku creating no-restart container and showing its status](image/CLI%20Commands%20and%20reply/no-restart.png)

---

## 2.4 Observation

Record:

* Did container restart?
* What state is it in?

---

## 2.5 Quick Assessment

Why did the container stop permanently?

A. Docker bug
B. No restart policy configured
C. BusyBox failed

<details>
<summary>Show Answer</summary>

B — No restart policy configured.

</details>

---

## 2.6 Checkpoint

**Self-Assessment**

* [ ] Container executed once
* [ ] Container stopped permanently
* [ ] You understand default behavior

---

# Chapter 3: Restart on Failure

Some workloads should restart only after failure.

---

## 3.1 What You Will Build

Create a container that fails intentionally.

---

## 3.2 Think First

A process exits with code `1`.

**Question:** What does Docker interpret this as?

<details>
<summary>Click to review</summary>

Failure.

</details>

---

## 3.3 Implementation

Prompt to Puku:

```text
Create a container named fail-restart that exits with status code 1 and uses Docker restart policy on-failure. Monitor its behavior.
```

Output:

![Terminal capture: Puku creating fail-restart with on-failure policy and confirming restart loop](image/CLI%20Commands%20and%20reply/on-failure.png)

---

## 3.4 Observation

Record:

* Did restart occur?
* Why?

---

## 3.5 Prediction Exercise

Will this restart?

* Exit code = 0
* Restart policy = on-failure

<details>
<summary>Show Answer</summary>

No.

`on-failure` only triggers on non-zero exit codes. Exit code 0 is treated as success, so Docker leaves the container in `Exited (0)` and does not restart it.

</details>

---

## 3.6 Student Task

Use Puku to create a container with:

* restart policy: `on-failure:3`
* exit code: `1`

Record:

* Total restart attempts
* Final container state

---

## 3.7 Checkpoint

**Self-Assessment**

* [ ] You understand on-failure
* [ ] You understand retry limits
* [ ] You can predict behavior

---

# Chapter 4: Automatic Restart and Exponential Backoff

Repeated failures can overload systems if restart attempts happen too frequently.

Docker uses exponential backoff to reduce restart frequency.

---

## 4.1 What You Will Build

Create a crash-looping container and observe backoff.

---

## 4.2 Think First

A container crashes every 0.2 seconds.

**Question:** What happens if Docker restarts it immediately forever?

<details>
<summary>Click to review</summary>

CPU usage increases, logs flood, and resources become exhausted.

</details>

---

## 4.3 Implementation

Prompt to Puku:

```text
Create a Docker container named backoff-detector using BusyBox that prints the current UTC time and exits immediately. Apply restart policy always and continuously monitor restart behavior.
```

Output:

![Terminal capture: Puku creating backoff-detector with restart policy always](image/CLI%20Commands%20and%20reply/backoff-create-1.png)

![Terminal capture: Puku confirming backoff-detector is running and restarting](image/CLI%20Commands%20and%20reply/backoff-create-2.png)

---

## 4.4 Predict Before Observation

Will restart intervals remain constant?

* Yes
* No

<details>
<summary>Show Answer</summary>

No.

</details>

---

## 4.5 Observe Logs

Prompt to Puku:

```text
Show restart timestamps for backoff-detector.
```

Output:

![Terminal capture: Puku showing backoff-detector restart timestamps](image/CLI%20Commands%20and%20reply/backoff-logs.png)

## 4.6 Analyze Backoff

Prompt to Puku:

```text
Analyze restart timestamps and calculate restart delays.
```

Expected pattern:

![Terminal capture: Puku computing restart delays from timestamps](image/CLI%20Commands%20and%20reply/restart-delays.png)

## 4.7 Assessment

Why does Docker use exponential backoff?

A. Faster restart
B. Prevent restart storms
C. Reduce image size

<details>
<summary>Show Answer</summary>

B — Prevent restart storms.

</details>

---

## 4.8 Checkpoint

**Self-Assessment**

* [ ] You observed exponential backoff
* [ ] You understand restart storms
* [ ] You understand why backoff exists

---

# Chapter 5: Interaction During Backoff

A restarting container is not always available for interaction.

---

## 5.1 Think First

Can commands execute inside a restarting container?

<details>
<summary>Click to review</summary>

No.

</details>

---

## 5.2 Implementation

Prompt to Puku:

```text
Try executing "echo hello" inside backoff-detector while it is restarting.
```

Output:

![Terminal capture: Puku failing to exec into backoff-detector during restart loop](image/CLI%20Commands%20and%20reply/exec-failure.png)

Expected:

* container restarting error

---

## 5.3 Understanding Failure

Prompt to Puku:

```text
Explain why command execution failed during restart backoff.
```

---

## 5.4 Quick Assessment

Why did execution fail?

A. BusyBox missing
B. Container not running
C. Docker permissions issue

<details>
<summary>Show Answer</summary>

B — Container not running.

</details>

---

## 5.5 Checkpoint

**Self-Assessment**

* [ ] You understand container states
* [ ] You know why exec failed
* [ ] You understand restart delay

---

# Chapter 6: Inspect Container Metadata

Metadata provides valuable debugging signals.

---

## 6.1 What You Will Build

Inspect restart count and state.

---

## 6.2 Think First

Which metric indicates repeated restart attempts?

<details>
<summary>Click to review</summary>

RestartCount

</details>

---

## 6.3 Implementation

Prompt to Puku:

```text
Inspect backoff-detector and show container state and restart count.
```

Output:

![Terminal capture: Puku inspecting backoff-detector metadata showing state and RestartCount](image/CLI%20Commands%20and%20reply/inspect.png)

![Terminal capture: Puku computing restart delays from timestamps](image/CLI%20Commands%20and%20reply/restart-delays.png)

Expected:

```text
State: restarting
RestartCount: 13
```

---

## 6.4 Understanding Metadata

Question:

What does a high RestartCount suggest?

<details>
<summary>Click to review</summary>

The application repeatedly fails or exits.

</details>

---

## 6.5 Checkpoint

**Self-Assessment**

* [ ] You can inspect metadata
* [ ] You understand RestartCount
* [ ] You can diagnose crash loops

---

# Chapter 7: Experiment — Deliberate Failure

Intentional failure reveals why restart policies matter.

---

## 7.1 Experiment

Prompt to Puku:

```text
Compare repeated crash behavior using:
1. no
2. on-failure
3. always
4. unless-stopped
```

---

## 7.2 Assessment

Which policy provides maximum availability?

A. no
B. on-failure
C. always
D. unless-stopped

<details>
<summary>Show Answer</summary>

D. unless-stopped

</details>

---

## 7.3 Practical Task

Use Puku to create a container that:

* exits every 3 seconds
* uses restart policy `always`

Observe:

* restart count
* delay pattern
* system behavior

Record findings.

---

# Epilogue: The Complete System

You investigated Docker restart behavior using AI-assisted infrastructure operations.

You explored:

| Feature    | Purpose                |
| ---------- | ---------------------- |
| no         | No automatic restart   |
| on-failure | Restart after failure  |
| always     | Continuous restart     |
| Backoff    | Prevent restart storms |

You used Puku CLI for:

* infrastructure inspection
* container creation
* monitoring
* analysis
* debugging

---

# Final Assessment

Complete the following tasks using Puku CLI.

---

## Task 1

Create a container that exits successfully using restart policy `always`.

Questions:

* Does it restart?
* Why?

---

## Task 2

Create a container with:

* restart policy: `on-failure:3`
* exit code: `1`

Questions:

* How many restarts occurred?
* What is the final state?

---

## Task 3

Explain:

1. Why Docker uses exponential backoff
2. Why restart policies cannot replace debugging
3. When Kubernetes is preferable to Docker restart policies

---

# The Principles

1. Automatic recovery improves availability
2. Restart policies must match workload behavior
3. Restart does not fix application bugs
4. Backoff prevents infrastructure overload
5. Observability is essential for debugging

---

# Troubleshooting

## Error: Docker daemon unavailable

**Cause:** Docker service not running.

**Solution:** Start Docker service.

---

## Error: Puku cannot access Docker

**Cause:** MCP permissions missing.

**Solution:** Verify Docker tool access in Puku.

---

## Error: Container stuck restarting

**Cause:** Application exits immediately.

Prompt to Puku:

```text
Inspect logs of backoff-detector and identify root cause of repeated exits.
```

---

# Next Steps

Extend this lab by exploring:

1. Docker health checks
2. Multi-container systems
3. Process supervision with supervisord
4. Orchestration using Kubernetes

Advanced prompt:

```text
Compare Docker restart policies with Kubernetes pod restart behavior.
```

---

# Additional Resources

* [Docker Restart Policies Documentation](https://docs.docker.com/engine/containers/start-containers-automatically/?utm_source=chatgpt.com)
* [BusyBox Documentation](https://www.busybox.net/?utm_source=chatgpt.com)

---

# Final Conceptual Questions

1. Why is immediate infinite restart dangerous?
2. Why does `on-failure` ignore exit code 0?
3. Why is restart policy not a substitute for debugging?
4. When is Kubernetes preferable to raw Docker?
