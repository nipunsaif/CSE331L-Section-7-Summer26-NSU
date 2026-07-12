# Edge-AI Storytelling Device Architecture

## Overview
This document outlines the architecture for an offline, edge-based AI storytelling device. The system utilizes a dual-controller design, offloading heavy compute tasks (LLM, STT, Context Analysis) to a Raspberry Pi 4, while real-time hardware I/O and peripheral management are handled by an STM32 microcontroller. 

## System Pipeline Diagram

```text
                 Voice Input
                       │
             USB / I2S Microphone
                       │
               Raspberry Pi 4
        (Offline Whisper.cpp Speech-to-Text)
                       │
             Intent Recognition
                       │
        TinyML Context Analysis Engine
      ┌──────────┬───────────┬──────────┐
      │Emotion   │Environment│Attention │
      │Detection │Detection  │Detection │
      └──────────┴───────────┴──────────┘
                       │
             Context Fusion Module
                       │
            Story Memory (SQLite)
                       │
          Multi-Agent Story Engine
      ┌──────────┬───────────┬──────────┐
      │Narrator  │Character  │Monster   │
      │Guide     │Judge      │Planner   │
      └──────────┴───────────┴──────────┘
                       │
             Ollama Local LLM
        (TinyLlama / Qwen / Gemma)
                       │
             Story Adaptation Module
                       │
      ┌──────────────┬─────────────┐
      │OLED Display  │Text-to-Speech│
      └──────────────┴─────────────┘
                       │
                 STM32 Controller
                       │
 Button • LEDs • Sensors • OLED • UART
                       │
          Telegram Notification
```

## Pipeline Components Breakdown

### 1. Input Layer
* **Audio Capture:** Voice input is captured via a **USB or I2S Microphone** connected directly to the Raspberry Pi.

### 2. Processing Core (Raspberry Pi 4)
* **Speech-to-Text (STT):** Utilizes **Whisper.cpp** for offline, low-latency audio transcription without cloud dependency.
* **Intent Recognition:** Parses the transcribed text to determine the user's primary goal, action, or command.
* **TinyML Context Analysis Engine:** Evaluates additional modalities:
  * **Emotion Detection:** Assesses the user's vocal tone or facial expressions.
  * **Environment Detection:** Analyzes background noise or lighting.
  * **Attention Detection:** Determines if the user is actively engaging with the device.
* **Context Fusion Module:** Merges the recognized intent with the TinyML context data to create a comprehensive prompt state.
* **Story Memory (SQLite):** Acts as the long-term memory, storing character progression, past interactions, and narrative state to ensure continuity.
* **Multi-Agent Story Engine:** Routes the augmented prompt to specialized system prompts or agents (e.g., Narrator, Character, Monster Planner) based on the current context.
* **Local LLM Engine:** Powered by **Ollama**, running 4-bit quantized lightweight models (like TinyLlama, Qwen 1.5B, or Gemma 2B) to generate the story locally.
* **Story Adaptation Module:** Parses and formats the raw LLM output, separating spoken dialogue for TTS from system status text for the display.

### 3. Hardware & Output Layer (STM32 Controller)
* **Audio & Visuals:** The parsed outputs are sent to the **Text-to-Speech** engine and **OLED Display**.
* **STM32 Controller:** Acts as the real-time hardware manager, communicating with the Pi via **UART**.
* **Peripherals:** Manages physical **Buttons, LEDs, and Sensors**, ensuring the Pi's CPU isn't bogged down by hardware polling.
* **Connectivity:** Capable of dispatching **Telegram Notifications** for remote monitoring or parental alerts.

## Execution Flow
1. **Listen:** Microphone captures audio $\rightarrow$ Pi 4 transcribes it.
2. **Analyze:** Pi evaluates intent and fuses it with environmental/emotional data.
3. **Generate:** The Multi-Agent system fetches memory and queries the local LLM.
4. **Act:** The LLM responds, the output is adapted for audio (TTS) and screen (OLED), and the STM32 triggers physical LED/Sensor feedback.
