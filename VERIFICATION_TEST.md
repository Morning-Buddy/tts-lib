# Verification Test for tts-lib Fix

This document provides steps to verify the MLXUtilsLibrary linking fix works correctly.

## Quick Verification (Already Done)

✅ Package resolves: `swift package resolve`
✅ Package builds: `swift build`
✅ Both products exposed: `swift package describe`
✅ Wrapper target builds: `swift build --target MLXUtilsLibraryWrapper`
✅ Main target builds: `swift build --target KokoroSwift`

## Integration Test (For Consuming Apps)

To test this fix in a consuming app, create a test project:

### 1. Create Test Package

```bash
mkdir TestKokoroIntegration
cd TestKokoroIntegration
swift package init --type executable
```

### 2. Update Package.swift

```swift
// swift-tools-version: 5.9
import PackageDescription

let package = Package(
    name: "TestKokoroIntegration",
    platforms: [.iOS(.v18), .macOS(.v15)],
    dependencies: [
        .package(url: "https://github.com/Morning-Buddy/tts-lib", from: "1.0.12")
    ],
    targets: [
        .executableTarget(
            name: "TestKokoroIntegration",
            dependencies: [
                .product(name: "KokoroSwift", package: "tts-lib"),
                .product(name: "MLXUtilsLibrary", package: "tts-lib")
            ]
        )
    ]
)
```

### 3. Update main.swift

```swift
import Foundation
import KokoroSwift
import MLXUtilsLibrary

print("✅ Successfully imported KokoroSwift")
print("✅ Successfully imported MLXUtilsLibrary")

// Test that KokoroTTS class is accessible
print("✅ KokoroTTS class is available: \(KokoroTTS.self)")

print("\n🎉 All imports successful! The linking fix works correctly.")
```

### 4. Build and Run

```bash
swift build
swift run
```

### Expected Output

```
✅ Successfully imported KokoroSwift
✅ Successfully imported MLXUtilsLibrary
✅ KokoroTTS class is available: KokoroTTS
🎉 All imports successful! The linking fix works correctly.
```

## Xcode Project Test

### 1. Create New iOS App
- Open Xcode
- Create new iOS App project
- Set minimum deployment to iOS 18.0

### 2. Add Package Dependency
- File → Add Package Dependencies
- Enter: `https://github.com/Morning-Buddy/tts-lib`
- Select version 1.0.12 or later
- When prompted, select BOTH products:
  - ✅ KokoroSwift
  - ✅ MLXUtilsLibrary

### 3. Test Import in ContentView.swift

```swift
import SwiftUI
import KokoroSwift
import MLXUtilsLibrary

struct ContentView: View {
    var body: some View {
        VStack {
            Text("Kokoro TTS Integration Test")
            Text("✅ Imports successful")
        }
        .onAppear {
            print("KokoroTTS available: \(KokoroTTS.self)")
        }
    }
}
```

### 4. Build and Run
- Select a simulator or device
- Build (Cmd+B)
- Run (Cmd+R)

### Expected Results
- ✅ Build succeeds without linker errors
- ✅ App launches without dyld errors
- ✅ Console shows "KokoroTTS available: KokoroTTS"

## Common Issues and Solutions

### Issue: "Undefined symbol: MLXUtilsLibrary.NpyzReader"
**Solution**: Add MLXUtilsLibrary to target dependencies

### Issue: "Library not loaded: @rpath/KokoroSwift.framework"
**Solution**: Ensure both products are linked in target settings

### Issue: "Module 'MLXUtilsLibrary' not found"
**Solution**: Import MLXUtilsLibrary in your Swift files

### Issue: Build succeeds but runtime crash
**Solution**: Check that both frameworks are embedded in app bundle

## Regression Test

To prevent this issue from recurring, add this to your CI/CD:

```bash
#!/bin/bash
# test-integration.sh

set -e

echo "Testing tts-lib integration..."

# Create temp test package
mkdir -p /tmp/test-kokoro
cd /tmp/test-kokoro

cat > Package.swift << 'EOF'
// swift-tools-version: 5.9
import PackageDescription

let package = Package(
    name: "TestIntegration",
    platforms: [.macOS(.v15)],
    dependencies: [
        .package(path: "../../")  // Local path to tts-lib
    ],
    targets: [
        .executableTarget(
            name: "TestIntegration",
            dependencies: [
                .product(name: "KokoroSwift", package: "tts-lib"),
                .product(name: "MLXUtilsLibrary", package: "tts-lib")
            ]
        )
    ]
)
EOF

cat > Sources/main.swift << 'EOF'
import KokoroSwift
import MLXUtilsLibrary
print("✅ Integration test passed")
EOF

swift build
swift run

echo "✅ All integration tests passed!"
```

## Performance Verification

After confirming the fix works, verify performance hasn't regressed:

```swift
import KokoroSwift
import MLXUtilsLibrary
import Foundation

let start = Date()
// Initialize TTS and generate audio
let elapsed = Date().timeIntervalSince(start)
print("Generation time: \(elapsed)s")
// Should still be ~3.3x faster than real-time
```

## Checklist for Release

- [x] Package.swift updated with MLXUtilsLibrary product
- [x] MLXUtilsLibraryWrapper target created
- [x] README.md updated with correct instructions
- [x] CHANGELOG.md created
- [x] INTEGRATION_GUIDE.md created
- [x] Package builds successfully
- [x] Both products exposed correctly
- [ ] Tag as v1.0.12
- [ ] Push to GitHub
- [ ] Test in MorningBuddy app
- [ ] Verify on physical device
- [ ] Update app to use new version

## Success Criteria

✅ Package resolves without errors
✅ Package builds without errors
✅ Both products visible in `swift package describe`
✅ Test consumer app builds successfully
✅ Test consumer app runs without dyld errors
✅ KokoroTTS functionality works as expected
✅ No performance regression
✅ Documentation is clear and complete
