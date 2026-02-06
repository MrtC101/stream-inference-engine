# Stream Inference Engine

## Problem to solve

This project implements a real-time video inference system designed to process multiple concurrent RTSP streams under strict input, output, performance and resource constraints. 

## Constraints


### System Input 
- All Input streams are from different remote sources.
- All sources are assumed to stream using RTSP protocol over http/s and are h265 encoded.
- Remote connection is unstable and can be lost at any time.

### System Output
- streams have to be served using RTSP over http and coded in H264 or H256 codec.  
- Inference results are shown as layouts that are drawn over the frame
- No metadata can be generated to be transmitted with the stream.
- Inference should trigger events based on the inference rules declared in a DSL YAML configuration file. 
These events should be consumed externally in the future.
- 

### System Output
- Output should be coded in a H264 format stream served by an RTSP server 
- Achieve real-time output keeping frame drop rate metric under 5% 
- Inference result overlays must be plotted on top of the input video
- No ovelayes medata is generated or transfered as output

### Inference Contraints
- System must be able to process a DSL inference configuration YAML file.
- System must be able to hot-reload inference configurations while changed.
- System must provide a templates and configurations processing capability to ensure reusability. 
- Capability to start and stop inference processing over each different processed stream.

### Resources constraints
- Reduce CPU usage
- Do stream decode and encode using hardware
- Do inference using GPU computation  

## Responsibility boundaries
- System must reconnect autocamitaclly if input stream is lost.
- System must be able to handle multiple concurrent streams.
- System must provide an API to load configuration files, model weights, and run inference streams over input streams.
