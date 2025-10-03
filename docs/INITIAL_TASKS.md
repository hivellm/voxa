# VOXA - INITIAL TASKS (DELIVERABLES)

## Overview

This document outlines the initial tasks and deliverables for the Voxa voice assistant project. These tasks represent the core functionality needed for a working prototype.

## Core Components (Priority 1)

### 1. UI WPF with WaveCanvas + Push-to-Talk

**Objective**: Create the main user interface with visual feedback and interaction controls.

**Deliverables**:
- WPF application with circular wave visualization
- SkiaSharp-based WaveCanvas that responds to audio levels
- Push-to-talk button and spacebar hotkey
- Status indicators for system states
- Conversation panel showing recent interactions

**Technical Requirements**:
- .NET 8 WPF application
- SkiaSharp for graphics rendering
- Real-time audio level visualization
- Responsive UI with smooth animations

**Acceptance Criteria**:
- [ ] Wave visualization responds to microphone input
- [ ] Push-to-talk button works correctly
- [ ] Spacebar hotkey functions properly
- [ ] UI remains responsive during audio processing
- [ ] Status indicators show correct system states

### 2. Microphone → ASR (Speech Recognition)

**Objective**: Implement real-time speech recognition with partial and final results.

**Deliverables**:
- Audio capture from default microphone
- Voice Activity Detection (VAD)
- Fast-Whisper integration for ASR
- Partial transcription results
- Final transcription results
- Portuguese language optimization

**Technical Requirements**:
- NAudio for audio capture
- Fast-Whisper Python subprocess or Whisper.NET
- VAD with configurable thresholds
- Real-time transcription streaming
- Error handling and recovery

**Acceptance Criteria**:
- [ ] Microphone captures audio correctly
- [ ] VAD detects voice activity accurately
- [ ] ASR produces partial results in real-time
- [ ] Final transcription is accurate (>90% for clear speech)
- [ ] Portuguese language support works
- [ ] Latency < 200ms for partial results

### 3. LLM Client (llama.cpp HTTP Stream)

**Objective**: Implement HTTP client for llama.cpp server with streaming support.

**Deliverables**:
- HTTP client for Front 3B model
- HTTP client for Planner 7B model
- Streaming token support
- Timeout handling
- Error recovery and retry logic

**Technical Requirements**:
- HttpClient with streaming support
- JSON serialization/deserialization
- Configurable timeouts
- Retry policy implementation
- Connection pooling

**Acceptance Criteria**:
- [ ] Front 3B client connects successfully
- [ ] Planner 7B client connects successfully
- [ ] Streaming tokens work correctly
- [ ] Timeout handling functions properly
- [ ] Error recovery works as expected
- [ ] TTFB ≤ 400ms for Front model

### 4. Tool Registry with Core Tools

**Objective**: Implement tool registry system with essential tools.

**Deliverables**:
- Tool registry interface
- Web search tool
- Document lookup tool
- Assistant deep tool (calls Planner 7B)
- Tool execution engine
- Result processing

**Technical Requirements**:
- MCP (Model Context Protocol) compliance
- JSON Schema validation
- Async tool execution
- Error handling
- Result formatting

**Acceptance Criteria**:
- [ ] Tool registry registers tools correctly
- [ ] Web search tool returns relevant results
- [ ] Document lookup tool searches local files
- [ ] Assistant deep tool calls Planner 7B
- [ ] Tool execution is async and non-blocking
- [ ] JSON validation works correctly

### 5. Piper TTS + AudioPlayer + Barge-in

**Objective**: Implement text-to-speech with streaming audio output and interruption capability.

**Deliverables**:
- Piper TTS integration
- Streaming audio output
- AudioPlayer for playback
- Barge-in functionality
- Portuguese voice model
- Audio format conversion

**Technical Requirements**:
- Piper TTS subprocess
- NAudio for audio playback
- Streaming audio processing
- VAD for barge-in detection
- Audio format handling
- Error recovery

**Acceptance Criteria**:
- [ ] Piper TTS synthesizes Portuguese speech
- [ ] Streaming audio output works
- [ ] AudioPlayer plays audio correctly
- [ ] Barge-in interrupts TTS within 80ms
- [ ] Audio quality is natural and clear
- [ ] Latency < 300ms for TTS generation

### 6. Structured Logging + JSONL Export

**Objective**: Implement comprehensive logging and data export for fine-tuning.

**Deliverables**:
- Structured logging system
- JSONL export for SFT data
- JSONL export for DPO data
- JSONL export for tool call data
- Log rotation and management
- Performance metrics collection

**Technical Requirements**:
- Structured logging (JSON format)
- File-based log storage
- JSONL export functionality
- Log rotation policies
- Metrics collection
- Error logging

**Acceptance Criteria**:
- [ ] All system events are logged
- [ ] JSONL exports are generated correctly
- [ ] Log rotation works properly
- [ ] Performance metrics are collected
- [ ] Error logging is comprehensive
- [ ] Export format is compatible with training tools

## Scripts and Automation (Priority 2)

### 7. Model Server Scripts

**Objective**: Create automation scripts for model server management.

**Deliverables**:
- run_front.bat script for 3B model
- run_planner.bat script for 7B model
- VRAM optimization settings
- KV cache configuration
- Model loading automation
- Health check scripts

**Technical Requirements**:
- Batch script automation
- llama.cpp server configuration
- GPU memory management
- Model path configuration
- Service health monitoring

**Acceptance Criteria**:
- [ ] Scripts start model servers correctly
- [ ] VRAM usage is optimized
- [ ] KV cache is configured properly
- [ ] Models load within 30 seconds
- [ ] Health checks work correctly
- [ ] Scripts handle errors gracefully

### 8. Smoke Tests

**Objective**: Implement automated smoke tests for core functionality.

**Deliverables**:
- JSON validation tests
- Barge-in functionality tests
- TTFB performance tests
- Token/s throughput tests
- End-to-end conversation tests
- Error handling tests

**Technical Requirements**:
- Unit test framework
- Integration test setup
- Performance test tools
- Mock data generation
- Automated test execution
- Test reporting

**Acceptance Criteria**:
- [ ] JSON validation tests pass
- [ ] Barge-in tests work correctly
- [ ] TTFB tests meet requirements
- [ ] Throughput tests meet requirements
- [ ] End-to-end tests pass
- [ ] Error handling tests work

## Development Phases

### Phase 1: Core Infrastructure (Weeks 1-2)
- [ ] Project setup and structure
- [ ] Basic WPF application
- [ ] Audio capture implementation
- [ ] Message bus implementation
- [ ] Configuration system

### Phase 2: ASR and LLM Integration (Weeks 3-4)
- [ ] Fast-Whisper integration
- [ ] LLM client implementation
- [ ] Basic conversation flow
- [ ] Error handling
- [ ] Performance optimization

### Phase 3: Tools and TTS (Weeks 5-6)
- [ ] Tool registry implementation
- [ ] Core tools development
- [ ] Piper TTS integration
- [ ] Barge-in functionality
- [ ] Audio playback system

### Phase 4: Polish and Testing (Weeks 7-8)
- [ ] UI improvements
- [ ] Performance tuning
- [ ] Comprehensive testing
- [ ] Documentation completion
- [ ] Deployment preparation

## Success Metrics

### Performance Targets
- **ASR Latency**: < 200ms for partial results
- **Front LLM TTFB**: ≤ 400ms
- **Front LLM Throughput**: ≥ 8-12 tok/s
- **TTS Latency**: < 300ms
- **Barge-in Response**: < 80ms
- **Total Pipeline**: < 1.5s for simple queries

### Quality Targets
- **ASR Accuracy**: > 90% for clear speech
- **Tool JSON Validity**: ≥ 98%
- **System Uptime**: ≥ 99.5%
- **Error Recovery**: < 5s for recoverable errors

### User Experience Targets
- **UI Responsiveness**: < 100ms for UI updates
- **Audio Quality**: Natural and clear speech
- **Interaction Flow**: Smooth and intuitive
- **Error Messages**: Clear and helpful

## Risk Mitigation

### Technical Risks
- **Model Performance**: Test with multiple model sizes
- **Audio Quality**: Implement noise reduction
- **Latency Issues**: Optimize pipeline performance
- **Memory Usage**: Monitor and optimize resource usage

### Development Risks
- **Scope Creep**: Focus on core functionality first
- **Integration Issues**: Implement comprehensive testing
- **Performance Problems**: Regular performance monitoring
- **User Experience**: Early user feedback and iteration

## Dependencies

### External Dependencies
- .NET 8 SDK
- Python 3.9+ (for ASR)
- CUDA Toolkit (for GPU acceleration)
- llama.cpp (for LLM serving)
- Fast-Whisper (for ASR)
- Piper TTS (for speech synthesis)

### Internal Dependencies
- Message bus implementation
- Configuration system
- Logging framework
- Error handling system
- Performance monitoring

## Conclusion

These initial tasks provide a solid foundation for the Voxa voice assistant. By focusing on core functionality first and implementing comprehensive testing, we can ensure a robust and performant system that meets the specified requirements.
