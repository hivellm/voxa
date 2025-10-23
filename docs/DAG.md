# VOXA - DAG (Directed Acyclic Graph)

## System Flow DAG

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Microphone    │    │   WaveCanvas    │    │   StatusUI      │
│   (Audio Input) │    │   (Visual)      │    │   (Feedback)    │
└─────────┬───────┘    └─────────────────┘    └─────────────────┘
          │
          ▼
┌─────────────────┐
│   AudioCapture  │
│   (NAudio)      │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐    ┌─────────────────┐
│   VADProcessor  │    │   AudioBuffer   │
│   (Voice Detection) │    │   (Circular)    │
└─────────┬───────┘    └─────────────────┘
          │
          ▼
┌─────────────────┐
│   ASRService    │
│   (Fast-Whisper)│
└─────────┬───────┘
          │
          ▼
┌─────────────────┐    ┌─────────────────┐
│   AsrPartial    │    │   AsrFinal      │
│   (Real-time)   │    │   (Complete)    │
└─────────┬───────┘    └─────────┬───────┘
          │                      │
          ▼                      ▼
┌─────────────────┐    ┌─────────────────┐
│   UI Update     │    │   MessageBus    │
│   (Partial Text)│    │   (Routing)     │
└─────────────────┘    └─────────┬───────┘
                                 │
                                 ▼
┌─────────────────┐
│   Router        │
│   (Intent Class)│
└─────────┬───────┘
          │
          ▼
    ┌─────────────┐
    │   Decision  │
    │   Point     │
    └─────┬───────┘
          │
    ┌─────┴─────┐
    │           │
    ▼           ▼
┌─────────┐ ┌─────────┐ ┌─────────┐
│ Direct  │ │ Tool    │ │ Planner │
│Response │ │ Call    │ │ Required│
└────┬────┘ └────┬────┘ └────┬────┘
     │           │            │
     ▼           ▼            ▼
┌─────────┐ ┌─────────┐ ┌─────────┐
│ LLMFront│ │ToolExec │ │LLMPlanner│
│ (3B)    │ │ (MCP)   │ │ (7B)    │
└────┬────┘ └────┬────┘ └────┬────┘
     │           │            │
     ▼           ▼            ▼
┌─────────┐ ┌─────────┐ ┌─────────┐
│LlmDelta │ │ToolResult│ │LlmDelta │
│(Tokens) │ │ (Data)  │ │(Tokens) │
└────┬────┘ └────┬────┘ └────┬────┘
     │           │            │
     ▼           ▼            ▼
┌─────────────────────────────┐
│       MessageBus            │
│       (Aggregation)         │
└─────────────┬───────────────┘
              │
              ▼
┌─────────────────┐
│   TTSService    │
│   (Piper)       │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐    ┌─────────────────┐
│   TtsStart      │    │   TtsStop       │
│   (Begin)       │    │   (Complete)    │
└─────────┬───────┘    └─────────────────┘
          │
          ▼
┌─────────────────┐
│   AudioPlayer   │
│   (NAudio)      │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   Speakers      │
│   (Audio Output)│
└─────────────────┘
```

## Barge-in Flow DAG

```
┌─────────────────┐
│   Microphone    │
│   (User Speech) │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   VADProcessor  │
│   (Voice Detect)│
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   VoiceActivity │
│   (Active=true) │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   MessageBus    │
│   (Interrupt)   │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   TTSService    │
│   (Stop TTS)    │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   TtsStop       │
│   (reason:      │
│   "bargein")    │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   ASRService    │
│   (Restart)     │
└─────────────────┘
```

## Tool Execution DAG

```
┌─────────────────┐
│   Router        │
│   (Tool Needed) │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   ToolCall      │
│   (JSON)        │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   MessageBus    │
│   (Route)       │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   ToolRegistry  │
│   (Validate)    │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   ToolExecutor  │
│   (Execute)     │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   ToolResult    │
│   (Data)        │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   MessageBus    │
│   (Return)      │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   Router        │
│   (Summarize)   │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   LLMFront      │
│   (Response)    │
└─────────────────┘
```

## Memory Operations DAG

```
┌─────────────────┐
│   LLMClient     │
│   (Need Context)│
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   MemoryRetrieve│
│   (Query)       │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   MessageBus    │
│   (Route)       │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   VectorizerClient│
│   (Search)      │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   MemoryResult  │
│   (Context)     │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   MessageBus    │
│   (Return)      │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   LLMClient     │
│   (Enhanced)    │
└─────────────────┘
```

## Error Handling DAG

```
┌─────────────────┐
│   Any Component │
│   (Error)       │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   ComponentError│
│   (Event)       │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   MessageBus    │
│   (Route)       │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   ErrorHandler  │
│   (Process)     │
└─────────┬───────┘
          │
          ▼
    ┌─────────────┐
    │   Severity  │
    │   Check     │
    └─────┬───────┘
          │
    ┌─────┴─────┐
    │           │
    ▼           ▼
┌─────────┐ ┌─────────┐
│ Warning │ │ Critical│
│ (Log)   │ │ (Alert) │
└────┬────┘ └────┬────┘
     │           │
     ▼           ▼
┌─────────┐ ┌─────────┐
│ Continue│ │ Shutdown│
│ (Normal)│ │ (Emergency)│
└─────────┘ └─────────┘
```

## Performance Monitoring DAG

```
┌─────────────────┐
│   All Components│
│   (Metrics)     │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   MetricsCollector│
│   (Collect)     │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   MessageBus    │
│   (Aggregate)   │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   PerformanceMonitor│
│   (Analyze)     │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   HealthCheck   │
│   (Validate)    │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│   AlertSystem   │
│   (Notify)      │
└─────────────────┘
```

## Key DAG Properties

### 1. Acyclic Structure
- No circular dependencies between components
- Clear flow from input to output
- Predictable execution order

### 2. Parallel Processing
- ASR partial and final results can be processed simultaneously
- Tool execution and LLM processing can run in parallel
- Audio capture and playback are independent

### 3. Event-Driven
- Components react to messages on the bus
- Asynchronous processing throughout
- Non-blocking operations

### 4. Fault Tolerance
- Error handling at each node
- Graceful degradation on failures
- Recovery mechanisms in place

### 5. Scalability
- Components can be scaled independently
- Message bus handles load distribution
- Resource allocation is optimized

## DAG Execution Flow

### Phase 1: Input Processing
1. Audio capture from microphone
2. VAD processing for voice detection
3. ASR for speech-to-text conversion

### Phase 2: Intent Processing
1. Message routing through bus
2. Intent classification by router
3. Decision point for response type

### Phase 3: Response Generation
1. Direct response from Front LLM
2. Tool execution for external data
3. Planner LLM for complex reasoning

### Phase 4: Output Processing
1. TTS for speech synthesis
2. Audio playback through speakers
3. UI updates for user feedback

### Phase 5: Monitoring
1. Performance metrics collection
2. Error handling and recovery
3. System health monitoring

## DAG Optimization

### Latency Optimization
- Parallel processing where possible
- Streaming for real-time response
- Caching for frequent operations

### Resource Optimization
- GPU utilization for LLM inference
- Memory management for audio buffers
- CPU optimization for concurrent tasks

### Reliability Optimization
- Error recovery mechanisms
- Redundant processing paths
- Health monitoring and alerts

This DAG represents the complete flow of the Voxa voice assistant system, from audio input to speech output, with all the necessary components and their interactions clearly defined.
