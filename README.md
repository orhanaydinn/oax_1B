# OAX-1B Humanoid LLM

OAX-1B Humanoid LLM is an experimental 1B-parameter LLaMA-style language model developed for humanoid robot interaction, structured JSON tool-call generation, and task-level robotic reasoning.

The model was designed as part of a broader humanoid robotics project that combines natural language interaction, active vision, object detection, robot state memory, and controller-based action validation. Unlike a general-purpose chatbot, OAX-1B Humanoid LLM focuses on converting user commands into structured outputs that can be interpreted and validated by a robotic control pipeline.

The merged model is available on Hugging Face:

**Hugging Face Model:**  
https://huggingface.co/orhanaydinn/OAX-1B-Humanoid-Merged

---

## Project Overview

The main goal of this project is to explore how a compact custom language model can be pre-trained and fine-tuned for humanoid robot command understanding.

The model is intended to act as the high-level reasoning layer of a robot system. It receives natural language commands and produces structured JSON-style responses, such as tool calls for searching, picking, placing, handover actions, or asking clarification questions.

The LLM does not directly control motors or low-level hardware. Instead, its outputs are passed to a planner and controller layer, where they are validated, repaired, clarified, or rejected before any physical action is executed.

---

## System Context

OAX-1B Humanoid LLM was developed for a humanoid desktop robot system with active vision and object-based interaction.

The overall system follows this structure:

```text
User command
→ OAX-1B Humanoid LLM
→ JSON tool-call proposal
→ Planner / Controller validation
→ Vision and robot tools
→ Physical robot action
→ Updated robot state and memory
```

This separation is important because the language model is responsible for high-level reasoning, while the controller is responsible for safety, validation, and execution consistency.

---

## Intended Use

This model is intended for experimental research and development in:

- Humanoid robot command interpretation
- JSON tool-call generation
- Robot assistant behaviour
- Active vision and object interaction systems
- Planner/controller-based robotic architectures
- Human-robot interaction research

Example command types include:

- Asking what objects are visible
- Searching for a specific object
- Picking up an object
- Placing an object
- Handing an object to a person
- Checking robot status
- Asking clarification when the command is ambiguous
- Refusing unsafe or invalid requests

---

## Model Architecture

The model follows a LLaMA-style decoder-only Transformer architecture with approximately 1B parameters.

| Component | Configuration |
|---|---|
| Architecture | LLaMA-style decoder-only Transformer |
| Approximate size | 1B parameters |
| Hidden size | 1792 |
| Number of layers | 24 |
| Attention heads | 28 |
| Key-value heads | 4 |
| Intermediate size | 4608 |
| Activation | SiLU |
| Normalisation | RMSNorm |
| Positional encoding | RoPE |
| Context length | 2048 / 4096 experimental settings |
| Training framework | Hugging Face Transformers |

The model was first trained as a base language model and then adapted for humanoid robot interaction using supervised fine-tuning.

---

## Training Pipeline

The training process consisted of two main stages:

1. **Pre-training**  
   The base model was trained on a cleaned general text corpus to learn general language modelling capabilities.

2. **Supervised Fine-Tuning**  
   The model was fine-tuned on robotics-oriented instruction examples, with a focus on structured JSON outputs and tool-call behaviour.

The overall training pipeline can be summarised as:

```text
Raw text data
→ Cleaning and filtering
→ Tokenisation
→ Base model pre-training
→ Checkpoint saving
→ Robot instruction dataset generation
→ Supervised fine-tuning
→ JSON tool-call evaluation
→ Robot controller integration
```

---

## Dataset

The pre-training dataset was created by merging and cleaning multiple open text sources, including general language data, encyclopaedic text, web text, and technical discussion-style content.

The supervised fine-tuning dataset was designed specifically for humanoid robot interaction and includes examples for:

- Visible object queries
- Robot status queries
- Object search commands
- Pick and place instructions
- Handover commands
- Ambiguous command handling
- Clarification responses
- Invalid or unsafe request handling
- Structured JSON tool-call generation

Approximate dataset summary:

| Stage | Approximate Size | Purpose |
|---|---:|---|
| Pre-training | ~860k training rows | General language modelling |
| Validation | ~8.6k validation rows | Pre-training validation |
| Supervised fine-tuning | ~41.5k examples | Robot instruction and tool-call behaviour |

The full dataset is not included in this repository due to size, cleaning requirements, and source-specific licensing considerations.

---

## Example Tool-Call Outputs

### Example 1: Visible Object Query

User command:

```text
What do you see?
```

Model output:

```json
{
  "type": "tool_call",
  "response": "I will check the visible objects.",
  "tool": "get_visible_objects",
  "arguments": {}
}
```

---

### Example 2: Object Search

User command:

```text
Find the bottle.
```

Model output:

```json
{
  "type": "tool_call",
  "response": "Searching for the bottle.",
  "tool": "search_object",
  "arguments": {
    "object": "bottle"
  }
}
```

---

### Example 3: Pick Command

User command:

```text
Pick up the cup.
```

Model output:

```json
{
  "type": "tool_call",
  "response": "I will try to pick up the cup.",
  "tool": "pick_object",
  "arguments": {
    "object": "cup"
  }
}
```

---

### Example 4: Clarification

User command:

```text
Pick it up.
```

Model output:

```json
{
  "type": "clarify",
  "response": "Which object would you like me to pick up?",
  "tool": null,
  "arguments": {}
}
```

---

## Supported Tool Types

The model was fine-tuned to generate structured responses for a limited set of robot tools.

Example tools include:

| Tool | Purpose |
|---|---|
| `get_visible_objects` | Returns objects currently visible to the robot |
| `get_robot_status` | Checks the current robot state |
| `search_object` | Searches for a target object using active vision |
| `pick_object` | Attempts to pick up a target object |
| `place_object` | Places the currently held object |
| `handover_object` | Hands an object to a person |
| `go_home` | Moves the robot to a home or safe position |
| `stop` | Stops the current action |

These tool calls are intended to be validated by an external controller before execution.

---

## Robotic Controller Integration

In the full robot system, the model output is not executed directly.

Instead, the controller checks:

- Whether the JSON structure is valid
- Whether the requested tool exists
- Whether the target object is visible or known in memory
- Whether the robot is already holding an object
- Whether the command requires clarification
- Whether the action is safe and executable
- Whether the proposed action should be repaired, refused, or executed

This design reduces the risk of invalid LLM outputs causing unsafe robot behaviour.

---

## Repository Contents

This repository contains the development notebook and documentation for the OAX-1B Humanoid LLM project.

Current repository contents:

```text
oax_1B/
├── README.md
└── oax_1B_LLM.ipynb
```

Recommended future repository structure:

```text
oax_1B/
├── README.md
├── oax_1B_LLM.ipynb
├── requirements.txt
├── configs/
│   └── model_config.json
├── examples/
│   ├── sample_prompts.jsonl
│   └── sample_outputs.json
└── assets/
    └── system_overview.png
```

The model weights are hosted separately on Hugging Face.

---

## Model Availability

The merged model can be found here:

https://huggingface.co/orhanaydinn/OAX-1B-Humanoid-Merged

Depending on the release version, the Hugging Face repository may include the merged model files, tokenizer files, configuration files, and inference-related metadata.

---

## Limitations

This model is experimental and should not be treated as a production-ready robotics model.

Current limitations include:

- The model may occasionally generate invalid or incomplete JSON.
- It requires an external validation layer before robot execution.
- The model is focused on a limited humanoid robot tool set.
- Performance depends heavily on the quality and structure of the fine-tuning data.
- It is not designed to directly control motors, servos, or low-level robot hardware.
- It is not a general-purpose chatbot and may perform best in robotics-specific command scenarios.
- Real-world robot performance also depends on vision accuracy, object detection, kinematics, and hardware reliability.

---

## Future Work

Planned improvements include:

- Improving JSON format reliability
- Expanding the robotics supervised fine-tuning dataset
- Adding more multi-step task examples
- Improving ambiguity handling and dialogue memory
- Evaluating the model against larger open-source LLMs
- Integrating richer object memory and scene context
- Improving active vision interaction with the LLM
- Exploring Vision-Language-Action style models for smoother humanoid robot behaviour
- Testing more advanced planner and controller repair strategies

---

## Disclaimer

OAX-1B Humanoid LLM is an experimental research model. It should not be used for safety-critical applications without a robust external validation, monitoring, and control system.

All physical robot actions should be checked by deterministic safety layers before execution.

---

## Author

Developed by **Orhan Aydin**  
MSc Artificial Intelligence and Data Science  
Humanoid robotics, active vision, and custom LLM research

---

## Links

- **Hugging Face Model:** https://huggingface.co/orhanaydinn/OAX-1B-Humanoid-Merged
- **GitHub Repository:** This repository
