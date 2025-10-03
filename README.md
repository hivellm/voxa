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
```bash
# Download Qwen2.5-3B-Instruct
wget https://huggingface.co/Qwen/Qwen2.5-3B-Instruct-GGUF/resolve/main/qwen2.5-3b-instruct-q4_k_m.gguf

# Download Qwen2.5-7B-Instruct
wget https://huggingface.co/Qwen/Qwen2.5-7B-Instruct-GGUF/resolve/main/qwen2.5-7b-instruct-q4_k_m.gguf

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
```json
{
  "models": {
    "front_llm": {
      "name": "Qwen2.5-3B-Instruct",
      "server_url": "http://localhost:8081",
      "quantization": "Q4_K_M"
    },
    "planner_llm": {
      "name": "Qwen2.5-7B-Instruct",
      "server_url": "http://localhost:8082",
      "quantization": "Q4_K_M"
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

### Latency Targets
- **ASR**: < 200ms for partial results
- **Front LLM**: < 500ms for direct responses
- **TTS**: < 300ms for speech generation
- **Total Pipeline**: < 1.5s for simple queries

### Optimization Tips
- Use GPU acceleration for LLM inference
- Enable model quantization (Q4_K_M)
- Configure appropriate buffer sizes
- Monitor system resources

## 🔧 Troubleshooting

### Common Issues

#### Audio Not Working
- Check microphone permissions
- Verify audio device selection
- Test with Windows audio settings

#### LLM Server Not Responding
- Check if llama.cpp server is running
- Verify model file paths
- Check GPU memory usage

#### High Latency
- Reduce model size
- Enable quantization
- Check system resources
- Optimize buffer sizes

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
