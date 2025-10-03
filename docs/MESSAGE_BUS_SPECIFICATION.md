# VOXA - MESSAGE BUS SPECIFICATION

## Overview

The Voxa system uses an internal message bus for communication between components. This document defines the message types, protocols, and flow patterns used throughout the system.

## Message Types

### Core Message Types

```typescript
// ASR Messages
type AsrPartial = { 
  type: "asr.partial", 
  text: string, 
  ts: number 
}

type AsrFinal = { 
  type: "asr.final", 
  text: string, 
  ts: number 
}

// LLM Messages
type LlmDelta = { 
  type: "llm.delta", 
  model: "front" | "planner", 
  token: string, 
  ts: number 
}

// Tool Messages
type ToolCall = { 
  type: "tool.call", 
  name: string, 
  args: any, 
  ts: number 
}

type ToolResult = { 
  type: "tool.result", 
  name: string, 
  result: any, 
  ts: number 
}

// TTS Messages
type TtsStart = { 
  type: "tts.start", 
  text: string, 
  ts: number 
}

type TtsStop = { 
  type: "tts.stop", 
  reason: "done" | "bargein" | "error", 
  ts: number 
}
```

### Extended Message Types

```typescript
// Audio Messages
type AudioData = {
  type: "audio.data",
  data: Uint8Array,
  sampleRate: number,
  channels: number,
  rms: number,
  ts: number
}

type VoiceActivity = {
  type: "voice.activity",
  active: boolean,
  confidence: number,
  ts: number
}

// System Messages
type SystemState = {
  type: "system.state",
  state: "idle" | "listening" | "processing" | "speaking" | "tool_execution" | "error",
  ts: number
}

type ErrorEvent = {
  type: "error",
  component: string,
  message: string,
  severity: "warning" | "error" | "critical",
  ts: number
}

// Memory Messages
type MemoryStore = {
  type: "memory.store",
  text: string,
  metadata?: Record<string, any>,
  ts: number
}

type MemoryRetrieve = {
  type: "memory.retrieve",
  query: string,
  topK: number,
  ts: number
}

type MemoryResult = {
  type: "memory.result",
  results: Array<{
    id: string,
    text: string,
    similarity: number,
    metadata?: Record<string, any>
  }>,
  ts: number
}
```

## Message Bus Architecture

### Bus Interface

```csharp
public interface IMessageBus
{
    event EventHandler<MessageEventArgs> MessageReceived;
    
    Task PublishAsync<T>(T message) where T : IMessage;
    Task SubscribeAsync<T>(Func<T, Task> handler) where T : IMessage;
    Task UnsubscribeAsync<T>(Func<T, Task> handler) where T : IMessage;
    
    Task StartAsync();
    Task StopAsync();
}

public interface IMessage
{
    string Type { get; }
    long Timestamp { get; }
}
```

### Message Handler Interface

```csharp
public interface IMessageHandler<T> where T : IMessage
{
    Task HandleAsync(T message);
    int Priority { get; }
    string[] Dependencies { get; }
}
```

## Message Flow Patterns

### 1. Basic Conversation Flow

```
[AudioCapture] → AudioData → [ASRService] → AsrPartial/AsrFinal
                                                      ↓
[Router] ← AsrFinal ← [MessageBus] ← [IntentClassifier]
    ↓
[LLMClient] → LlmDelta → [MessageBus] → [TTSService] → TtsStart/TtsStop
```

### 2. Tool Execution Flow

```
[Router] → ToolCall → [MessageBus] → [ToolRegistry] → [ToolExecutor]
                                                              ↓
[ToolExecutor] → ToolResult → [MessageBus] → [Router] → [LLMClient]
```

### 3. Memory Operations Flow

```
[LLMClient] → MemoryRetrieve → [MessageBus] → [VectorizerClient]
                                                      ↓
[VectorizerClient] → MemoryResult → [MessageBus] → [LLMClient]
```

### 4. Barge-in Flow

```
[AudioCapture] → VoiceActivity(active: true) → [MessageBus] → [TTSService]
                                                                    ↓
[TTSService] → TtsStop(reason: "bargein") → [MessageBus] → [ASRService]
```

## Message Routing

### Router Configuration

```csharp
public class MessageRouter
{
    private readonly Dictionary<string, List<IMessageHandler>> _handlers = new();
    
    public void RegisterHandler<T>(IMessageHandler<T> handler) where T : IMessage
    {
        var type = typeof(T).Name;
        if (!_handlers.ContainsKey(type))
            _handlers[type] = new List<IMessageHandler>();
        
        _handlers[type].Add(handler);
    }
    
    public async Task RouteAsync<T>(T message) where T : IMessage
    {
        var type = message.Type;
        if (_handlers.TryGetValue(type, out var handlers))
        {
            var tasks = handlers.Select(h => h.HandleAsync(message));
            await Task.WhenAll(tasks);
        }
    }
}
```

### Handler Registration

```csharp
// Register ASR handlers
router.RegisterHandler(new AsrPartialHandler());
router.RegisterHandler(new AsrFinalHandler());

// Register LLM handlers
router.RegisterHandler(new LlmDeltaHandler());

// Register Tool handlers
router.RegisterHandler(new ToolCallHandler());
router.RegisterHandler(new ToolResultHandler());

// Register TTS handlers
router.RegisterHandler(new TtsStartHandler());
router.RegisterHandler(new TtsStopHandler());
```

## Message Validation

### Schema Validation

```csharp
public class MessageValidator
{
    private readonly JsonSchema _schemas = new();
    
    public bool Validate<T>(T message) where T : IMessage
    {
        var schema = _schemas.GetSchema<T>();
        var json = JsonSerializer.Serialize(message);
        
        return schema.Validate(json);
    }
}
```

### Message Schemas

```json
{
  "AsrPartial": {
    "type": "object",
    "properties": {
      "type": { "const": "asr.partial" },
      "text": { "type": "string" },
      "ts": { "type": "number" }
    },
    "required": ["type", "text", "ts"]
  },
  "AsrFinal": {
    "type": "object",
    "properties": {
      "type": { "const": "asr.final" },
      "text": { "type": "string" },
      "ts": { "type": "number" }
    },
    "required": ["type", "text", "ts"]
  },
  "LlmDelta": {
    "type": "object",
    "properties": {
      "type": { "const": "llm.delta" },
      "model": { "enum": ["front", "planner"] },
      "token": { "type": "string" },
      "ts": { "type": "number" }
    },
    "required": ["type", "model", "token", "ts"]
  }
}
```

## Error Handling

### Error Message Types

```typescript
type ComponentError = {
  type: "component.error",
  component: string,
  error: string,
  severity: "warning" | "error" | "critical",
  ts: number
}

type BusError = {
  type: "bus.error",
  operation: string,
  error: string,
  ts: number
}
```

### Error Handling Strategy

```csharp
public class ErrorHandler : IMessageHandler<ComponentError>
{
    public async Task HandleAsync(ComponentError message)
    {
        switch (message.severity)
        {
            case "warning":
                _logger.LogWarning("Component {Component} warning: {Error}", 
                    message.component, message.error);
                break;
                
            case "error":
                _logger.LogError("Component {Component} error: {Error}", 
                    message.component, message.error);
                await NotifyUserAsync(message);
                break;
                
            case "critical":
                _logger.LogCritical("Component {Component} critical error: {Error}", 
                    message.component, message.error);
                await EmergencyShutdownAsync();
                break;
        }
    }
}
```

## Performance Considerations

### Message Batching

```csharp
public class BatchedMessageProcessor
{
    private readonly List<IMessage> _batch = new();
    private readonly Timer _flushTimer;
    
    public async Task ProcessBatchAsync()
    {
        if (_batch.Count == 0) return;
        
        var messages = _batch.ToArray();
        _batch.Clear();
        
        await _messageBus.PublishBatchAsync(messages);
    }
}
```

### Message Prioritization

```csharp
public enum MessagePriority
{
    Critical = 0,    // System errors, barge-in
    High = 1,        // Audio data, ASR results
    Normal = 2,      // LLM tokens, tool results
    Low = 3          // Logging, metrics
}

public class PriorityMessageQueue
{
    private readonly SortedDictionary<MessagePriority, Queue<IMessage>> _queues = new();
    
    public void Enqueue(IMessage message, MessagePriority priority)
    {
        if (!_queues.ContainsKey(priority))
            _queues[priority] = new Queue<IMessage>();
            
        _queues[priority].Enqueue(message);
    }
    
    public IMessage Dequeue()
    {
        foreach (var queue in _queues.Values)
        {
            if (queue.Count > 0)
                return queue.Dequeue();
        }
        
        return null;
    }
}
```

## Monitoring and Metrics

### Message Metrics

```csharp
public class MessageMetrics
{
    public long TotalMessages { get; set; }
    public long MessagesPerSecond { get; set; }
    public Dictionary<string, long> MessageTypeCounts { get; set; }
    public Dictionary<string, double> AverageProcessingTime { get; set; }
    public Dictionary<string, long> ErrorCounts { get; set; }
}
```

### Metrics Collection

```csharp
public class MetricsCollector : IMessageHandler<IMessage>
{
    private readonly MessageMetrics _metrics = new();
    
    public async Task HandleAsync(IMessage message)
    {
        var stopwatch = Stopwatch.StartNew();
        
        try
        {
            // Process message
            await ProcessMessageAsync(message);
        }
        finally
        {
            stopwatch.Stop();
            
            // Update metrics
            _metrics.TotalMessages++;
            _metrics.MessageTypeCounts[message.Type]++;
            _metrics.AverageProcessingTime[message.Type] = 
                (_metrics.AverageProcessingTime[message.Type] + stopwatch.ElapsedMilliseconds) / 2;
        }
    }
}
```

## Testing

### Message Bus Tests

```csharp
[Test]
public async Task MessageBus_ShouldRouteMessagesCorrectly()
{
    // Arrange
    var messageBus = new InMemoryMessageBus();
    var handler = new MockMessageHandler<AsrFinal>();
    
    await messageBus.SubscribeAsync<AsrFinal>(handler.HandleAsync);
    
    // Act
    var message = new AsrFinal { Type = "asr.final", Text = "Hello", Timestamp = DateTimeOffset.UtcNow.ToUnixTimeMilliseconds() };
    await messageBus.PublishAsync(message);
    
    // Assert
    Assert.IsTrue(handler.ReceivedMessages.Contains(message));
}

[Test]
public async Task MessageBus_ShouldHandleErrorsGracefully()
{
    // Arrange
    var messageBus = new InMemoryMessageBus();
    var errorHandler = new MockMessageHandler<ComponentError>();
    
    await messageBus.SubscribeAsync<ComponentError>(errorHandler.HandleAsync);
    
    // Act
    var errorMessage = new ComponentError 
    { 
        Type = "component.error", 
        Component = "ASR", 
        Error = "Model not found", 
        Severity = "error",
        Timestamp = DateTimeOffset.UtcNow.ToUnixTimeMilliseconds()
    };
    
    await messageBus.PublishAsync(errorMessage);
    
    // Assert
    Assert.IsTrue(errorHandler.ReceivedMessages.Contains(errorMessage));
}
```

## Configuration

### Message Bus Configuration

```json
{
  "messageBus": {
    "type": "InMemory",
    "maxQueueSize": 10000,
    "batchSize": 100,
    "batchTimeout": 100,
    "retryAttempts": 3,
    "retryDelay": 1000,
    "enableMetrics": true,
    "enableValidation": true
  }
}
```

## Conclusion

The message bus provides a robust, scalable communication layer for the Voxa system. It ensures loose coupling between components while maintaining performance and reliability requirements. The structured message types and flow patterns enable predictable behavior and easier debugging.
