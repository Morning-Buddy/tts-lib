# Changelog

All notable changes to this project will be documented in this file.

## [1.0.12] - 2026-03-24

### Fixed
- **CRITICAL**: Fixed MLXUtilsLibrary linking issues that caused "Undefined symbol" linker errors
  - Added MLXUtilsLibrary as an exposed product in Package.swift
  - Created MLXUtilsLibraryWrapper target to re-export MLXUtilsLibrary
  - Consuming apps can now properly link against MLXUtilsLibrary
  
### Changed
- Updated README.md with correct installation instructions
  - Both KokoroSwift and MLXUtilsLibrary must be added to target dependencies
  - Added MLXUtilsLibrary import requirement in usage examples
  - Updated package URL to reflect new repository location

### Technical Details
The package now exposes two products:
1. `KokoroSwift` - The main TTS engine (dynamic library)
2. `MLXUtilsLibrary` - Utility library wrapper (required for linking)

This fixes runtime dyld errors and linker failures when consuming apps try to use the package.

## [1.0.11] - Previous Release
- Previous version with linking issues

## Earlier Versions
See git history for details on earlier releases.
