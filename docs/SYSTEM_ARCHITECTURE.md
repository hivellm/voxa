# VOXA - SYSTEM ARCHITECTURE

## Architecture Overview

Voxa follows a modular, event-driven architecture designed for real-time voice interaction with minimal latency. The system is built around a central orchestrator that coordinates between audio processing, AI models, and external services.

## High-Level Architecture

\\\
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Voice UI      │    │   Audio Engine  │    │   AI Pipeline   │
│   (WPF/Skia)    │◄──►│   (NAudio)      │◄──►│   (LLM/TTS)     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Orchestrator  │    │   Memory Store  │    │   Tool Registry │
│   (Core Logic)  │◄──►│   (Vectorizer)  │◄──►│   (MCP)         │
└─────────────────┘    └─────────────────┘    └─────────────────┘
\\\

## Core Components

### 1. Voice Interface Layer
**Responsibility**: User interaction and visual feedback

**Components**:
- \WaveCanvas\: SkiaSharp-based circular wave visualization
- \MainWindow\: WPF main interface with controls
- \StatusIndicator\: Shows current system state
- \ConversationPanel\: Displays recent interactions

### 2. Audio Processing Engine
**Responsibility**: Audio I/O and preprocessing

**Components**:
- \AudioCapture\: Microphone input handling
- \AudioPlayer\: Speaker output management
- \VADProcessor\: Voice activity detection
- \AudioBuffer\: Circular buffer for audio chunks

### 3. ASR (Automatic Speech Recognition)
**Responsibility**: Speech-to-text conversion

**Components**:
- \ASRService\: Main ASR orchestrator
- \WhisperProcessor\: Fast-Whisper integration
- \TranscriptionBuffer\: Partial result management
- \LanguageProcessor\: Portuguese language optimization

### 4. AI Pipeline
**Responsibility**: Language understanding and generation

**Components**:
- \LLMClient\: HTTP client for model servers
- \ResponseClassifier\: Intent classification
- \StreamProcessor\: Token streaming handling
- \ContextManager\: Conversation context management

### 5. TTS (Text-to-Speech)
**Responsibility**: Speech synthesis

**Components**:
- \TTSService\: Main TTS orchestrator
- \PiperProcessor\: Piper TTS integration
- \AudioStreamer\: Streaming audio output
- \VoiceManager\: Voice selection and configuration

### 6. Memory and Context
**Responsibility**: Persistent memory and context management

**Components**:
- \VectorizerClient\: Vector store integration
- \MemoryManager\: Conversation memory
- \ContextRetriever\: Semantic search
- \EmbeddingService\: Text vectorization

### 7. Tool Integration
**Responsibility**: External tool and service integration

**Components**:
- \MCPClient\: Model Context Protocol client
- \ToolRegistry\: Tool registration and management
- \ExecutionEngine\: Async tool execution
- \ResultProcessor\: Tool result handling

## Internal Message Bus

### Message Types
\`\`\`typescript
type AsrPartial = { type:"asr.partial", text:string, ts:number }
type AsrFinal   = { type:"asr.final", text:string, ts:number }
type LlmDelta   = { type:"llm.delta", model:"front"|"planner", token:string, ts:number }
type ToolCall   = { type:"tool.call", name:string, args:any, ts:number }
type ToolResult = { type:"tool.result", name:string, result:any, ts:number }
type TtsStart   = { type:"tts.start", text:string, ts:number }
type TtsStop    = { type:"tts.stop", reason:"done"|"bargein"|"error", ts:number }
\`\`\`

### Message Flow
\`\`\`
Audio Input → ASR → Bus → Router → LLM/Tools → Bus → TTS → Audio Output
\`\`\`

## Front Response Rules

### Response Logic
1. **Simple Query**: Direct response (≤2 sentences)
2. **External Data Required**: Emit only valid JSON tool call; await tool.result; then summarize in 1-2 sentences
3. **Complex Reasoning**: Emit assistant_deep; with response, summarize in 1-2 sentences

### Tool Call Protocol
- Front must emit ONLY valid JSON object for tool calls
- Never mix text with JSON
- After tool result, respond in 1-2 sentences
- Format numbers/dates for natural TTS
