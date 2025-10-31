# CS 5416: Final Project
## Building and Optimizing a Distributed ML Inference Pipeline

**Out:** November 4  
**Preliminary Check (required but not graded):** December 1  
**Final Project Due:** December 12  
**Work in groups of 4 (minimum 3)**

## 1. Introduction

In real production systems, machine learning is almost never "just call the model." Inputs are validated, transformed, enriched, routed, and sometimes filtered before the model ever runs. Then the model output is post-processed, checked for safety, logged, and sent to other services.

One real-world example of this would be an LLM with tool calls or web search. That system might operate like:
- User sends query
- System classifies intent
- System calls a vector store / web search / internal tool
- Retrieved content is merged into a prompt
- LLM is called with context
- Output is safety-filtered and returned

Another example might be a fraud detection pipeline that operates as:
- Transaction arrives
- Features are constructed from recent customer history
- Model is run
- Output is thresholded and maybe sent to a second model or rule engine
- Alert is sent to customer service or the transaction is blocked

The point is: production ML = pipelines. And pipelines = systems problems (distribution, batching, network, memory, GPU/CPU placement, orchestration, correctness).

This project is designed to give you hands-on practice in taking a naive, monolithic ML pipeline and turning it into a distributed, optimized version. You will explore various optimization strategies and profile real tradeoffs (memory vs. batch size, batch size vs. throughput, CPU vs. GPU).

This is the kind of work ML systems engineers actually do.

## 2. Objective

You will be given a barebones monolithic implementation of an ML pipeline (link provided in the starter code section). Your job is to:

1. **Transform it into a distributed service that runs across 3 nodes**
2. **Implement batching** (batch requests together when possible)
3. **Profile your design** and report on the effects of different optimizations
4. **Show that you did systems work** (even if an unoptimized version is faster)

You may employ advanced tactics for higher grades such as:
- Breaking apart the monolithic pipeline into microservices
- Implementing opportunistic batching
- Caching strategies
- GPU acceleration
- Data compression for network transfer
- And more... This is an open ended project, so any other optimizations that you can think of, you may implement

## 3. Grading Tiers

### **Grading Philosophy**
Grades for this project will be primarily based on how many systems concepts you successfully implement and analyze, NOT on raw performance. A slower but well-engineered microservice implementation with thorough profiling and a good report will still receive a good grade.

### Tier 1: Passing Grade (B to B+ range)
**Minimum Requirements:**
- Pipeline successfully runs across 3 nodes (can be 3 monolithic instances)
- Basic batching implementation (static batch sizes are fine)
- Node 0 receives client requests and returns responses
- Basic profiling showing throughput and latency of the pipeline vs batch size
- Correctness maintained
- Code runs within hardware constraints (16GB RAM per node)

### Tier 2: Good Grade (A- to A range)
**All Tier 1 requirements PLUS:**
- True microservices architecture (pipeline stages separated into services)
- Opportunistic batching: Each step or microservice can take different batch sizes depending on load
- Orchestration and request routing across nodes
- Per-node memory and full pipeline throughput profiling of at least 2 optimizations (e.g. which services to put on which nodes, maximum batch size, 3 monolithic pipeliens versus microservices implementation, etc)
- Memory, throughput, and latnecy vs batch size analysis for each pipeline step
- Clear documentation of design decisions in report
- Explanation of how your profiling led to your final configuration

### Tier 3: Excellent Grade (A to A+ range)
**All Tier 2 requirements PLUS at least 2 of:**
- A switch to turn on GPU acceleration
- Caching strategies
- Data compression for inter-node communication
- Exceptional profiling with detailed analysis of system bottlenecks
- Top tier performance on the leaderboard
- Fault tolerance or recovery mechanisms
- Other creative optimizations

## 4. Scenario

You've been "hired" by a large e-commerce company to fix their customer support AI pipeline. Their current version is a single Python script that runs everything in one process. It cannot handle production load.

Your task: implement the pipeline as a distributed system across 3 nodes.

The pipeline stages are:
1. **Tokenizer** on the query from the client
2. **Embedder** on the tokenized query
3. **FAISS RAG lookup** (ANN) — retrieve top-10 docs using the embedding
4. **Fetch documents** from disk based on step 3
5. **Reranker** on the docs from step 4
6. **Sensitivity filter** on the docs from step 4
7. **LLM response generation** (max 128 tokens) using the reranked and filtered docs from steps 5 and 6
8. **Sentiment analysis model** on the LLM output from step 7
9. **Sensitivity filter** on the LLM output from step 7
10. **Return to client:**
    - generated response
    - sentiment analysis result
    - sensitivity filter result

**Important:**
- The starter code does this monolithically
- Your submission must do this across 3 nodes
- Our client will only send requests to Node 0; you must route/orchestrate the rest

## 5. Provided Starter Code

Starter code is inside this repository

It Contains a monolithic pipeline that already performs the stages above. You must preserve accuracy/correctness while distributing and optimizing it.

It also contains a client script that follows the same spec that we will use to test your implementations. It is essential that your implemetation is compatible with this script.

Note that the given monolothic pipeline is not at all optimal. Since the combined size of the components is too large to fit in 16gb of memory, it has to remove from and reload components into memory during a single pipeline execution. Furthermore, it only uses a batch size of 1.

## 6. System Requirements / Spec

### 6.1 Overall Requirement
Your pipeline must run on 3 nodes. You must provide:
- `install.sh`
- `run.sh`

We (the TAs) will run three separate processes (one per node), each calling your `run.sh`.

### 6.2 Environment Variables (given by us)
When we run your code, we will provide:
- `TOTAL_NODES` — will be 3
- `NODE_NUMBER` — will be 0, 1, or 2
- `NODE_0_IP`, `NODE_1_IP`, `NODE_2_IP` — IPs for each node

You must use these to:
- Differentiate per-node roles
- Configure services
- Set up routing between nodes

**Client behavior:**
- Our client script will send requests one by one to the Node 0 IP address, port 8000
- Node 0 must orchestrate the full pipeline and return the final response to the client

### 6.3 Request / Response Format
<working on>

Your final response to the client must include:
- `generated_response`
- `sentiment`
- `sensitivity_result`

### 6.4 Implementation Examples

#### Basic Implementation (Passing Grade)
- **Simple distribution:** Can run 3 copies of the monolith, each handling the full pipeline
- **Basic batching:** Fixed batch sizes, process when batch is full
- **Simple routing:** Node 0 can round-robin or randomly distribute to other nodes
- **Basic profiling:** Memory usage, throughput measurements

#### Advanced Implementation (Higher Grades)
- **Microservices:** Separate pipeline stages into distinct services
- **Opportunistic batching:** Each step or microservice can take different batch sizes depending on load
- **Smart orchestration:** Optimize stage placement based on compute requirements
- **Performance-aware data passing:** Minimize data transfer, transfer IDs instead of full documents
- **Comprehensive profiling:** Detailed analysis of bottlenecks and tradeoffs

## 7. Allowed Environment

**Preinstalled:**
- torch
- transformers
- numpy
- datasets
- faiss
- requests
- flask

You may request additional packages from Jamal if:
- They do not require sudo
- They do not abstract away what you are supposed to implement (e.g. you cannot use a library that automatically handles opportunistic batching and routing for you)

You may use any programming language, but Python is recommended (starter code is Python).

## 8. Autograder & Leaderboard Hardware / Runtime Constraints

**3 nodes, each with:**
- CPU: Intel(R) Xeon(R) Gold 6242 @ 2.80GHz
- 16 GB RAM
- GPU: NVIDIA Tesla T4, 16 GB VRAM

**Key note:** 16 GB RAM is a hard practical constraint. If you go over, you'll hit swap → very slow.

**Initialization time:** We will wait for 5 minutes for any initialization you may have before sending your system requests

## 9. GPU Support (Optional)

GPU support is **NOT required** but can earn bonus points for the highest grades.

If you choose to implement GPU support:
- Make an appointment with Jamal to test your GPU-enabled code on the GPU nodes
- Make sure your code can still run fully on CPU
- Document in your report:
  - What you offloaded to GPU
  - Where there was CPU<->GPU transfers
  - Whether you used both CPU and GPU simultaneously

## 10. Deliverables

### 10.1 Codebase
Submit your full codebase with:
- `install.sh` (no sudo)
- `run.sh` (entry point for each node)
- Full source code

### 10.2 Report

**IMPORTANT:** The profiling that you do for the report should be done on UGClinux or your group's local computers. If you implement GPU acceleration, you do not need to worry about running your profiling on GPUs. You will still receive full credit for your GPU implementation even if your report profiling is done on CPU

Your report must include:

#### Required Sections (All Submissions):
1. **System design overview**
   - How you distributed work across 3 nodes
   - How requests flow through your system
   - Diagrams would be helpful here

2. **Batching implementation**
   - What stages use batching
   - Batch size choices and rationale

3. **Basic profiling**
   - Memory usage per node
   - Overall throughput

#### Additional Sections for Higher Grades:
5. **Optimization steps** (if implemented)
   - Microservice architecture details (Diagrams would be helpful here)
   - Opportunistic batching strategy
   - Any other optimizations you implemented (e.g. caching, data transfer, etc)
   - You should both detail the implementation details of your optimization steps as well as justification for why you chose them

6. **Comprehensive experiments** (if performed)
   - Microbenchmarks per stage
   - Memory vs. batch size analysis
   - Throughput vs. batch size analysis
   - Latency vs batch size analysis
   - Analysis of 2+ tunable facets

7. **GPU section** (if implemented)
   - What you offloaded to GPU
   - Where there were CPU<->GPU transfers
   - Whether you used both CPU and GPU simultaneously

## 11. Example Architectures

These architectures may or may not optimal, they are just examples. Do your own analysis!

### Basic Architecture
- Node 0: Full pipeline, receives requests
- Node 1: Full pipeline, processes forwarded requests
- Node 2: Full pipeline, processes forwarded requests
- Simple load distribution from Node 0

### Advanced Architecture
- Node 0: Frontend + tokenizer + embedder + orchestration
- Node 1: FAISS + document retrieval + reranking
- Node 2: LLM + sentiment + sensitivity filters
- Request routing with opportunistic batching at each stage

## 12. Evaluation Criteria

### Core Requirements (Must Have to Pass):
- ✅ Runs on 3 nodes
- ✅ Node 0 handles client interface
- ✅ Some form of batching
- ✅ Maintains correctness
- ✅ Basic profiling data
- ✅ Stays within 16GB RAM limit

### Systems Concepts (More = Higher Grade):
- Microservices architecture
- Opportunistic batching
- Request orchestration
- Performance profiling
- Caching strategies
- GPU utilization
- Network optimization
- Load balancing

**Remember:** We value systems engineering and analysis over raw performance.

## 13. Testing / Logistics

You must test on 3 machines (3 laptops or 3 ugclinux instances). That's why groups of at least 3 are required.

## 14. Timeline

- **November 4** – Project released
- **December 1** – Preliminary submission (required)
  - Used to verify we can run your code
  - No direct effect on final grade
- **December 12** – Final project due

## 15. What to Turn In (Checklist)

### Required:
- [ ] `install.sh` (no sudo)
- [ ] `run.sh` (entry point; respects env vars)
- [ ] Full source code
- [ ] Report (PDF or md) with required sections
