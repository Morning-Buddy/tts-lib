# Integration Guide for tts-lib

This guide helps you integrate the Kokoro TTS library into your iOS/macOS app.

## Quick Start

### 1. Add Package Dependency

In your `Package.swift`:

```swift
dependencies: [
    .package(url: "https://github.com/Morning-Buddy/tts-lib", from: "1.0.12")
]
```

### 2. Link Both Products

**IMPORTANT**: You must link both products to avoid linker errors:

```swift
.target(
    name: "YourApp",
    dependencies: [
        .product(name: "KokoroSwift", package: "tts-lib"),
        .product(name: "MLXUtilsLibrary", package: "tts-lib")  // ← Required!
    ]
)
```

### 3. Import in Your Code

```swift
import KokoroSwift
import MLXUtilsLibrary  // Required for proper linking
```

## Why Both Products Are Required

The `KokoroSwift` framework internally uses `MLXUtilsLibrary` for utility functions. Due to how Swift Package Manager handles transitive dependencies, consuming apps must explicitly link against `MLXUtilsLibrary` to resolve all symbols at link time.

### What Happens If You Don't Link MLXUtilsLibrary?

You'll encounter one or both of these errors:

1. **Linker Error** (compile time):
   ```
   Undefined symbol: MLXUtilsLibrary.NpyzReader
   ```

2. **Runtime Error** (dyld):
   ```
   dyld: Library not loaded: @rpath/KokoroSwift.framework/KokoroSwift
   Reason: image not found
   ```

## Complete Example

```swift
import SwiftUI
import KokoroSwift
import MLXUtilsLibrary
import MLX

struct ContentView: View {
    @State private var isGenerating = false
    
    func generateSpeech() {
        isGenerating = true
        
        Task {
            do {
                // Initialize TTS engine
                let modelPath = Bundle.main.url(forResource: "kokoro-v0_19", withExtension: "safetensors")!
                let tts = KokoroTTS(modelPath: modelPath, g2p: .misaki)
                
                // Load voice embedding (see KokoroTestApp for details)
                let voiceURL = Bundle.main.url(forResource: "voice_af_bella", withExtension: "npz")!
                let voiceData = try Data(contentsOf: voiceURL)
                let voiceEmbedding = // ... load from NPZ file
                
                // Generate audio
                let text = "Hello, this is Kokoro TTS speaking."
                let (audioSamples, tokens) = try tts.generateAudio(
                    voice: voiceEmbedding,
                    language: .enUS,
                    text: text,
                    speed: 1.0
                )
                
                // Play audio (convert [Float] to audio buffer)
                // ... your audio playback code here
                
                print("Generated \(audioSamples.count) audio samples")
                if let tokens = tokens {
                    print("Token timestamps: \(tokens.count) tokens")
                }
                
            } catch {
                print("TTS Error: \(error)")
            }
            
            isGenerating = false
        }
    }
    
    var body: some View {
        Button("Generate Speech") {
            generateSpeech()
        }
        .disabled(isGenerating)
    }
}
```

## Xcode Project Integration

If you're using Xcode (not SPM directly):

1. Go to your project settings
2. Select your target
3. Go to "Frameworks, Libraries, and Embedded Content"
4. Click "+" and add both:
   - KokoroSwift
   - MLXUtilsLibrary

Or use File → Add Package Dependencies and add the package URL, then select both products when prompted.

## Requirements

- iOS 18.0+ or macOS 15.0+
- Xcode 16.0+
- Swift 6.2+
- MLX-compatible Apple Silicon device (for optimal performance)

## Troubleshooting

### "Undefined symbol" errors
- Make sure you've added both `KokoroSwift` AND `MLXUtilsLibrary` to your target dependencies
- Clean build folder (Cmd+Shift+K) and rebuild

### Runtime dyld errors
- Verify both products are linked in your target settings
- Check that the package version is 1.0.12 or later

### Build errors
- Ensure you're targeting iOS 18+ or macOS 15+
- Update to the latest Xcode version
- Run `swift package update` to refresh dependencies

## Additional Resources

- [KokoroTestApp](https://github.com/mlalma/KokoroTestApp) - Complete example application
- [MLX Swift Documentation](https://github.com/ml-explore/mlx-swift)
- [Original Kokoro TTS](https://github.com/hexgrad/kokoro) - PyTorch implementation

## Support

For issues specific to this package, please open an issue on the [tts-lib repository](https://github.com/Morning-Buddy/tts-lib/issues).
