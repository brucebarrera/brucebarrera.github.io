---
layout: post
title: Zero-shot image classification using C# and .NET with a hypothetical pre-trained CLIP model via ONNX Runtime
date: 2025-04-05 19:58:00 -0500
categories: [AI]
tags: [AI, csharp]
published: true
---

In this article I'll provide a C# implementation for zero-shot image classification using ONNX Runtime with a hypothetical CLIP model, along with unit tests. 

### Assumptions
- A pre-trained CLIP model (ONNX format) is available, with inputs for an image (preprocessed to 720x480 pixels, normalized)  and text embeddings for labels.
- The image is a 720x480 RGB image containing  giraffes, an elephants, and a jeep car in a savanna-like environment (see image below).
- ONNX Runtime is used for inference.
- The CLIP model outputs logits that can be converted to probabilities for each label

![a Savannah scene with Giraffes, Elephants and a Jeep car.](/assets/img/savannah_scene.png "Savannah Scene")

### C# Implementation

#### Project Setup
- **Dependencies**: Install `Microsoft.ML.OnnxRuntime` NuGet package for ONNX Runtime.
- **Image**: Assume a local image file `savanna_scene.jpg` (720x480 pixels) containing giraffes, elephants, and a jeep car.
- **Model**: Assume a pre-trained CLIP model file `clip.onnx` is available locally.

#### Code
Below is the C# implementation for zero-shot image classification.

```csharp
using Microsoft.ML.OnnxRuntime;
using Microsoft.ML.OnnxRuntime.Tensors;
using System;
using System.Drawing;
using System.Drawing.Imaging;
using System.Linq;
using System.Collections.Generic;

namespace ZeroShotImageClassification
{
    public class ZeroShotClassifier
    {
        private readonly InferenceSession _session;
        private readonly string[] _labels = { "giraffe", "elephant", "jeep car" };

        public ZeroShotClassifier(string modelPath)
        {
            _session = new InferenceSession(modelPath);
        }

        public Dictionary<string, float> ClassifyImage(string imagePath)
        {
            // Load and preprocess image
            float[] imageTensor = PreprocessImage(imagePath);

            // Prepare text embeddings for labels (hypothetical CLIP text encoder)
            float[] textEmbeddings = GetTextEmbeddings(_labels);

            // Run inference
            var inputs = new List<NamedOnnxValue>
            {
                NamedOnnxValue.CreateFromTensor("image", new DenseTensor<float>(imageTensor, new[] { 1, 3, 720, 480 })),
                NamedOnnxValue.CreateFromTensor("text", new DenseTensor<float>(textEmbeddings, new[] { _labels.Length, 512 }))
            };

            using var results = _session.Run(inputs);
            float[] logits = results.First().AsTensor<float>().ToArray();

            // Convert logits to probabilities using softmax
            var probabilities = Softmax(logits);

            // Map probabilities to labels
            var result = new Dictionary<string, float>();
            for (int i = 0; i < _labels.Length; i++)
            {
                result[_labels[i]] = probabilities[i];
            }

            return result;
        }

        private float[] PreprocessImage(string imagePath)
        {
            using var bitmap = new Bitmap(imagePath);
            if (bitmap.Width != 720 || bitmap.Height != 480)
            {
                throw new ArgumentException("Image must be 720x480 pixels.");
            }

            // Normalize image (assuming CLIP's normalization: mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])
            float[] tensor = new float[3 * 720 * 480];
            int index = 0;
            float[] mean = { 0.485f, 0.456f, 0.406f };
            float[] std = { 0.229f, 0.224f, 0.225f };

            for (int y = 0; y < 480; y++)
            {
                for (int x = 0; x < 720; x++)
                {
                    var pixel = bitmap.GetPixel(x, y);
                    tensor[index++] = (pixel.R / 255f - mean[0]) / std[0]; // R
                    tensor[index++] = (pixel.G / 255f - mean[1]) / std[1]; // G
                    tensor[index++] = (pixel.B / 255f - mean[2]) / std[2]; // B
                }
            }

            return tensor;
        }

        private float[] GetTextEmbeddings(string[] labels)
        {
            // Hypothetical: CLIP text encoder generates 512-dim embeddings for each label
            // In practice, you'd run the text through CLIP's text encoder
            // Here, we return a placeholder (randomized for demo)
            return new float[labels.Length * 512]; // 3 labels x 512 dims
        }

        private float[] Softmax(float[] logits)
        {
            float maxLogit = logits.Max();
            float sum = 0f;
            float[] probabilities = new float[logits.Length];

            for (int i = 0; i < logits.Length; i++)
            {
                probabilities[i] = (float)Math.Exp(logits[i] - maxLogit);
                sum += probabilities[i];
            }

            for (int i = 0; i < probabilities.Length; i++)
            {
                probabilities[i] /= sum;
            }

            return probabilities;
        }
    }
}
```

#### Main Program
```csharp
using System;

namespace ZeroShotImageClassification
{
    class Program
    {
        static void Main(string[] args)
        {
            try
            {
                var classifier = new ZeroShotClassifier("clip.onnx");
                var results = classifier.ClassifyImage("savanna_scene.jpg");

                Console.WriteLine("Classification Results:");
                foreach (var kvp in results)
                {
                    Console.WriteLine($"{kvp.Key}: {kvp.Value:F4}");
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Error: {ex.Message}");
            }
        }
    }
}
```

#### Unit Tests (Using xUnit)
```csharp
using System.Collections.Generic;
using System.IO;
using Xunit;

namespace ZeroShotImageClassification.Tests
{
    public class ZeroShotClassifierTests
    {
        [Fact]
        public void ClassifyImage_ValidImage_ReturnsProbabilities()
        {
            // Arrange
            var modelPath = "clip.onnx"; // Mock path
            var imagePath = "savanna_scene.jpg"; // Mock path
            var classifier = new ZeroShotClassifier(modelPath);

            // Act
            var results = classifier.ClassifyImage(imagePath);

            // Assert
            Assert.Equal(3, results.Count);
            Assert.Contains("giraffe", results.Keys);
            Assert.Contains("elephant", results.Keys);
            Assert.Contains("jeep car", results.Keys);
            Assert.All(results.Values, prob => Assert.InRange(prob, 0f, 1f));
            Assert.True(results.Values.Sum() >= 0.99 && results.Values.Sum() <= 1.01, "Probabilities should sum to ~1");
        }

        [Fact]
        public void ClassifyImage_InvalidImageSize_ThrowsException()
        {
            // Arrange
            var modelPath = "clip.onnx";
            var classifier = new ZeroShotClassifier(modelPath);
            var invalidImagePath = "invalid_image.jpg"; // Assume 100x100 image

            // Act & Assert
            Assert.Throws<ArgumentException>(() => classifier.ClassifyImage(invalidImagePath));
        }
    }
}
```

### Expected Output
Running the program with the hypothetical image `savanna_scene.jpg` might produce:

```
Classification Results:
giraffe: 0.4500
elephant: 0.3500
jeep car: 0.2000
```

These probabilities are hypothetical, assuming the model identifies the giraffe as the most prominent object, followed by the elephant, and the jeep car as less dominant in the scene.

### Notes
- **Model Availability**: The `clip.onnx` model is hypothetical. In practice, you'd need to export a pre-trained CLIP model (e.g., from PyTorch) to ONNX format.
- **Text Embeddings**: The `GetTextEmbeddings` method is a placeholder. In a real implementation, you'd use CLIP's text encoder to generate embeddings for the labels.
- **Image Preprocessing**: The preprocessing follows CLIP's standard normalization. Ensure the image is 720x480 pixels and normalized correctly.
- **Unit Tests**: The tests assume the model and image files exist. For real testing, you'd need to mock the ONNX model or use a test image.
- **Dependencies**: Ensure `Microsoft.ML.OnnxRuntime` is installed via NuGet (`dotnet add package Microsoft.ML.OnnxRuntime`).
- **Running Tests**: Use `dotnet test` to run the xUnit tests.
