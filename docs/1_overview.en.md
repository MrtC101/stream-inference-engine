# Overview ***draft***

# Stream Inference Engine

This project implements a real-time video inference system designed to process multiple concurrent RTSP streams under strict latency and resource constraints.

The system targets continuous operation (24/7) on a single GPU host, prioritizing predictable degradation over maximum throughput. The primary design goal is not peak performance, but controlled behavior under load: bounded latency, explicit backpressure, and failure isolation between streams.

This documentation describes the system from an engineering perspective, focusing on architectural boundaries, performance characteristics, and design trade-offs rather than implementation details.



This project presents a technical overview of a [MindColab](http://mindcolab.com)'s new *real-time video processing* ***engine*** designed to use interchangeable *AI* models to perform inference over multiple continuous video streams. The system, hereafter referred to as *inference engine*, acts as a core piece for a microservice architecture used to deliver a commercial service called "ironstream". To make the *engine* a reliable solution, the implementation and internal design need to meet key requirements such as *24/7 availability*, *efficient use of computational resources*, *state-of-the-art frameworks* and *extensibility and customization of inference features*.

The primary goal is not to showcase a finished product, but to **document engineering decisions**, trade-offs, and real-world limitations encountered while building a functional *MVP* under practical constraints such as time, hardware, and system complexity.

The system evolved from an initial CPU-only implementation—unable to sustain real-time performance—into a GPU-accelerated architecture based on **GStreamer**, **CUDA/NVMM**, and decoupled inference components. 

---

The first CPU-bound approach took over 3 months of implementation to reach a functional starting point. This implementation included
- Project setup using uv 
- Development container with cpu-bound libraries 
- Configuration yaml field validation using Cerverus
- Design of a DSL grammar for Inference Rules and implementation of a interpreter built with lark
- DSL to create yaml configurations and templates to inference specifications and streams configuration.
- OpenCV based approach to read from stream sources.
- A zero-copy Frame transmition using numpy.
- Motion detection mechanism to trigger inference over specific frames.

The second step was to implement a GPU-bound apporach over the next 4 months. 
- Re-implementation of zero-copy approach using pytorch tensors.
- Re design  Development container to install GPU specifi frameworks as Cuda, CDNN, Deepstream, Gstreamer
- First intent of a production ready docker container
- Inference model architecture to add Inference models extensibiliy
- Implementing a multiprocessing approch with custom IPC and shared memory approach to transfer frames between piplines using gstreamr's shmsrc.
- Implementing a numba kernel to draw lines in a GPU
- Integratting drawing module to show visual metada and text using deepstream plugin nvdsosd
- Implement and optimize dockerfile and docker compose file to build the lightests posible docker image
- Developed simple API to query stream processing jobs called "runs" and store the in a sqlite database.
- Implemented mechanism to read database runs, update state
- Implemente engine metric capture mechanism to evaluate performance
. notebook with analisis of system over stress with the inference engien in mvp state.

## Project Scope

The current scope of the project includes:

- Real-time video stream ingestion.
- GPU-accelerated processing and inference.
- Re-streaming of processed video with bounded latency.
- Continuous operation in production-like environments (24/7).
- Hot-reload support for selected configuration parameters.

Explicitly **out of scope** at this stage:

- Automatic horizontal scaling.
- Advanced orchestration (Kubernetes, autoscaling).
- Fully guaranteed end-to-end zero-copy.
- Dynamic redeploy without process restart.
- Exhaustive performance analysis and deep profiling.

---

## Design Principles

Engineering decisions throughout the project prioritized:

- **Operational simplicity** over maximum flexibility.
- **Clear separation of responsibilities** between pipelines.
- **Minimal yet sufficient observability** for an MVP.
- **Incremental evolution** instead of premature over-engineering.
- Explicit use of **NVMM buffers** to reduce unnecessary memory copies.

The system is designed to be **understandable, modifiable, and extensible**, even where optimizations are intentionally left unimplemented.

---

## Current State (MVP)

In its current state, the system:

- Operates in real time under controlled conditions.
- Measures end-to-end latency in independent pipeline segments.
- Uses GPU resources consistently, with basic metrics for GPU, VRAM, and RAM utilization.
- Supports controlled restarts, graceful shutdown, and partial configuration hot-reload.
- Contains known inefficiencies, particularly in rendering and frame drawing stages.

This state is considered **intentionally incomplete**, but stable and observable.

---

## Target Audience

This documentation is intended for:

- Engineers interested in **real-time video architectures**.
- Technical reviewers evaluating **engineering judgment**, not just features.
- Teams that value **explicit acknowledgment of limitations** over polished claims.

It is not meant to be end-user documentation or marketing material.

---

## How to Read This Documentation

The documentation is structured progressively:

1. Overall system architecture.
2. Performance and latency considerations.
3. Key engineering decisions.
4. Current limitations and technical debt.
5. Production and operational considerations.

Each section can be read independently, but the full value emerges from understanding the system as a whole.
