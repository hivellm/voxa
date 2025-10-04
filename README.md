# Voxa - Local Voice Assistant

<div align="center">

![Voxa Logo](https://img.shields.io/badge/Voxa-Voice%20Assistant-blue?style=for-the-badge&logo=speech&logoColor=white)

**A sophisticated local voice assistant for Windows desktop with real-time interaction capabilities**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![.NET 8](https://img.shields.io/badge/.NET-8-purple.svg)](https://dotnet.microsoft.com/download/dotnet/8.0)
[![Python 3.9+](https://img.shields.io/badge/Python-3.9+-green.svg)](https://python.org)
[![CUDA](https://img.shields.io/badge/CUDA-Required-red.svg)](https://developer.nvidia.com/cuda-toolkit)

</div>

## 🎯 Overview

Voxa is a sophisticated local voice assistant designed for Windows desktop that provides real-time voice interaction with minimal latency. It integrates small local LLMs for quick responses, larger models for complex tasks, vectorizer for persistent memory, and MCP (Model Context Protocol) for extended interactions beyond standard LLM capabilities.

## ✨ Key Features

- 🎤 **Real-time Voice Interaction**: Low-latency voice processing with VAD and barge-in capabilities
- 🔒 **Local Processing**: All voice data processed locally for privacy
- 🧠 **Multi-Model Architecture**: 3B model for quick responses, 7B model for complex reasoning
- 💾 **Persistent Memory**: Vectorizer integration for conversation memory and context
- 🔧 **Tool Integration**: MCP protocol for external tool and service integration
- 🎨 **Beautiful UI**: Circular wave visualization that responds to audio input
- 🇧🇷 **Portuguese Support**: Optimized for Portuguese language (pt-BR)

## 🏗️ Architecture

Voxa follows a modular, event-driven architecture with the following components:

- **Voice Interface**: WPF application with SkiaSharp wave visualization
- **Audio Engine**: NAudio-based audio capture and playback
- **ASR**: Fast-Whisper for speech recognition
- **LLM Pipeline**: Multi-model language processing
- **TTS**: Piper for text-to-speech synthesis
- **Memory Store**: Vectorizer for persistent memory
- **Tool Registry**: MCP-based tool integration

## 📋 System Requirements

### Hardware Detection & Fallback Strategy

Voxa implements intelligent hardware detection to ensure optimal performance across different system configurations:

#### **GPU Detection & Requirements**
- **Recommended**: RTX 4090 (24GB VRAM) or equivalent high-end GPU
- **Minimum**: RTX 3060 (8GB VRAM) or equivalent mid-range GPU
- **Fallback**: CPU-only mode with BitNet quantization for systems without adequate GPU

#### **Hardware Verification Process**
1. **GPU Detection**: Automatically detects available GPU hardware and VRAM
2. **Performance Assessment**: Evaluates GPU capabilities for LLM inference
3. **Model Selection**: Chooses appropriate model size based on available resources
4. **Fallback Activation**: Switches to BitNet CPU mode if GPU is insufficient

#### **BitNet CPU Fallback**
When GPU hardware is inadequate or unavailable:
- **Model**: BitNet-1.58B quantized for CPU inference
- **Quantization**: INT8 quantization for memory efficiency
- **Performance**: Optimized for CPU-only execution
- **Memory**: Requires only 2-4GB RAM instead of 24GB+ VRAM

### Hardware Requirements

#### **High-End Configuration (GPU)**
- **GPU**: RTX 4090 (24GB VRAM) or equivalent
- **RAM**: 128GB system memory
- **Storage**: SSD with sufficient space for models
- **Audio**: Default microphone and speakers

#### **Mid-Range Configuration (GPU)**
- **GPU**: RTX 3060 (8GB VRAM) or equivalent
- **RAM**: 32GB system memory
- **Storage**: SSD with sufficient space for models
- **Audio**: Default microphone and speakers

#### **Low-End Configuration (CPU Fallback)**
- **GPU**: Not required (BitNet CPU mode)
- **RAM**: 8GB system memory minimum
- **Storage**: SSD with sufficient space for BitNet models
- **Audio**: Default microphone and speakers

### Software
- **OS**: Windows 10/11
- **Runtime**: .NET 8
- **Python**: 3.9+ (for ASR subprocess)
- **CUDA**: Latest version for GPU acceleration (optional for BitNet fallback)

## 🚀 Quick Start

### 1. Clone the Repository
```bash
git clone https://github.com/hivellm/voxa.git
cd voxa
```

### 2. Install Dependencies
```bash
# Install .NET 8 SDK
# Install Python 3.9+
# Install CUDA Toolkit
```

### 3. Download Models

#### **GPU Configuration (Recommended)**
```bash
# Download Qwen2.5-3B-Instruct (Front LLM)
wget https://huggingface.co/Qwen/Qwen2.5-3B-Instruct-GGUF/resolve/main/qwen2.5-3b-instruct-q4_k_m.gguf

# Download Qwen2.5-7B-Instruct (Planner LLM)
wget https://huggingface.co/Qwen/Qwen2.5-7B-Instruct-GGUF/resolve/main/qwen2.5-7b-instruct-q4_k_m.gguf
```

#### **CPU Fallback Configuration (BitNet)**
```bash
# Download BitNet-1.58B for CPU inference
wget https://huggingface.co/microsoft/BitNet-1.58B-Instruct-GGUF/resolve/main/bitnet-1.58b-instruct-q4_k_m.gguf
```

#### **TTS Model (Required for both configurations)**
```bash
# Download Piper TTS model
wget https://huggingface.co/rhasspy/piper-voices/resolve/v1.0.0/pt/pt_BR/faber/medium/piper_pt_BR_faber_medium.onnx
```

### 4. Start Services
```bash
# Start LLM servers
./start_llm_servers.sh

# Start Vectorizer
./start_vectorizer.sh

# Start Voxa
dotnet run --project src/Voxa.UI
```

## ⚙️ Configuration

### Model Configuration

#### **GPU Configuration (High Performance)**
```json
{
  "models": {
    "front_llm": {
      "name": "Qwen2.5-3B-Instruct",
      "server_url": "http://localhost:8081",
      "quantization": "Q4_K_M",
      "gpu_layers": 35,
      "context_length": 4096
    },
    "planner_llm": {
      "name": "Qwen2.5-7B-Instruct", 
      "server_url": "http://localhost:8082",
      "quantization": "Q4_K_M",
      "gpu_layers": 35,
      "context_length": 4096
    }
  }
}
```

#### **CPU Fallback Configuration (BitNet)**
```json
{
  "models": {
    "front_llm": {
      "name": "BitNet-1.58B-Instruct",
      "server_url": "http://localhost:8081", 
      "quantization": "Q4_K_M",
      "gpu_layers": 0,
      "context_length": 2048,
      "cpu_threads": 8
    },
    "planner_llm": {
      "name": "BitNet-1.58B-Instruct",
      "server_url": "http://localhost:8082",
      "quantization": "Q4_K_M", 
      "gpu_layers": 0,
      "context_length": 2048,
      "cpu_threads": 8
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
      "channels": 1
    },
    "output": {
      "device": "default",
      "sample_rate": 22050,
      "channels": 1
    },
    "vad": {
      "threshold": 0.5,
      "min_duration": 0.1,
      "max_duration": 10.0
    }
  }
}
```

## 🎮 Usage

### Basic Voice Interaction
1. Launch Voxa
2. Speak into your microphone
3. Voxa will transcribe, process, and respond
4. Use barge-in to interrupt Voxa mid-speech

### Hotkeys
- **Spacebar**: Push-to-talk mode
- **Ctrl+Space**: Toggle listening mode
- **Esc**: Stop current operation

### Voice Commands
- **"Pesquisar [query]"**: Web search
- **"Lembrar [text]"**: Store in memory
- **"O que você sabe sobre [topic]"**: Search memory
- **"Criar código [description]"**: Code generation
- **"Explicar [concept]"**: Detailed explanation

## 🛠️ Development

### Project Structure
```
voxa/
├── src/
│   ├── Voxa.Core/           # Core business logic
│   ├── Voxa.UI/             # WPF user interface
│   ├── Voxa.Audio/          # Audio processing
│   ├── Voxa.ASR/            # Speech recognition
│   ├── Voxa.LLM/            # Language model client
│   ├── Voxa.TTS/            # Text-to-speech
│   ├── Voxa.Memory/         # Memory management
│   └── Voxa.Tools/          # Tool integration
├── tests/
├── models/                    # AI model files
├── config/                    # Configuration files
└── docs/                      # Documentation
```

### Building from Source
```bash
# Clone repository
git clone https://github.com/hivellm/voxa.git
cd voxa

# Restore dependencies
dotnet restore

# Build solution
dotnet build

# Run tests
dotnet test

# Run application
dotnet run --project src/Voxa.UI
```

### Adding Custom Tools
```csharp
public class CustomTool : ITool
{
    public string Name => "custom_tool";
    public string Description => "Custom tool description";
    
    public async Task<ToolResult> ExecuteAsync(Dictionary<string, object> parameters)
    {
        // Tool implementation
        return new ToolResult
        {
            Success = true,
            Result = "Tool executed successfully"
        };
    }
}
```

## 📊 Performance

### Hardware-Based Performance Profiles

#### **High-End GPU (RTX 4090)**
- **ASR**: < 150ms for partial results
- **Front LLM**: < 300ms for direct responses  
- **Planner LLM**: < 800ms for complex reasoning
- **TTS**: < 200ms for speech generation
- **Total Pipeline**: < 1.2s for simple queries, < 2.5s for complex tasks

#### **Mid-Range GPU (RTX 3060)**
- **ASR**: < 200ms for partial results
- **Front LLM**: < 500ms for direct responses
- **Planner LLM**: < 1.2s for complex reasoning  
- **TTS**: < 300ms for speech generation
- **Total Pipeline**: < 1.8s for simple queries, < 3.5s for complex tasks

#### **CPU Fallback (BitNet)**
- **ASR**: < 200ms for partial results
- **Front LLM**: < 1.5s for direct responses
- **Planner LLM**: < 3.0s for complex reasoning
- **TTS**: < 300ms for speech generation  
- **Total Pipeline**: < 2.5s for simple queries, < 5.0s for complex tasks

### Optimization Tips
- **GPU Mode**: Use GPU acceleration for LLM inference when available
- **CPU Mode**: Enable BitNet quantization for memory efficiency
- **Model Selection**: Choose appropriate model size based on hardware
- **Buffer Configuration**: Optimize buffer sizes for your system
- **Resource Monitoring**: Monitor GPU/CPU usage and adjust accordingly

## 🔧 Troubleshooting

### Common Issues

#### Audio Not Working
- Check microphone permissions
- Verify audio device selection
- Test with Windows audio settings

#### LLM Server Not Responding
- **GPU Mode**: Check if llama.cpp server is running with GPU acceleration
- **CPU Mode**: Verify BitNet model is loaded and CPU threads are configured
- **Model Files**: Verify model file paths and quantization compatibility
- **Memory Usage**: Check GPU VRAM or system RAM usage
- **Hardware Detection**: Ensure hardware detection completed successfully

#### High Latency
- **GPU Mode**: Reduce model size or enable more aggressive quantization
- **CPU Mode**: Switch to BitNet for better CPU performance
- **System Resources**: Check GPU/CPU usage and thermal throttling
- **Buffer Sizes**: Optimize audio and processing buffer sizes
- **Model Selection**: Choose appropriate model size for your hardware

### Logs
Logs are stored in:
- `logs/voxa.log` - Main application logs
- `logs/audio.log` - Audio processing logs
- `logs/llm.log` - LLM interaction logs

## 📚 Documentation

Comprehensive documentation is available in the `/docs` directory:

- [Technical Specification](docs/TECHNICAL_SPECIFICATION.md) - Detailed technical requirements
- [Implementation Guide](docs/IMPLEMENTATION_GUIDE.md) - Step-by-step implementation
- [System Architecture](docs/SYSTEM_ARCHITECTURE.md) - Architecture overview
- [API Specification](docs/API_SPECIFICATION.md) - API documentation
- [Roadmap](docs/ROADMAP.md) - Development roadmap
- [Initial Tasks](docs/INITIAL_TASKS.md) - Core deliverables

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests
5. Submit a pull request

### Development Guidelines
- Follow C# coding conventions
- Write unit tests for new features
- Update documentation
- Ensure performance requirements are met

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- [llama.cpp](https://github.com/ggerganov/llama.cpp) - LLM inference
- [Fast-Whisper](https://github.com/SYSTRAN/faster-whisper) - Speech recognition
- [Piper](https://github.com/rhasspy/piper) - Text-to-speech
- [NAudio](https://github.com/naudio/NAudio) - Audio processing
- [SkiaSharp](https://github.com/mono/SkiaSharp) - Graphics rendering

## 📞 Support

For support and questions:
- Create an issue on GitHub
- Join our Discord community
- Check the documentation in `/docs`

## 🗺️ Roadmap

### Phase 1: Core Functionality ✅
- Basic voice interface
- LLM integration
- Audio processing

### Phase 2: Advanced Features ✅
- Memory system
- Tool integration
- Performance optimization

### Phase 3: Future Enhancements
- Multi-language support
- Custom voice training
- Cloud integration
- Advanced tool ecosystem

---

<div align="center">

**Made with ❤️ by the HiveLLM Team**

[![GitHub](https://img.shields.io/badge/GitHub-hivellm-black?style=flat&logo=github)](https://github.com/hivellm)
[![Discord](https://img.shields.io/badge/Discord-Join-blue?style=flat&logo=discord)](https://discord.gg/hivellm)

</div>
