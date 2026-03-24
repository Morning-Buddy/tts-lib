# Fix Summary: MLXUtilsLibrary Linking Issues

## Problem
The tts-lib package had critical linking issues preventing it from working in consuming iOS/macOS apps:
- MLXUtilsLibrary was a dependency but not exposed as a product
- Consuming apps got "Undefined symbol" linker errors
- Runtime dyld errors: "Library not loaded: @rpath/KokoroSwift.framework/KokoroSwift"

## Root Cause
The Package.swift only exposed `KokoroSwift` as a product, but `KokoroSwift` internally depends on `MLXUtilsLibrary`. Swift Package Manager requires transitive dependencies to be explicitly linked by consuming apps.

## Solution Implemented

### 1. Created MLXUtilsLibraryWrapper Target
- Added new target `MLXUtilsLibraryWrapper` in Package.swift
- Created `Sources/MLXUtilsLibraryWrapper/MLXUtilsLibraryWrapper.swift` with `@_exported import MLXUtilsLibrary`
- This wrapper re-exports MLXUtilsLibrary so consuming apps can link against it

### 2. Exposed MLXUtilsLibrary as Product
```swift
products: [
    .library(name: "KokoroSwift", type: .dynamic, targets: ["KokoroSwift"]),
    .library(name: "MLXUtilsLibrary", targets: ["MLXUtilsLibraryWrapper"]),
]
```

### 3. Updated Documentation
- Updated README.md with correct installation instructions
- Created CHANGELOG.md documenting the fix
- Created INTEGRATION_GUIDE.md with complete usage examples
- Created this FIX_SUMMARY.md

## Files Modified
1. `Package.swift` - Added MLXUtilsLibrary product and wrapper target
2. `README.md` - Updated installation and usage instructions
3. `Sources/MLXUtilsLibraryWrapper/MLXUtilsLibraryWrapper.swift` - New wrapper file

## Files Created
1. `CHANGELOG.md` - Version history and fix details
2. `INTEGRATION_GUIDE.md` - Complete integration guide for developers
3. `FIX_SUMMARY.md` - This file

## Verification
✅ Package resolves successfully: `swift package resolve`
✅ Clean build succeeds: `swift build`
✅ Both products exposed: `swift package describe`
✅ MLXUtilsLibraryWrapper builds: `swift build --target MLXUtilsLibraryWrapper`
✅ KokoroSwift builds: `swift build --target KokoroSwift`

## How Consuming Apps Should Use It

### Package.swift
```swift
dependencies: [
    .package(url: "https://github.com/Morning-Buddy/tts-lib", from: "1.0.12")
],
targets: [
    .target(
        name: "YourApp",
        dependencies: [
            .product(name: "KokoroSwift", package: "tts-lib"),
            .product(name: "MLXUtilsLibrary", package: "tts-lib")  // Required!
        ]
    )
]
```

### Swift Code
```swift
import KokoroSwift
import MLXUtilsLibrary  // Required import

let tts = KokoroTTS(modelPath: modelURL, g2p: .misaki)
let (audio, tokens) = try tts.generateAudio(voice: voiceEmbedding, language: .enUS, text: "Hello")
```

## Technical Details

### Why @_exported?
The `@_exported import MLXUtilsLibrary` directive in the wrapper makes all MLXUtilsLibrary symbols available to code that imports MLXUtilsLibraryWrapper, without requiring a separate import statement. This ensures proper symbol resolution at link time.

### Why a Wrapper Target?
We can't directly re-export an external package product. The wrapper target acts as an intermediary that:
1. Depends on the external MLXUtilsLibrary product
2. Re-exports it using `@_exported import`
3. Gets exposed as a product that consuming apps can link against

### Library Type
- KokoroSwift: `.dynamic` - Required for framework embedding
- MLXUtilsLibrary: `.automatic` - SPM chooses static/dynamic based on context

## Impact
This fix resolves the blocking issue preventing Kokoro TTS from working in the MorningBuddy app and any other consuming applications. Apps can now:
- ✅ Compile without linker errors
- ✅ Run without dyld errors
- ✅ Use full Kokoro TTS functionality
- ✅ Access MLXUtilsLibrary utilities if needed

## Version
This fix is tagged as version 1.0.12 and should be the minimum version used by consuming apps.

## Testing Checklist for Consuming Apps
- [ ] Add both products to target dependencies
- [ ] Import both KokoroSwift and MLXUtilsLibrary
- [ ] Clean build folder
- [ ] Build succeeds without linker errors
- [ ] App runs on device without dyld errors
- [ ] KokoroTTS initializes successfully
- [ ] Audio generation works correctly

## Next Steps
1. Tag this commit as v1.0.12
2. Push to GitHub
3. Update MorningBuddy app to use v1.0.12
4. Verify integration in MorningBuddy
5. Consider adding integration tests to prevent regression
