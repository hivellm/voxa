# VOXA - API SPECIFICATION

## Overview

This document defines the APIs and protocols used by Voxa for internal communication, external tool integration, and model server interactions.

## Internal APIs

### 1. Audio Processing API

#### AudioCapture Interface
```csharp
public interface IAudioCapture
{
    event EventHandler<AudioDataEventArgs> AudioDataReceived;
    event EventHandler<VoiceActivityEventArgs> VoiceActivityDetected;
    
    Task StartAsync();
    Task StopAsync();
    bool IsCapturing { get; }
    AudioDeviceInfo CurrentDevice { get; }
    Task SetDeviceAsync(AudioDeviceInfo device);
}

public class AudioDataEventArgs : EventArgs
{
    public byte[] AudioData { get; set; }
    public int SampleRate { get; set; }
    public int Channels { get; set; }
    public double RmsLevel { get; set; }
    public DateTime Timestamp { get; set; }
}

public class VoiceActivityEventArgs : EventArgs
{
    public bool IsVoiceActive { get; set; }
    public double Confidence { get; set; }
    public DateTime Timestamp { get; set; }
}
```

#### AudioPlayer Interface
```csharp
public interface IAudioPlayer
{
    event EventHandler<PlaybackEventArgs> PlaybackStarted;
    event EventHandler<PlaybackEventArgs> PlaybackCompleted;
    event EventHandler<PlaybackEventArgs> PlaybackInterrupted;
    
    Task PlayAsync(Stream audioStream);
    Task PlayAsync(byte[] audioData);
    Task StopAsync();
    Task PauseAsync();
    Task ResumeAsync();
    bool IsPlaying { get; }
    double Volume { get; set; }
}

public class PlaybackEventArgs : EventArgs
{
    public string AudioId { get; set; }
    public TimeSpan Duration { get; set; }
    public DateTime Timestamp { get; set; }
}
```

### 2. ASR (Speech Recognition) API

#### ASRService Interface
```csharp
public interface IASRService
{
    event EventHandler<TranscriptionEventArgs> PartialTranscription;
    event EventHandler<TranscriptionEventArgs> FinalTranscription;
    event EventHandler<ASRErrorEventArgs> ErrorOccurred;
    
    Task<string> TranscribeAsync(byte[] audioData);
    Task<string> TranscribeAsync(Stream audioStream);
    Task StartContinuousTranscriptionAsync();
    Task StopContinuousTranscriptionAsync();
    bool IsTranscribing { get; }
}

public class TranscriptionEventArgs : EventArgs
{
    public string Text { get; set; }
    public bool IsPartial { get; set; }
    public double Confidence { get; set; }
    public TimeSpan Duration { get; set; }
    public DateTime Timestamp { get; set; }
}

public class ASRErrorEventArgs : EventArgs
{
    public string ErrorMessage { get; set; }
    public Exception Exception { get; set; }
    public DateTime Timestamp { get; set; }
}
```

### 3. LLM Client API

#### LLMClient Interface
```csharp
public interface ILLMClient
{
    event EventHandler<TokenEventArgs> TokenReceived;
    event EventHandler<CompletionEventArgs> CompletionCompleted;
    event EventHandler<LLMErrorEventArgs> ErrorOccurred;
    
    Task<string> CompleteAsync(string prompt, LLMOptions options = null);
    IAsyncEnumerable<string> StreamCompleteAsync(string prompt, LLMOptions options = null);
    Task<LLMResponse> CompleteWithMetadataAsync(string prompt, LLMOptions options = null);
}

public class LLMOptions
{
    public int MaxTokens { get; set; } = 512;
    public double Temperature { get; set; } = 0.7;
    public double TopP { get; set; } = 0.9;
    public string[] StopSequences { get; set; }
    public bool Stream { get; set; } = false;
    public Dictionary<string, object> CustomParameters { get; set; }
}

public class LLMResponse
{
    public string Text { get; set; }
    public int TokensUsed { get; set; }
    public TimeSpan ProcessingTime { get; set; }
    public double Perplexity { get; set; }
    public Dictionary<string, object> Metadata { get; set; }
}

public class TokenEventArgs : EventArgs
{
    public string Token { get; set; }
    public int TokenIndex { get; set; }
    public DateTime Timestamp { get; set; }
}

public class CompletionEventArgs : EventArgs
{
    public string FullText { get; set; }
    public int TotalTokens { get; set; }
    public TimeSpan TotalTime { get; set; }
    public DateTime Timestamp { get; set; }
}
```

### 4. TTS (Text-to-Speech) API

#### TTSService Interface
```csharp
public interface ITTSService
{
    event EventHandler<TTSProgressEventArgs> SynthesisProgress;
    event EventHandler<TTSCompletedEventArgs> SynthesisCompleted;
    event EventHandler<TTSErrorEventArgs> ErrorOccurred;
    
    Task<Stream> SynthesizeAsync(string text, TTSOptions options = null);
    Task<byte[]> SynthesizeToBytesAsync(string text, TTSOptions options = null);
    Task StartStreamingSynthesisAsync(string text, TTSOptions options = null);
    Task StopStreamingSynthesisAsync();
    bool IsSynthesizing { get; }
}

public class TTSOptions
{
    public string Voice { get; set; } = "default";
    public double Speed { get; set; } = 1.0;
    public double Pitch { get; set; } = 1.0;
    public double Volume { get; set; } = 1.0;
    public string Language { get; set; } = "pt-BR";
    public bool Stream { get; set; } = false;
}

public class TTSProgressEventArgs : EventArgs
{
    public int ProgressPercentage { get; set; }
    public string CurrentWord { get; set; }
    public DateTime Timestamp { get; set; }
}

public class TTSCompletedEventArgs : EventArgs
{
    public Stream AudioStream { get; set; }
    public TimeSpan Duration { get; set; }
    public DateTime Timestamp { get; set; }
}
```

## External Tool Integration API

### 1. MCP (Model Context Protocol) Client

#### MCPClient Interface
```csharp
public interface IMCPClient
{
    event EventHandler<ToolCallEventArgs> ToolCallStarted;
    event EventHandler<ToolResultEventArgs> ToolResultReceived;
    event EventHandler<MCPErrorEventArgs> ErrorOccurred;
    
    Task<ToolResult> CallToolAsync(string toolName, Dictionary<string, object> parameters);
    Task<ToolResult> CallToolAsync(ToolCall toolCall);
    Task RegisterToolAsync(ToolDefinition toolDefinition);
    Task UnregisterToolAsync(string toolName);
    Task<ToolDefinition[]> GetAvailableToolsAsync();
}

public class ToolCall
{
    public string ToolName { get; set; }
    public Dictionary<string, object> Parameters { get; set; }
    public string CallId { get; set; }
    public DateTime Timestamp { get; set; }
}

public class ToolResult
{
    public string CallId { get; set; }
    public bool Success { get; set; }
    public object Result { get; set; }
    public string ErrorMessage { get; set; }
    public TimeSpan ExecutionTime { get; set; }
    public DateTime Timestamp { get; set; }
}

public class ToolDefinition
{
    public string Name { get; set; }
    public string Description { get; set; }
    public JsonSchema ParametersSchema { get; set; }
    public string[] RequiredParameters { get; set; }
    public Dictionary<string, object> Metadata { get; set; }
}
```

### 2. Vectorizer Integration API

#### VectorizerClient Interface
```csharp
public interface IVectorizerClient
{
    event EventHandler<MemoryStoredEventArgs> MemoryStored;
    event EventHandler<MemoryRetrievedEventArgs> MemoryRetrieved;
    event EventHandler<VectorizerErrorEventArgs> ErrorOccurred;
    
    Task<string> StoreMemoryAsync(string text, Dictionary<string, object> metadata = null);
    Task<MemoryResult[]> SearchMemoriesAsync(string query, int topK = 5);
    Task<MemoryResult[]> GetRecentMemoriesAsync(int count = 10);
    Task<bool> DeleteMemoryAsync(string memoryId);
    Task<MemoryStats> GetMemoryStatsAsync();
}

public class MemoryResult
{
    public string Id { get; set; }
    public string Text { get; set; }
    public double SimilarityScore { get; set; }
    public Dictionary<string, object> Metadata { get; set; }
    public DateTime CreatedAt { get; set; }
}

public class MemoryStats
{
    public int TotalMemories { get; set; }
    public long TotalSizeBytes { get; set; }
    public DateTime LastUpdated { get; set; }
    public Dictionary<string, int> MemoryTypes { get; set; }
}
```

## Model Server APIs

### 1. LLM Server API (llama.cpp)

#### HTTP Endpoints
```http
POST /completion
Content-Type: application/json

{
  "prompt": "string",
  "max_tokens": 512,
  "temperature": 0.7,
  "top_p": 0.9,
  "stream": true,
  "stop": ["string"]
}

Response (Streaming):
data: {"token": "string", "index": 0}
data: {"token": "string", "index": 1}
data: [DONE]
```

#### WebSocket Endpoint
```javascript
ws://localhost:8081/ws

// Send message
{
  "type": "completion",
  "prompt": "string",
  "options": {
    "max_tokens": 512,
    "temperature": 0.7
  }
}

// Receive tokens
{
  "type": "token",
  "token": "string",
  "index": 0
}
```

### 2. ASR Server API (Fast-Whisper)

#### HTTP Endpoints
```http
POST /transcribe
Content-Type: multipart/form-data

file: audio.wav
language: pt
model: base
vad: true

Response:
{
  "text": "transcribed text",
  "language": "pt",
  "confidence": 0.95,
  "duration": 3.2
}
```

#### WebSocket Endpoint
```javascript
ws://localhost:8083/ws

// Send audio chunks
{
  "type": "audio",
  "data": "base64_encoded_audio"
}

// Receive transcription
{
  "type": "transcription",
  "text": "partial text",
  "is_final": false
}
```

### 3. TTS Server API (Piper)

#### HTTP Endpoints
```http
POST /synthesize
Content-Type: application/json

{
  "text": "text to synthesize",
  "voice": "pt-BR-voice",
  "speed": 1.0,
  "pitch": 1.0
}

Response: audio/wav stream
```

#### WebSocket Endpoint
```javascript
ws://localhost:8084/ws

// Send text
{
  "type": "synthesize",
  "text": "text to synthesize",
  "options": {
    "voice": "pt-BR-voice",
    "speed": 1.0
  }
}

// Receive audio chunks
{
  "type": "audio",
  "data": "base64_encoded_audio",
  "is_final": false
}
```

## Configuration API

### 1. Configuration Manager

#### ConfigurationManager Interface
```csharp
public interface IConfigurationManager
{
    Task<T> GetConfigurationAsync<T>(string key);
    Task SetConfigurationAsync<T>(string key, T value);
    Task<Dictionary<string, object>> GetAllConfigurationsAsync();
    Task ResetConfigurationAsync(string key);
    Task ResetAllConfigurationsAsync();
    event EventHandler<ConfigurationChangedEventArgs> ConfigurationChanged;
}

public class ConfigurationChangedEventArgs : EventArgs
{
    public string Key { get; set; }
    public object OldValue { get; set; }
    public object NewValue { get; set; }
    public DateTime Timestamp { get; set; }
}
```

### 2. Configuration Schema

#### Model Configuration
```json
{
  "models": {
    "front_llm": {
      "name": "Qwen2.5-3B-Instruct",
      "server_url": "http://localhost:8081",
      "model_path": "./models/qwen2.5-3b-instruct.gguf",
      "quantization": "Q4_K_M",
      "gpu_layers": -1,
      "context_size": 4096,
      "batch_size": 512
    },
    "planner_llm": {
      "name": "Qwen2.5-7B-Instruct",
      "server_url": "http://localhost:8082",
      "model_path": "./models/qwen2.5-7b-instruct.gguf",
      "quantization": "Q4_K_M",
      "gpu_layers": 20,
      "context_size": 8192,
      "batch_size": 256
    }
  }
}
```

#### Audio Configuration
```json
{
  "audio": {
    "input": {
      "device": "default",
      "sample_rate": 16000,
      "channels": 1,
      "format": "PCM16",
      "buffer_size": 1024
    },
    "output": {
      "device": "default",
      "sample_rate": 22050,
      "channels": 1,
      "format": "PCM16",
      "buffer_size": 2048
    },
    "vad": {
      "enabled": true,
      "threshold": 0.5,
      "min_duration": 0.1,
      "max_duration": 10.0,
      "silence_duration": 0.5
    }
  }
}
```

## Error Handling

### 1. Error Types

#### AudioError
```csharp
public class AudioError : Exception
{
    public AudioErrorType ErrorType { get; }
    public string DeviceName { get; }
    public AudioError(AudioErrorType errorType, string message, string deviceName = null) : base(message)
    {
        ErrorType = errorType;
        DeviceName = deviceName;
    }
}

public enum AudioErrorType
{
    DeviceNotFound,
    DeviceInUse,
    FormatNotSupported,
    PermissionDenied,
    HardwareFailure
}
```

#### LLMError
```csharp
public class LLMError : Exception
{
    public LLMErrorType ErrorType { get; }
    public string ModelName { get; }
    public LLMError(LLMErrorType errorType, string message, string modelName = null) : base(message)
    {
        ErrorType = errorType;
        ModelName = modelName;
    }
}

public enum LLMErrorType
{
    ModelNotFound,
    ServerUnavailable,
    Timeout,
    InvalidRequest,
    OutOfMemory,
    InvalidResponse
}
```

### 2. Error Recovery

#### Retry Policy
```csharp
public class RetryPolicy
{
    public int MaxRetries { get; set; } = 3;
    public TimeSpan InitialDelay { get; set; } = TimeSpan.FromSeconds(1);
    public TimeSpan MaxDelay { get; set; } = TimeSpan.FromSeconds(30);
    public double BackoffMultiplier { get; set; } = 2.0;
    public Func<Exception, bool> ShouldRetry { get; set; }
}
```

## Performance Monitoring

### 1. Metrics Collection

#### MetricsCollector Interface
```csharp
public interface IMetricsCollector
{
    void RecordLatency(string operation, TimeSpan latency);
    void RecordThroughput(string operation, int count);
    void RecordError(string operation, Exception exception);
    void RecordResourceUsage(string resource, double value);
    Task<MetricsSnapshot> GetSnapshotAsync();
}

public class MetricsSnapshot
{
    public DateTime Timestamp { get; set; }
    public Dictionary<string, LatencyMetrics> Latencies { get; set; }
    public Dictionary<string, ThroughputMetrics> Throughputs { get; set; }
    public Dictionary<string, ErrorMetrics> Errors { get; set; }
    public Dictionary<string, ResourceMetrics> Resources { get; set; }
}
```

### 2. Health Checks

#### HealthCheck Interface
```csharp
public interface IHealthCheck
{
    Task<HealthStatus> CheckHealthAsync();
    string Name { get; }
    TimeSpan Timeout { get; }
}

public class HealthStatus
{
    public bool IsHealthy { get; set; }
    public string Status { get; set; }
    public Dictionary<string, object> Details { get; set; }
    public DateTime Timestamp { get; set; }
}
```

## Security

### 1. Authentication

#### AuthenticationManager Interface
```csharp
public interface IAuthenticationManager
{
    Task<bool> AuthenticateAsync(string token);
    Task<string> GenerateTokenAsync(string userId);
    Task RevokeTokenAsync(string token);
    Task<bool> IsTokenValidAsync(string token);
}
```

### 2. Authorization

#### AuthorizationManager Interface
```csharp
public interface IAuthorizationManager
{
    Task<bool> HasPermissionAsync(string userId, string permission);
    Task<bool> CanAccessResourceAsync(string userId, string resource);
    Task<string[]> GetUserPermissionsAsync(string userId);
}
```

## Conclusion

This API specification provides a comprehensive foundation for implementing Voxa with clear interfaces, error handling, and performance monitoring. The modular design allows for easy extension and maintenance while ensuring consistent behavior across all components.
