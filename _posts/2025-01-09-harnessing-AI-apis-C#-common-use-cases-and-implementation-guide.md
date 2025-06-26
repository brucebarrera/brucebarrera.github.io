---
layout: post
title: "Harnessing AI APIs in C# and .NET: Common Use Cases and Implementation Guide"
date: 2025-01-09 20:18:00 -0500
categories: [AI]
tags: [AI, csharp, Azure, REST, API]
published: true
---

AI APIs are widely used in C# and .NET applications to integrate advanced AI capabilities without building models from scratch. In this article it'll be shown below common use cases and how they are typically implemented in C#/.NET:

### Common Usages of AI APIs in C#/.NET

1. **Natural Language Processing (NLP)**  
   - **Use Cases**: Text generation, sentiment analysis, chatbots, text summarization, language translation, entity recognition.  
   - **APIs**: OpenAI (ChatGPT), Azure Cognitive Services (Text Analytics, Translator), Google Cloud Natural Language, Hugging Face.  
   - **Implementation Example**:  
     Using OpenAI’s API to generate text:
     ```csharp
     using OpenAI_API;
     using OpenAI_API.Completions;

     var openAi = new OpenAIAPI("your-api-key");
     var prompt = "Write a short story about a robot.";
     var result = await openAi.Completions.CreateCompletionAsync(new CompletionRequest(prompt, model: "text-davinci-003", max_tokens: 200));
     Console.WriteLine(result.Completions[0].Text);
     ```
     - **NuGet Package**: `OpenAI_API` or `HttpClient` for custom API calls.  
     - **Details**: Developers use REST clients like `HttpClient` or SDKs to send text inputs to the API and process JSON responses.

2. **Speech and Audio Processing**  
   - **Use Cases**: Speech-to-text, text-to-speech, voice recognition, audio sentiment analysis.  
   - **APIs**: Azure Speech Service, Google Cloud Speech-to-Text, AWS Transcribe.  
   - **Implementation Example**:  
     Using Azure Speech Service for speech-to-text:
     ```csharp
     using Microsoft.CognitiveServices.Speech;

     var config = SpeechConfig.FromSubscription("your-key", "your-region");
     using var recognizer = new SpeechRecognizer(config);
     var result = await recognizer.RecognizeOnceAsync();
     Console.WriteLine($"Recognized: {result.Text}");
     ```
     - **NuGet Package**: `Microsoft.CognitiveServices.Speech`.  
     - **Details**: Stream audio or process WAV files, handling async operations for real-time transcription.

3. **Computer Vision**  
   - **Use Cases**: Image classification, object detection, facial recognition, OCR (text extraction from images).  
   - **APIs**: Azure Computer Vision, Google Cloud Vision, AWS Rekognition.  
   - **Implementation Example**:  
     Using Azure Computer Vision for OCR:
     ```csharp
     using Microsoft.Azure.CognitiveServices.Vision.ComputerVision;
     using Microsoft.Azure.CognitiveServices.Vision.ComputerVision.Models;

     var client = new ComputerVisionClient(new ApiKeyServiceClientCredentials("your-key"))
     {
         Endpoint = "your-endpoint"
     };
     var imageUrl = "https://example.com/image.jpg";
     var result = await client.ReadAsync(imageUrl);
     Console.WriteLine(result.Areas.SelectMany(a => a.Lines).Select(l => l.Text));
     ```
     - **NuGet Package**: `Microsoft.Azure.CognitiveServices.Vision.ComputerVision`.  
     - **Details**: Process images via URLs or file streams, extracting features like text or objects.

4. **Machine Learning Model Inference**  
   - **Use Cases**: Predictive analytics, classification, regression, recommendation systems.  
   - **APIs**: Azure Machine Learning, AWS SageMaker, custom models via ONNX Runtime.  
   - **Implementation Example**:  
     Using ONNX Runtime for model inference:
     ```csharp
     using Microsoft.ML.OnnxRuntime;
     using Microsoft.ML.OnnxRuntime.Tensors;

     var session = new InferenceSession("model.onnx");
     var input = new DenseTensor<float>(new[] { 1, featureLength });
     var inputs = new List<NamedOnnxValue> { NamedOnnxValue.CreateFromTensor("input", input) };
     var results = session.Run(inputs);
     Console.WriteLine(results[0].AsTensor<float>().ToArray());
     ```
     - **NuGet Package**: `Microsoft.ML.OnnxRuntime`.  
     - **Details**: Load pre-trained ONNX models and run inference locally or via API endpoints.

5. **Chatbots and Conversational AI**  
   - **Use Cases**: Customer support bots, virtual assistants, interactive Q&A systems.  
   - **APIs**: Microsoft Bot Framework, Dialogflow, xAI’s Grok API (via https://x.ai/api).  
   - **Implementation Example**:  
     Using Microsoft Bot Framework:
     ```csharp
     using Microsoft.Bot.Connector;
     using Microsoft.Bot.Builder;

     var connector = new ConnectorClient(new Uri("your-bot-service-url"), "your-app-id", "your-app-password");
     var message = Activity.CreateMessageActivity();
     message.Text = "Hello, I'm your bot!";
     await connector.Conversations.SendToConversationAsync((Activity)message);
     ```
     - **NuGet Package**: `Microsoft.Bot.Connector`.  
     - **Details**: Integrate with channels like Teams or Slack, handling user inputs via HTTP requests.

6. **Generative AI for Content Creation**  
   - **Use Cases**: Generating images, code, or text; auto-generating UI mockups or reports.  
   - **APIs**: DALL·E (via OpenAI), Stable Diffusion, xAI’s Grok API.  
   - **Implementation Example**:  
     Generating an image with DALL·E (OpenAI API):
     ```csharp
     using OpenAI_API.Images;

     var openAi = new OpenAIAPI("your-api-key");
     var request = new ImageGenerationRequest
     {
         Prompt = "A futuristic cityscape",
         Size = ImageSize._1024x1024
     };
     var image = await openAi.ImageGenerations.CreateImageAsync(request);
     Console.WriteLine(image.Data[0].Url);
     ```
     - **NuGet Package**: `OpenAI_API`.  
     - **Details**: Handle API responses with image URLs or base64-encoded data.

### Implementation Patterns in C#/.NET
- **HTTP Clients**: Most AI APIs are REST-based, so `HttpClient` is commonly used:
  ```csharp
  using System.Net.Http;
  using System.Text.Json;

  var client = new HttpClient();
  client.DefaultRequestHeaders.Add("Authorization", "Bearer your-api-key");
  var response = await client.PostAsync("https://api.example.com/endpoint", new StringContent(JsonSerializer.Serialize(new { prompt = "Hello" }), Encoding.UTF8, "application/json"));
  var result = JsonSerializer.Deserialize<dynamic>(await response.Content.ReadAsStringAsync());
  ```
- **Async/Await**: AI API calls are typically async to handle network latency:
  ```csharp
  public async Task<string> CallAiApiAsync(string input)
  {
      var response = await _httpClient.PostAsync("https://api.example.com", new StringContent(input));
      return await response.Content.ReadAsStringAsync();
  }
  ```
- **Dependency Injection**: Register API clients in .NET’s DI container:
  ```csharp
  services.AddHttpClient("AiClient", client =>
  {
      client.BaseAddress = new Uri("https://api.example.com");
      client.DefaultRequestHeaders.Add("Authorization", "Bearer your-api-key");
  });
  ```
- **Error Handling**: Handle rate limits, timeouts, and API errors:
  ```csharp
  try
  {
      var response = await _httpClient.GetAsync("https://api.example.com");
      response.EnsureSuccessStatusCode();
  }
  catch (HttpRequestException ex)
  {
      Console.WriteLine($"API error: {ex.Message}");
  }
  ```

### Popular .NET Libraries for AI APIs
- **Microsoft.Azure.CognitiveServices**: For Azure’s AI services (vision, speech, text).  
- **OpenAI_API**: Unofficial SDK for OpenAI’s APIs.  
- **Google.Cloud.***: Google Cloud APIs (e.g., Vision, Speech).  
- **AWSSDK.***: AWS services like Rekognition or Transcribe.  
- **Microsoft.ML.OnnxRuntime**: For running ONNX models locally.  
- **Microsoft.Bot.Builder**: For building conversational AI.  

### Best Practices
- **Secure API Keys**: Store keys in `appsettings.json` or Azure Key Vault, not in code.  
- **Rate Limiting**: Implement retry logic with exponential backoff for API rate limits.  
- **Caching**: Cache frequent API responses using `IMemoryCache` to reduce costs and latency.  
- **Logging**: Use `ILogger` to log API interactions for debugging.  
- **Testing**: Mock API responses with `Moq` or use libraries like `WireMock.Net` for testing.
