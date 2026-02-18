# Limitations

This section explicitly states the technical constraints observed in the current MVP state, differentiating limits derived from implementation, environment, and scope priorities from structural design limitations. It defines the current operational boundaries of the system in terms of concurrency, zero-copy, metrics, scalability, and observability, establishing which behaviors are inherent to the present state and which constitute planned technical debt for future evolution.

---

## Stream Concurrency Limit (MVP)

- The MVP currently supports up to **6 concurrent streams** with acceptable performance.
- This limit arises from **aggregate resource consumption** (RAM, VRAM, hardware encoder/decoder, and GPU compute), not from a logical system constraint.
- The limit **is not imposed by configuration nor hardcoded**; it is the point at which the frame drop rate exceeds acceptable real-time thresholds.
- With additional technical optimization, the system is estimated to scale to **~10 streams** on the same hardware, depending on:
  - resolution,
  - input FPS,
  - model complexity,
  - type of inference task.
- Even with optimization, the hardware imposes a **practical ceiling** that limits scalability without increasing resources.

---

## Zero-Copy Limitations

- A **complete zero-copy** scheme throughout the entire pipeline has not been achieved.
- On Jetson platforms, although RAM and VRAM share the same chip, full zero-copy is not viable within the current limits of:
  - Python,
  - CUDA,
  - PyTorch,
  - DeepStream,
  - GStreamer.
- During the drawing phase:
  - certain operations (e.g., segmentation) are simpler to implement on CPU using OpenCV,
  - which introduces explicit or implicit memory copies.
- Some models require `float32` input, implying additional conversions and copies.
- In practice, inference frameworks perform internal copies, so this behavior is expected.
- Conclusion: a **best-effort zero-copy** approach is applied, keeping frames on GPU as long as possible, but accepting copies when necessary.

---

## Metrics and Observability Limitations

- The architecture based on **two pipelines decoupled via shared memory** causes **loss of frame identity** between pipelines.
- This prevents precise measurement of:
  - real end-to-end latency,
  - effective throughput,
  - frame-to-frame correlation between input and output.
- Currently:
  - **frame drop rate** is used as the primary metric to evaluate real-time behavior,
  - other metrics are approximated.
- Implementing full traceability would require:
  - extending the shared memory schema with metadata headers,
  - or introducing explicit inter-process synchronization mechanisms.
- Given the MVP scope, this improvement is considered **technical debt**.

---

## MVP Scope Limitations

- The system was developed as an MVP oriented toward **technical and business validation**, not as a final product.
- Not implemented:
  - resource-based load control mechanisms,
  - dynamic run execution limits,
  - automatic scaling.
- The number of supported inference frameworks is limited:
  - YOLO as the primary framework,
  - limited support for Anomalib.
- Inference quality depends exclusively on the loaded model; the system does not apply semantic validation of results.
- The system does not guarantee:
  - source accessibility,
  - network stability,
  - input image quality,
  - behavior of stream consumers.

---

## Resolved Technical Debt

- The **full frame cache in CPU within the drawing module** was eliminated, reducing the excessive RAM consumption observed in previous versions. The module retains cache of inference results and drawing metadata between frames, but no longer stores frame copies in CPU memory.

---

## Deferred Improvements

The following limitations are considered addressable in future development phases:

- Additional optimization of RAM and VRAM memory consumption.
- Improved run lifecycle and greater resilience to failures.
- Implementation of resource metrics-based load control.
- Improved metrics and frame traceability.
- Expanded framework and model type support.
