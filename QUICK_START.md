# Quick Start Guide

Get Kokoro TTS working in your app in 3 steps.

## Step 1: Add Package

In your `Package.swift`:

```swift
dependencies: [
    .package(url: "https://github.com/Morning-Buddy/tts-lib", from: "1.0.12")
]
```

## Step 2: Link Both Products

**CRITICAL**: Link BOTH products to avoid errors:

```swift
.target(
    name: "YourApp",
    dependencies: [
        .product(name: "KokoroSwift", package: "tts-lib"),
        .product(name: "MLXUtilsLibrary", package: "tts-lib")  // Don't forget!
    ]
)
```

## Step 3: Import and Use

```swift
import KokoroSwift
import MLXUtilsLibrary  // Required

// Initialize
let tts = KokoroTTS(modelPath: modelURL, g2p: .misaki)

// Generate audio
let (audio, tokens) = try tts.generateAudio(
    voice: voiceEmbedding,
    language: .enUS,
    text: "Hello world",
    speed: 1.0
)
```

## Requirements

- iOS 18.0+ or macOS 15.0+
- Xcode 16.0+
- Swift 6.2+

## Common Mistakes

❌ Forgetting to link MLXUtilsLibrary
❌ Not importing MLXUtilsLibrary
❌ Using old package URL (mlalma/kokoro-ios)
❌ Using version < 1.0.12

## Need Help?

- Full guide: [INTEGRATION_GUIDE.md](INTEGRATION_GUIDE.md)
- Example app: [KokoroTestApp](https://github.com/mlalma/KokoroTestApp)
- Issues: [GitHub Issues](https://github.com/Morning-Buddy/tts-lib/issues)

## That's It!

You're ready to generate high-quality speech with Kokoro TTS. 🎉
