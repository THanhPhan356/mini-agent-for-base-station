# Mini Agent for Base Station

## Problem

AI assistants such as ChatGPT and Claude demonstrate the capabilities of modern AI. However, using them through cloud services makes users dependent on the provider's infrastructure, policies, and service availability. This project explores deploying and operating an agent on hardware under the user's control.

Using an AI service and running a model locally are different deployment choices; AI models are not inherently limited to the cloud. This project's specific goal is to build a specialized model small enough to support selected control decisions at a base station (BS), rather than reproduce a general-purpose AI assistant.

## Research Question

**Can an agent using a small neural network run on a microcontroller such as an ESP32, connect to a BS, and support control decisions with acceptable quality and latency?**

This question requires experimental validation. A small network alone does not establish feasibility: runtime memory, input processing, communication with the BS, and decision deadlines must also be considered.

## Proposed Approach

Design a local agent around the following control loop:

```text
BS reports state → Agent runs inference → BS validates and applies the decision
       ↑                                                    |
       └──────────────── performance feedback ───────────────┘
```

- **Inputs:** compact channel features, SNR/CQI, traffic load, queue state, and user equipment (UE) quality-of-service requirements, depending on the task.
- **Model:** evaluate a small neural network, such as an MLP, trained on a computer and deployed for inference on the embedded device. Evaluate INT8 quantization to reduce memory and computational requirements.
- **Outputs:** a decision within the BS's supported action space, such as a beam index or a proposed resource allocation.
- **Feedback:** evaluate the policy using performance after applying its decisions; online training on the ESP32 is not assumed.

The agent supports control decisions. The BS continues to handle radio signal processing and checks constraints before applying a proposal. If inputs are stale, a decision is invalid, or the agent misses its deadline, the BS uses a fallback algorithm.

## Target Tasks

| Task | Decision the agent could support | Evaluation metrics |
|---|---|---|
| Beamforming | Select a beam from a codebook; the BS applies the corresponding antenna weights | SNR/SINR, beam measurement overhead, decision latency |
| Resource allocation | Propose how to divide time/frequency resources among UEs within the available budget | Aggregate throughput, fairness, QoS satisfaction |

A UE's data rate is an outcome to evaluate, not a value that can be set arbitrarily. The optimization objective must be explicit: increase aggregate throughput, improve rates for underserved UEs, or meet service requirements.

The planned first experiment is **beam selection in a Python simulation**. Resource allocation and embedded deployment follow validation of the initial task.

## Feasibility Evaluation

| Question | Planned evaluation |
|---|---|
| Is the policy useful? | Compare against beam sweeping and simple beam selection algorithms using the same channels, input information, and measurement budget |
| Does the model fit on the device? | Measure weight size, flash usage, and peak RAM usage, including buffers and communication software |
| Are decisions timely? | Measure latency from state acquisition to decision application at the BS and compare it with the task deadline |
| What is the effect of quantization? | Compare decision quality before and after quantization |
| Does the policy generalize beyond training conditions? | Test on trajectories, channels, and loads not used for training |

The ESP32 is a reference hardware target; the chip variant and network architecture have not been selected. The original ESP32 has 520 KB of on-chip SRAM shared by data and instructions, so this entire capacity cannot be treated as memory available to the model. [Espressif datasheet](https://documentation.espressif.com/esp32_datasheet_en.html).

ESP-DL supports the ESP32, but Espressif notes that execution is slower than on the ESP32-S3 and ESP32-P4. Measurements on the selected device are therefore required before drawing conclusions about its capabilities. [ESP-DL getting started guide](https://docs.espressif.com/projects/esp-dl/en/latest/getting_started/readme.html).

## Status

The project is currently defining the problem and approach. No model, simulation code, or experimental results are available yet. Hardware integration will require a simulated BS or a testbed exposing a suitable control interface; commercial BS integration and real-time operation have not been established.
