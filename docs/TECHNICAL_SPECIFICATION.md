# VOXA - TECHNICAL SPECIFICATION

## Overview

Voxa is a local voice assistant designed for Windows desktop that provides real-time voice interaction with minimal latency. It integrates small local LLMs for quick responses, larger models for complex tasks, vectorizer for persistent memory, and MCP (Model Context Protocol) for extended interactions beyond standard LLM capabilities.

## Core Architecture

### System Components

1. **Voice Interface (C# WPF)**
   - Circular wave visualization that responds to audio input
   - Real-time microphone capture and audio output
   - VAD (Voice Activity Detection) for speech detection
   - Barge-in capability to interrupt AI speech

2. **ASR (Automatic Speech Recognition)**
   - Fast-Whisper local processing
   - Real-time transcription with partial results
   - Portuguese language optimization

3. **LLM Pipeline**
   - **Front LLM (3B)**: Qwen2.5-3B-Instruct for quick responses
   - **Planner LLM (7B)**: For complex reasoning and planning
   - **External API**: DeepSeek Chat for additional support

4. **TTS (Text-to-Speech)**
   - Piper TTS for natural Portuguese speech
   - Streaming audio output
   - Voice interruption handling

5. **Memory & Context**
   - Vectorizer for persistent memory storage
   - Conversation context management
   - Tool integration via MCP

## Technical Stack

### Desktop Application
- **Framework**: .NET 8 + WPF
- **Graphics**: SkiaSharp for wave visualization
- **Audio**: NAudio for I/O operations
- **Configuration**: appsettings.json + environment variables

### AI Models
- **ASR**: Fast-Whisper (Python subprocess) or Whisper.NET (C#)
- **Front LLM**: Qwen2.5-3B-Instruct (GGUF Q4_K_M/Q5_K_M)
- **Planner LLM**: Qwen2.5-7B-Instruct (GGUF Q4_K_M)
- **TTS**: Piper (Portuguese)

### Infrastructure
- **LLM Server**: llama.cpp server with GPU acceleration
- **Memory**: SQLite/Parquet for logs and datasets
- **Vector Store**: Vectorizer for persistent memory
- **Protocol**: MCP for tool integration

## System Requirements

### Hardware
- **GPU**: RTX 4090 (24GB VRAM) or equivalent
- **RAM**: 128GB system memory
- **Storage**: SSD with sufficient space for models
- **Audio**: Default microphone and speakers

### Software
- **OS**: Windows 10/11
- **Runtime**: .NET 8
- **Python**: 3.9+ (for ASR subprocess)
- **CUDA**: Latest version for GPU acceleration

## Data Flow

### Voice Input Pipeline
1. **Audio Capture**: Microphone → PCM 16-bit chunks
2. **VAD**: Voice activity detection
3. **ASR**: Speech-to-text conversion
4. **Intent Classification**: Determine response type
5. **LLM Processing**: Generate response
6. **TTS**: Text-to-speech conversion
7. **Audio Output**: Play response

### Response Types
1. **Direct Response**: Front LLM (3B) provides immediate answer
2. **Tool Call**: Access MCP tools for external actions
3. **Planner Required**: Use larger model for complex reasoning
4. **External API**: DeepSeek Chat for additional support

## Performance Targets

### Latency Goals (NFR)
- **ASR**: < 200ms for partial results
- **Front LLM TTFB**: ≤ 400ms
- **Front LLM Throughput**: ≥ 8-12 tok/s on 3B model
- **TTS**: < 300ms for speech generation, starts by sentence
- **Total Pipeline**: < 1.5s for simple queries

### Reliability (NFR)
- **Tool JSON Validity**: ≥ 98%
- **Barge-in Response**: < 80ms from threshold
- **System Uptime**: ≥ 99.5%

### GPU Usage (NFR)
- **Front 3B**: Always resident in GPU
- **Planner 7B**: On-demand loading
- **KV Cache**: In RAM for multi-sessions

### Throughput
- **Concurrent Users**: 1 (single user interface)
- **Model Loading**: < 30s startup time
- **Memory Usage**: < 32GB RAM, < 20GB VRAM

### Observability (NFR)
- **Structured Logging**: JSON format with timestamps
- **Metrics**: TTFB, tok/s, WER ASR, TTS time, % tool-call
- **Export**: JSONL format for SFT/DPO training data

### Security (NFR)
- **Process Isolation**: Subprocess isolation for external tools
- **Input Sanitization**: All inputs sanitized
- **Resource Limits**: Time/memory limits per tool execution
- **Local Processing**: No data sent to external services

## Model Prompts

### Front Model (System Prompt)
```
Você é um assistente de voz PT-BR, respostas objetivas (≤2 frases).
Quando precisar de dados externos, emita apenas um objeto JSON válido chamando uma ferramenta registrada.
Nunca misture texto com JSON.
Após o resultado da ferramenta, responda em 1–2 frases.
Formate números/datas para TTS natural.
Em interrupção de fala, recomece.
```

### Planner Model (System Prompt)
```
Você é o planejador. Escreva o plano mínimo e retorne somente o conteúdo essencial, sem floreios.
Respostas curtas; quando houver dados extensos, devolva um resumo pronto para ser falado.
```

## Configuration

### Model Configuration
```json
{
  "models": {
    "front_llm": {
      "name": "Qwen2.5-3B-Instruct",
      "format": "GGUF",
      "quantization": "Q4_K_M",
      "server_port": 8081,
      "gpu_layers": -1
    },
    "planner_llm": {
      "name": "Qwen2.5-7B-Instruct",
      "format": "GGUF",
      "quantization": "Q4_K_M",
      "server_port": 8082,
      "gpu_layers": 20
    },
    "asr": {
      "model": "fast-whisper",
      "language": "pt",
      "vad_enabled": true
    },
    "tts": {
      "model": "piper",
      "language": "pt-BR",
      "voice": "default"
    }
  }
}
```

### Audio Configuration
```json
{
  "audio": {
    "input": {
      "device": "default",
      "sample_rate": 16000,
      "channels": 1,
      "format": "PCM16"
    },
    "output": {
      "device": "default",
      "sample_rate": 22050,
      "channels": 1,
      "format": "PCM16"
    },
    "vad": {
      "threshold": 0.5,
      "min_duration": 0.1,
      "max_duration": 10.0
    }
  }
}
```

## Integration Points

### MCP (Model Context Protocol)
- **Tool Registry**: Register external tools
- **JSON Schema**: Define tool interfaces
- **Execution**: Async tool execution
- **Results**: Structured response handling

### Vectorizer Integration
- **Memory Storage**: Persistent conversation memory
- **Semantic Search**: Context retrieval
- **Embeddings**: Text vectorization
- **Indexing**: Real-time memory updates

### External APIs
- **DeepSeek Chat**: Additional reasoning support
- **Web Search**: Real-time information retrieval
- **Document Lookup**: Local document search
- **Code Generation**: Development assistance

## Development Phases

### Phase 1: Core Voice Interface
- Basic WPF application with wave visualization
- Microphone capture and audio output
- Simple ASR integration
- Basic TTS functionality

### Phase 2: LLM Integration
- Front LLM (3B) integration
- Response generation and classification
- Basic conversation flow
- Error handling and recovery

### Phase 3: Advanced Features
- Planner LLM (7B) integration
- MCP tool integration
- Vectorizer memory system
- External API support

### Phase 4: Optimization
- Performance tuning
- Latency reduction
- Memory optimization
- User experience improvements