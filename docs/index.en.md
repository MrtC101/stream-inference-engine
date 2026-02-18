# Stream Inference Engine

This document defines the functional and operational scope of the Stream Inference Engine. It describes the problem it solves, the constraints imposed by the MVP context, the technical assumptions under which it operates, and the explicit limits of system responsibility. It does not detail internal implementation or specific engineering decisions, but rather the contractual and operational framework under which the system should be evaluated.

## System Purpose (Problem Space)

- Process multiple concurrent video streams in real time.
- Run inference on video streams and generate visual output.
- Publish the result as a video stream, not as structured metadata.
- Operate in a B2B production context, not as a generic or self-service offering.
- Define "real time" through frame drop rate (< 5% relative to input framerate).

---

## Context Assumptions

- Video sources are external remote cameras.
- Sources expose streams via RTSP.
- Input codec is H.264 (imposed by the MVP context).
- Output protocol is RTSP.
- Inference runs primarily on GPU.
- The system is deployed on x86_64 and arm64 (Jetson) platforms.
- Several technical decisions were imposed by the organizational context of the MVP.

---

## Non-Negotiable Constraints

- The system does not control the protocol or codec of the sources.
- The system does not control network availability or stability.
- Output must remain real time under the defined metric.
- Output is exclusively video with overlays; no additional metadata is emitted.
- Continuity, stability, and latency are prioritized over extreme optimization.
- The service is delivered directly in a B2B context (without automatic elasticity).

---

## System Operational Posture

- The system validates source reachability before starting processing.
- Upon connection loss, the system retries according to configuration.
- If the retry limit is exceeded, the stream is considered failed.
- In the current MVP state, there are no automatic load-limiting mechanisms.
- Only configurations considered acceptable to avoid performance degradation are offered.

---

## Explicit Responsibility Boundaries

- The system does not guarantee inference quality or correctness.
- Accuracy is the responsibility of the loaded model.
- The system does not guarantee source accessibility.
- Network failures are out of scope.
- Failures of the stream emitter are out of scope.
- Failures of output stream consumers are out of scope.
- The system does not guarantee optimal resource utilization in the current MVP state.

---

## Current System State

- The system is in MVP state.
- It demonstrates end-to-end functional viability:
  - stream startup
  - inference execution
  - result visualization in a frontend
- Known resource consumption limitations exist (RAM, partial NVENC/NVDEC usage).
- These limitations are recognized as technical debt, not functional failures.