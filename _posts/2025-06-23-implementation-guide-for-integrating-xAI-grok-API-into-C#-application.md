---
layout: post
title: Implementation guide for integrating xAI's Grok API into a C#/.NET application
date: 2025-06-19 19:58:00 -0500
categories: [AI]
tags: [Grok, C#, OpenAI, API, AI, xAI]
published: true
---

In this article I'll show an example of an implementation for integrating xAI's Grok API into a C#/.NET application, focusing on a practical use case: generating text completions using the Grok API. This article includes setup instructions, a sample C# code snippet, and error handling, leveraging the API's compatibility with OpenAI's SDK format. The example assumes you have an API key from xAI’s console (https://console.x.ai) and uses the `HttpClient` for REST API calls, as there’s no official C# SDK for Grok, but its REST API is straightforward.

## Code Implementation
```csharp
using System;
using System.Net.Http;
using System.Text;
using System.Text.Json;
using System.Threading.Tasks;

namespace GrokApiExample
{
    public class GrokApiClient
    {
        private readonly HttpClient _httpClient;
        private readonly string _apiKey;

        public GrokApiClient(string apiKey)
        {
            _apiKey = apiKey ?? throw new ArgumentNullException(nameof(apiKey));
            _httpClient = new HttpClient
            {
                BaseAddress = new Uri("https://api.x.ai/v1")
            };
            _httpClient.DefaultRequestHeaders.Add("Authorization", $"Bearer {_apiKey}");
            _httpClient.DefaultRequestHeaders.Add("Content-Type", "application/json");
        }

        public async Task<string> GetTextCompletionAsync(string prompt, string model = "grok-beta", int maxTokens = 100)
        {
            try
            {
                var requestBody = new
                {
                    model,
                    prompt,
                    max_tokens = maxTokens,
                    temperature = 0.7
                };

                var content = new StringContent(JsonSerializer.Serialize(requestBody), Encoding.UTF8, "application/json");
                var response = await _httpClient.PostAsync("/completions", content);
                
                response.EnsureSuccessStatusCode();
                
                var responseBody = await response.Content.ReadAsStringAsync();
                var result = JsonSerializer.Deserialize<GrokResponse>(responseBody);
                
                return result?.Choices?[0]?.Text ?? throw new Exception("No completion text returned");
            }
            catch (HttpRequestException ex) when (ex.StatusCode == System.Net.HttpStatusCode.Unauthorized)
            {
                throw new Exception("Authentication failed: Invalid API key", ex);
            }
            catch (HttpRequestException ex) when (ex.StatusCode == System.Net.HttpStatusCode.TooManyRequests)
            {
                throw new Exception("Rate limit exceeded: Try again later", ex);
            }
            catch (Exception ex)
            {
                throw new Exception($"API call failed: {ex.Message}", ex);
            }
        }

        private class GrokResponse
        {
            public Choice[] Choices { get; set; }
        }

        private class Choice
        {
            public string Text { get; set; }
        }
    }

    class Program
    {
        static async Task Main(string[] args)
        {
            // Replace with your xAI API key from https://console.x.ai
            string apiKey = Environment.GetEnvironmentVariable("XAI_API_KEY") ?? "your-api-key-here";
            var grokClient = new GrokApiClient(apiKey);

            try
            {
                string prompt = "Write a short description of a futuristic city.";
                string result = await grokClient.GetTextCompletionAsync(prompt);
                Console.WriteLine("Grok Response:");
                Console.WriteLine(result);
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Error: {ex.Message}");
            }
        }
    }
}
```

### Implementation Guide
1. **Obtain API Key**: Sign up at https://console.x.ai, navigate to "API Keys," and generate a key. Store it securely (e.g., in `appsettings.json` or as an environment variable).
2. **Set Up Project**:
   - Create a .NET Console App (e.g., `.NET 8.0`).
   - Install NuGet package: `System.Text.Json` (usually included in .NET Core).
   - Alternatively, use `OpenAI` NuGet package if you prefer OpenAI-compatible SDKs, adjusting the base URL to `https://api.x.ai/v1`.
3. **Code Explanation**:
   - The `GrokApiClient` class encapsulates API interactions, initializing `HttpClient` with the Grok API base URL and your API key.
   - The `GetTextCompletionAsync` method sends a POST request to `/completions`, passing a JSON payload with the model (`grok-beta`), prompt, and parameters like `max_tokens`.
   - Error handling covers common issues: 401 (invalid key), 429 (rate limit), and generic errors.
   - The response is deserialized into a simple `GrokResponse` class to extract the generated text.
4. **Running the Example**:
   - Replace `"your-api-key-here"` with your actual API key or set the `XAI_API_KEY` environment variable.
   - Run the program to send a prompt ("Write a short description of a futuristic city") and display the response.
5. **Improvements**:
   - **Secure API Key**: Use `IConfiguration` or Azure Key Vault for key management.
   - **Rate Limiting**: Respect the API’s limits (e.g., 1 request/second, 60 or 1200/hour depending on the model).[](https://www.merge.dev/blog/grok-api-key)
   - **Logging**: Add `ILogger` for debugging API calls.
   - **Dependency Injection**: Register `GrokApiClient` in `IServiceCollection` for larger applications.

### Notes
- The Grok API is compatible with OpenAI’s SDK format, so you can use `OpenAI_API` NuGet package by setting the endpoint to `https://api.x.ai/v1`.[](https://x.ai/news/api)
- As of June 2025, `grok-beta` is available, with `grok-3` accessible but not fully rolled out for all features.[](https://x.ai/news/api)
- For advanced use cases (e.g., function calling or multimodal inputs), extend the request payload per xAI’s documentation (https://docs.x.ai).[](https://docs.x.ai/docs/api-reference)

This example provides a foundation for integrating Grok’s text generation into your .NET applications. For further details, refer to xAI’s API docs at https://docs.x.ai.
