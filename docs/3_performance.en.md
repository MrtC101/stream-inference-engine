# Performance

This section describes the performance evaluation model of the system in the MVP context. It defines the validity scope, the formal criteria for "real time", the metrics used and their limitations. It does not constitute a comparative benchmark nor does it expose maximum system capacity.

---

## Scope and Validity Context

The considerations in this document are valid only under a controlled environment equivalent to the one used during MVP development:

- x86_64 architecture
- NVIDIA datacenter-class GPU
- Multi-core CPU
- Compatible CUDA + TensorRT + DeepStream
- Homogeneous streams
- Resolutions between 480p and 1080p
- Input FPS in the 25–30 range

Any variation in hardware, drivers, models, or configuration invalidates direct extrapolation of the described behavior.

---

## Load Definition

**Load** is understood as the number of streams processed in parallel, each associated with one or more inference models.

Increasing load impacts:

- Effective FPS per stream
- Aggregate frame drop rate
- Average latencies
- GPU and memory utilization

The system exhibits progressive degradation when resource contention prevents sustaining real-time processing for all active streams.

---

## Acceptance Criteria (Real-Time)

System state is classified according to the **aggregate frame drop rate**:

- **0% – 1%**
  Optimal real time within MVP scope.
- **1% – 5%**
  Acceptable performance for functional validation.
- **> 5%**
  The service is no longer considered real time in the MVP context.

These thresholds are defined for internal technical validation purposes and do not represent production SLAs.

The real-time state of the system is determined using the aggregate value corresponding to `workers_frame_drop_rate_avg`.

---

## Measurement Methodology (High Level)

- Metrics exclude warm-up and pipeline creation.
- Captured only under stable steady state.
- Stream homogeneity is assumed.
- Per-stream metrics are averaged over time.
- System metrics are periodic aggregations of all per-stream metrics.
- No outlier detection or removal is performed.

The goal is to characterize general behavior under load, not to perform exhaustive statistical analysis.

---

## Metrics Used

The metrics used are aggregates persisted by the Engine and derived from the `system_metrics` model.

The following are considered:

### Worker pipelines
- `workers_fps_avg`
- `workers_frame_drop_rate_avg`
- `workers_processing_latency_avg_ms`
- `workers_e2e_latency_avg_ms`

### Factory pipelines
- `factories_fps_avg`
- `factories_e2e_latency_avg_ms`

### System resources
- `gpu_compute_percent`
- `gpu_memory_percent`
- `ram_percent`
- `cpu_percent`

**Frame drop rate** is the primary evaluation metric, as it directly represents the system's inability to sustain the frame flow in real time.

---

## Behavior Under Increasing Load

As the number of parallel streams increases:

- GPU usage tends to grow until approaching the saturation point.
- Frame drop rate increases non-linearly.
- Average end-to-end latency increases due to resource contention.
- Effective FPS per stream may degrade progressively.

The degradation point is not a theoretical property of the hardware, but a consequence of the current implementation state.

The quantitative limit of sustainable streams is not published in this document.

---

## Measurement Limitations

- Jitter is not measured.
- Fairness between streams is not measured.
- Real end-to-end latency with full frame traceability is not calculated.
- Inference accuracy or quality is not evaluated.
- Deep statistical analysis is not performed.

Loss of frame identity between pipelines prevents exact calculation of real end-to-end latency.

---

## Implications and Expected Evolution

The current operational limit of the MVP is determined primarily by implementation efficiency and resource usage, not by structural architectural constraints.

Improvements in:

- Memory management
- Reduction of unnecessary copies
- Dynamic load control
- More precise metric instrumentation

should allow increasing effective capacity while maintaining the defined real-time thresholds.

Future evolution prioritizes per-stream horizontal scalability and explicit degradation control.

---

## Out of Scope

This chapter does not cover:

- Exact maximum system capacity
- Detailed benchmarking procedures
- Comparisons with other implementations
- Production SLAs
- Multi-node scalability
