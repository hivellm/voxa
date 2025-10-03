# VOXA - IMPLEMENTATION GUIDE

## Overview

This guide provides step-by-step instructions for implementing Voxa, a local voice assistant with real-time interaction capabilities.

## Prerequisites

### System Requirements
- Windows 10/11 (64-bit)
- .NET 8 SDK
- Visual Studio 2022 or VS Code
- Python 3.9+ (for ASR)
- CUDA Toolkit (for GPU acceleration)
- 128GB RAM minimum
- RTX 4090 or equivalent GPU

### Development Tools
- Git
- CMake
- Visual Studio Build Tools
- Python pip/conda

## Project Structure

```
voxa/
├── app/
│   ├── src/
│   │   ├── UI/                # WPF + Skia
│   │   ├── Audio/             # MicCapture, AudioPlayer
│   │   ├── ASR/               # ASRService
│   │   ├── LLM/               # LlmFrontClient, LlmPlannerClient
│   │   ├── Router/            # IntentClassifier, ToolRegistry
│   │   ├── TTS/               # PiperService
│   │   ├── Telemetry/         # EventLog, Exporters
│   │   └── Common/            # Contracts, Config, Utils
│   ├── models/                # GGUF, vozes Piper
│   ├── scripts/               # ft, merge, convert_gguf, run_llama_server, run_vllm
│   └── appsettings.json
├── tests/
└── docs/                      # Documentation
```

## Phase 1: Core Voice Interface

### Step 1: Create WPF Application

1. **Create new WPF project**
```bash
dotnet new wpf -n Voxa.UI
cd Voxa.UI
```

2. **Add required NuGet packages**
```xml
<PackageReference Include="SkiaSharp" Version="2.88.6" />
<PackageReference Include="NAudio" Version="2.2.1" />
<PackageReference Include="Microsoft.Extensions.Hosting" Version="8.0.0" />
<PackageReference Include="Microsoft.Extensions.Configuration" Version="8.0.0" />
<PackageReference Include="Microsoft.Extensions.Logging" Version="8.0.0" />
```

3. **Create main window with wave visualization**
```csharp
// MainWindow.xaml.cs
public partial class MainWindow : Window
{
    private readonly WaveCanvas _waveCanvas;
    private readonly StatusIndicator _statusIndicator;
    
    public MainWindow()
    {
        InitializeComponent();
        _waveCanvas = new WaveCanvas();
        _statusIndicator = new StatusIndicator();
    }
}
```

### Step 2: Implement Audio Capture

1. **Create AudioCapture service**
```csharp
public class AudioCapture : IAudioCapture
{
    private WaveInEvent _waveIn;
    private readonly ILogger<AudioCapture> _logger;
    
    public event EventHandler<AudioDataEventArgs> AudioDataReceived;
    public event EventHandler<VoiceActivityEventArgs> VoiceActivityDetected;
    
    public async Task StartAsync()
    {
        _waveIn = new WaveInEvent
        {
            WaveFormat = new WaveFormat(16000, 1)
        };
        
        _waveIn.DataAvailable += OnDataAvailable;
        _waveIn.StartRecording();
    }
    
    private void OnDataAvailable(object sender, WaveInEventArgs e)
    {
        var rms = CalculateRms(e.Buffer, e.BytesRecorded);
        var args = new AudioDataEventArgs
        {
            AudioData = e.Buffer,
            SampleRate = 16000,
            Channels = 1,
            RmsLevel = rms,
            Timestamp = DateTime.UtcNow
        };
        
        AudioDataReceived?.Invoke(this, args);
        
        if (rms > _vadThreshold)
        {
            VoiceActivityDetected?.Invoke(this, new VoiceActivityEventArgs
            {
                IsVoiceActive = true,
                Confidence = rms,
                Timestamp = DateTime.UtcNow
            });
        }
    }
}
```

### Step 3: Implement Wave Visualization

1. **Create SkiaSharp-based wave canvas**
```csharp
public class WaveCanvas : UserControl
{
    private SKCanvas _canvas;
    private readonly List<float> _audioLevels = new();
    private readonly Timer _animationTimer;
    
    public void UpdateAudioLevel(float level)
    {
        _audioLevels.Add(level);
        if (_audioLevels.Count > 100)
            _audioLevels.RemoveAt(0);
            
        InvalidateVisual();
    }
    
    protected override void OnPaintSurface(SKPaintSurfaceEventArgs e)
    {
        _canvas = e.Surface.Canvas;
        _canvas.Clear(SKColors.Black);
        
        DrawWave(_canvas);
    }
    
    private void DrawWave(SKCanvas canvas)
    {
        var centerX = ActualWidth / 2;
        var centerY = ActualHeight / 2;
        var maxRadius = Math.Min(centerX, centerY) - 50;
        
        for (int i = 0; i < _audioLevels.Count; i++)
        {
            var angle = (i * 2 * Math.PI) / _audioLevels.Count;
            var radius = maxRadius * _audioLevels[i];
            
            var x = centerX + radius * Math.Cos(angle);
            var y = centerY + radius * Math.Sin(angle);
            
            var paint = new SKPaint
            {
                Color = SKColors.Cyan,
                StrokeWidth = 2,
                IsAntialias = true
            };
            
            canvas.DrawCircle((float)x, (float)y, 3, paint);
        }
    }
}
```

## Phase 2: LLM Integration

### Step 1: Set up LLM Server

1. **Download and configure llama.cpp**
```bash
git clone https://github.com/ggerganov/llama.cpp.git
cd llama.cpp
make -j$(nproc)
```

2. **Download Qwen2.5-3B-Instruct model**
```bash
# Download GGUF model
wget https://huggingface.co/Qwen/Qwen2.5-3B-Instruct-GGUF/resolve/main/qwen2.5-3b-instruct-q4_k_m.gguf
```

3. **Start LLM server**
```bash
./server -m qwen2.5-3b-instruct-q4_k_m.gguf -c 4096 -ngl -1 --port 8081
```

### Step 2: Implement LLM Client

1. **Create LLM client service**
```csharp
public class LLMClient : ILLMClient
{
    private readonly HttpClient _httpClient;
    private readonly ILogger<LLMClient> _logger;
    
    public async Task<string> CompleteAsync(string prompt, LLMOptions options = null)
    {
        var request = new
        {
            prompt = prompt,
            max_tokens = options?.MaxTokens ?? 512,
            temperature = options?.Temperature ?? 0.7,
            stream = false
        };
        
        var json = JsonSerializer.Serialize(request);
        var content = new StringContent(json, Encoding.UTF8, "application/json");
        
        var response = await _httpClient.PostAsync("http://localhost:8081/completion", content);
        var responseJson = await response.Content.ReadAsStringAsync();
        
        var result = JsonSerializer.Deserialize<LLMResponse>(responseJson);
        return result.choices[0].text;
    }
    
    public async IAsyncEnumerable<string> StreamCompleteAsync(string prompt, LLMOptions options = null)
    {
        var request = new
        {
            prompt = prompt,
            max_tokens = options?.MaxTokens ?? 512,
            temperature = options?.Temperature ?? 0.7,
            stream = true
        };
        
        var json = JsonSerializer.Serialize(request);
        var content = new StringContent(json, Encoding.UTF8, "application/json");
        
        var response = await _httpClient.PostAsync("http://localhost:8081/completion", content);
        var stream = await response.Content.ReadAsStreamAsync();
        
        using var reader = new StreamReader(stream);
        string line;
        while ((line = await reader.ReadLineAsync()) != null)
        {
            if (line.StartsWith("data: "))
            {
                var data = line.Substring(6);
                if (data == "[DONE]") break;
                
                var token = JsonSerializer.Deserialize<StreamToken>(data);
                yield return token.content;
            }
        }
    }
}
```

### Step 3: Implement Response Classification

1. **Create intent classifier**
```csharp
public class ResponseClassifier : IResponseClassifier
{
    private readonly ILLMClient _llmClient;
    
    public async Task<ResponseType> ClassifyIntentAsync(string userInput)
    {
        var prompt = $@"Classify the following user input into one of these categories:
1. DIRECT_RESPONSE - Simple question that can be answered directly
2. TOOL_CALL - Requires external tool or service
3. PLANNER_REQUIRED - Complex reasoning or planning needed

User input: ""{userInput}""

Respond with only the category name:";
        
        var response = await _llmClient.CompleteAsync(prompt, new LLMOptions
        {
            MaxTokens = 10,
            Temperature = 0.1
        });
        
        return Enum.Parse<ResponseType>(response.Trim());
    }
}

public enum ResponseType
{
    DirectResponse,
    ToolCall,
    PlannerRequired
}
```

## Phase 3: Advanced Features

### Step 1: Integrate Vectorizer

1. **Create vectorizer client**
```csharp
public class VectorizerClient : IVectorizerClient
{
    private readonly HttpClient _httpClient;
    
    public async Task<string> StoreMemoryAsync(string text, Dictionary<string, object> metadata = null)
    {
        var request = new
        {
            text = text,
            metadata = metadata ?? new Dictionary<string, object>()
        };
        
        var json = JsonSerializer.Serialize(request);
        var content = new StringContent(json, Encoding.UTF8, "application/json");
        
        var response = await _httpClient.PostAsync("http://localhost:8085/insert", content);
        var responseJson = await response.Content.ReadAsStringAsync();
        
        var result = JsonSerializer.Deserialize<InsertResult>(responseJson);
        return result.id;
    }
    
    public async Task<MemoryResult[]> SearchMemoriesAsync(string query, int topK = 5)
    {
        var request = new
        {
            query = query,
            limit = topK
        };
        
        var json = JsonSerializer.Serialize(request);
        var content = new StringContent(json, Encoding.UTF8, "application/json");
        
        var response = await _httpClient.PostAsync("http://localhost:8085/search", content);
        var responseJson = await response.Content.ReadAsStringAsync();
        
        var result = JsonSerializer.Deserialize<SearchResult>(responseJson);
        return result.results;
    }
}
```

### Step 2: Implement MCP Integration

1. **Create MCP client**
```csharp
public class MCPClient : IMCPClient
{
    private readonly Dictionary<string, ToolDefinition> _tools = new();
    
    public async Task<ToolResult> CallToolAsync(string toolName, Dictionary<string, object> parameters)
    {
        if (!_tools.TryGetValue(toolName, out var tool))
            throw new ArgumentException($"Tool '{toolName}' not found");
        
        // Validate parameters against schema
        ValidateParameters(parameters, tool.ParametersSchema);
        
        // Execute tool based on type
        switch (toolName)
        {
            case "web_search":
                return await ExecuteWebSearch(parameters);
            case "doc_lookup":
                return await ExecuteDocLookup(parameters);
            default:
                throw new NotSupportedException($"Tool '{toolName}' not implemented");
        }
    }
    
    private async Task<ToolResult> ExecuteWebSearch(Dictionary<string, object> parameters)
    {
        var query = parameters["query"].ToString();
        var freshness = parameters.ContainsKey("freshness") ? parameters["freshness"].ToString() : "7d";
        
        // Implement web search logic
        var results = await SearchWeb(query, freshness);
        
        return new ToolResult
        {
            Success = true,
            Result = results,
            ExecutionTime = TimeSpan.FromSeconds(2)
        };
    }
}
```

### Step 3: Implement TTS Integration

1. **Set up Piper TTS**
```bash
# Download Piper
wget https://github.com/rhasspy/piper/releases/download/v1.2.0/piper_windows_amd64.tar.gz
tar -xzf piper_windows_amd64.tar.gz

# Download Portuguese voice model
wget https://huggingface.co/rhasspy/piper-voices/resolve/v1.0.0/pt/pt_BR/faber/medium/piper_pt_BR_faber_medium.onnx
wget https://huggingface.co/rhasspy/piper-voices/resolve/v1.0.0/pt/pt_BR/faber/medium/piper_pt_BR_faber_medium.onnx.json
```

2. **Implement TTS service**
```csharp
public class TTSService : ITTSService
{
    private readonly Process _piperProcess;
    
    public async Task<Stream> SynthesizeAsync(string text, TTSOptions options = null)
    {
        var startInfo = new ProcessStartInfo
        {
            FileName = "piper.exe",
            Arguments = $"--model piper_pt_BR_faber_medium.onnx --output-raw",
            UseShellExecute = false,
            RedirectStandardInput = true,
            RedirectStandardOutput = true,
            CreateNoWindow = true
        };
        
        _piperProcess = Process.Start(startInfo);
        
        // Send text to Piper
        await _piperProcess.StandardInput.WriteAsync(text);
        _piperProcess.StandardInput.Close();
        
        // Read audio output
        var audioStream = new MemoryStream();
        await _piperProcess.StandardOutput.BaseStream.CopyToAsync(audioStream);
        
        return audioStream;
    }
}
```

## Phase 4: Optimization

### Step 1: Performance Tuning

1. **Implement caching**
```csharp
public class ResponseCache
{
    private readonly MemoryCache _cache = new MemoryCache(new MemoryCacheOptions
    {
        SizeLimit = 1000,
        CompactionPercentage = 0.25
    });
    
    public async Task<string> GetOrSetAsync(string key, Func<Task<string>> factory)
    {
        if (_cache.TryGetValue(key, out string cachedResponse))
            return cachedResponse;
        
        var response = await factory();
        _cache.Set(key, response, TimeSpan.FromMinutes(30));
        return response;
    }
}
```

2. **Optimize audio processing**
```csharp
public class OptimizedAudioProcessor
{
    private readonly CircularBuffer<byte> _audioBuffer;
    private readonly Task _processingTask;
    
    public OptimizedAudioProcessor()
    {
        _audioBuffer = new CircularBuffer<byte>(1024 * 1024); // 1MB buffer
        _processingTask = Task.Run(ProcessAudioLoop);
    }
    
    private async Task ProcessAudioLoop()
    {
        while (true)
        {
            if (_audioBuffer.Count > 0)
            {
                var audioData = _audioBuffer.Read(1024);
                await ProcessAudioChunk(audioData);
            }
            await Task.Delay(10); // 10ms processing interval
        }
    }
}
```

### Step 2: Error Handling

1. **Implement comprehensive error handling**
```csharp
public class ErrorHandler
{
    private readonly ILogger<ErrorHandler> _logger;
    private readonly RetryPolicy _retryPolicy;
    
    public async Task<T> ExecuteWithRetryAsync<T>(Func<Task<T>> operation, string operationName)
    {
        var attempts = 0;
        var delay = _retryPolicy.InitialDelay;
        
        while (attempts < _retryPolicy.MaxRetries)
        {
            try
            {
                return await operation();
            }
            catch (Exception ex)
            {
                attempts++;
                _logger.LogWarning(ex, "Operation '{OperationName}' failed (attempt {Attempt}/{MaxRetries})", 
                    operationName, attempts, _retryPolicy.MaxRetries);
                
                if (attempts >= _retryPolicy.MaxRetries)
                {
                    _logger.LogError(ex, "Operation '{OperationName}' failed after {MaxRetries} attempts", 
                        operationName, _retryPolicy.MaxRetries);
                    throw;
                }
                
                if (_retryPolicy.ShouldRetry?.Invoke(ex) == false)
                    throw;
                
                await Task.Delay(delay);
                delay = TimeSpan.FromMilliseconds(Math.Min(
                    delay.TotalMilliseconds * _retryPolicy.BackoffMultiplier,
                    _retryPolicy.MaxDelay.TotalMilliseconds));
            }
        }
        
        throw new InvalidOperationException($"Operation '{operationName}' failed unexpectedly");
    }
}
```

## Testing Strategy

### Unit Tests
```csharp
[Test]
public async Task AudioCapture_ShouldDetectVoiceActivity()
{
    // Arrange
    var audioCapture = new AudioCapture();
    var voiceDetected = false;
    
    audioCapture.VoiceActivityDetected += (sender, args) => voiceDetected = true;
    
    // Act
    await audioCapture.StartAsync();
    // Simulate audio input
    await Task.Delay(1000);
    
    // Assert
    Assert.IsTrue(voiceDetected);
}
```

### Integration Tests
```csharp
[Test]
public async Task EndToEnd_ShouldProcessVoiceInput()
{
    // Arrange
    var services = CreateTestServices();
    var voxa = services.GetRequiredService<VoxaOrchestrator>();
    
    // Act
    var result = await voxa.ProcessVoiceInputAsync("test audio data");
    
    // Assert
    Assert.IsNotNull(result);
    Assert.IsTrue(result.Success);
}
```

## Deployment

### 1. Create Windows Installer
```xml
<!-- Voxa.Setup.wxs -->
<Wix xmlns="http://schemas.microsoft.com/wix/2006/wi">
  <Product Id="*" Name="Voxa Voice Assistant" Language="1033" Version="1.0.0.0" 
           Manufacturer="Voxa Team" UpgradeCode="PUT-GUID-HERE">
    
    <Package InstallerVersion="200" Compressed="yes" InstallScope="perMachine" />
    
    <MajorUpgrade DowngradeErrorMessage="A newer version is already installed." />
    <MediaTemplate />
    
    <Feature Id="ProductFeature" Title="Voxa Voice Assistant" Level="1">
      <ComponentGroupRef Id="ProductComponents" />
    </Feature>
  </Product>
</Wix>
```

### 2. Create Deployment Script
```powershell
# deploy.ps1
param(
    [string]$Configuration = "Release",
    [string]$OutputPath = ".\dist"
)

Write-Host "Building Voxa..." -ForegroundColor Green
dotnet build -c $Configuration

Write-Host "Publishing application..." -ForegroundColor Green
dotnet publish -c $Configuration -o $OutputPath

Write-Host "Copying models..." -ForegroundColor Green
Copy-Item -Path ".\models\*" -Destination "$OutputPath\models\" -Recurse

Write-Host "Creating installer..." -ForegroundColor Green
& "C:\Program Files (x86)\WiX Toolset v3.11\bin\candle.exe" Voxa.Setup.wxs
& "C:\Program Files (x86)\WiX Toolset v3.11\bin\light.exe" Voxa.Setup.wixobj

Write-Host "Deployment complete!" -ForegroundColor Green
```

## Monitoring and Maintenance

### 1. Implement Health Checks
```csharp
public class HealthCheckService : BackgroundService
{
    private readonly IServiceProvider _serviceProvider;
    private readonly ILogger<HealthCheckService> _logger;
    
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            await CheckSystemHealth();
            await Task.Delay(TimeSpan.FromMinutes(1), stoppingToken);
        }
    }
    
    private async Task CheckSystemHealth()
    {
        var healthChecks = new[]
        {
            CheckAudioDevices(),
            CheckLLMServer(),
            CheckMemoryUsage(),
            CheckDiskSpace()
        };
        
        var results = await Task.WhenAll(healthChecks);
        var unhealthy = results.Where(r => !r.IsHealthy).ToList();
        
        if (unhealthy.Any())
        {
            _logger.LogWarning("System health issues detected: {Issues}", 
                string.Join(", ", unhealthy.Select(h => h.Status)));
        }
    }
}
```

### 2. Performance Monitoring
```csharp
public class PerformanceMonitor
{
    private readonly IMetricsCollector _metricsCollector;
    
    public void RecordLatency(string operation, TimeSpan latency)
    {
        _metricsCollector.RecordLatency(operation, latency);
        
        if (latency > TimeSpan.FromSeconds(2))
        {
            _logger.LogWarning("Slow operation detected: {Operation} took {Latency}", 
                operation, latency);
        }
    }
    
    public void RecordResourceUsage()
    {
        var process = Process.GetCurrentProcess();
        var memoryMB = process.WorkingSet64 / 1024 / 1024;
        var cpuPercent = GetCpuUsage();
        
        _metricsCollector.RecordResourceUsage("memory_mb", memoryMB);
        _metricsCollector.RecordResourceUsage("cpu_percent", cpuPercent);
    }
}
```

## Key Flows

### Basic Conversation Flow
```
MicCapture ON → ASR parciais → UI anima
ASR final → Router consulta Front
Front: resposta direta ou tool-JSON
Se tool: executar, retornar result → Front resume
TTS fala; se VAD subir, parar (barge-in) e voltar ao passo 1
```

### Continuous Fine-tuning Flow
```
Exportar JSONL (SFT/chat, tool_call, DPO)
Rodar Axolotl (QLoRA) → merge → GGUF
Carregar versão nova no llama.cpp; smoke-tests; canary release
```

## Initial Tasks (Deliverables)

### Core Components
1. **UI WPF** com WaveCanvas + botão/tecla "Segurar para Falar"
2. **Mic → ASR** parciais/final (faster-whisper/whisper.net) + VAD
3. **Cliente llama.cpp** (HTTP stream) para Front 3B; PlannerClient com timeout
4. **ToolRegistry** com web_search, doc_lookup, assistant_deep
5. **Piper streaming** + AudioPlayer + barge-in
6. **Logs estruturados** + exportadores JSONL para SFT/DPO

### Scripts and Automation
7. **Scripts run_front.bat** (3B) e run_planner.bat (7B) com ajustes de VRAM/KV
8. **Smoke-tests**: JSON válido, barge-in, TTFB e tok/s mínimos

## Conclusion

This implementation guide provides a comprehensive roadmap for building Voxa. The modular architecture allows for incremental development and testing, while the performance optimization techniques ensure smooth real-time operation. Regular monitoring and maintenance will ensure the system remains stable and responsive over time.
```