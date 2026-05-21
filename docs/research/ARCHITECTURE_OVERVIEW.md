# RIBs Architecture Overview

## Project Structure

RIBs is a cross-platform architecture framework with implementations for iOS and Android. The project is organized as follows:

```
RIBs/
├── android/
│   ├── libraries/        # Core Android framework modules
│   ├── tutorials/        # Hands-on Android tutorials
│   └── tooling/          # IDE plugins and utilities
├── ios/
│   ├── RIBs/            # Core iOS framework
│   ├── RIBsTests/       # iOS framework tests
│   ├── tooling/         # iOS development tools
│   └── tutorials/       # Hands-on iOS tutorials
└── docs/
    └── research/        # Module research and documentation
```

## Core Concepts

### What is a RIB?

RIB stands for **Router, Interactor, and Builder** - the three core components of the architecture:

1. **Router**: Manages navigation and child RIB lifecycle
2. **Interactor**: Handles business logic and state management
3. **Builder**: Constructs RIB components and injects dependencies

### Key Principles

1. **Business logic drives app structure** - The app hierarchy is driven by business logic, not view tree
2. **Independent layers** - Business logic tree can be deep while view tree remains shallow
3. **Testability** - Classes have distinct responsibilities and can be unit tested in isolation
4. **Scalability** - Architecture scales to hundreds of engineers and hundreds of RIBs
5. **Cross-platform** - iOS and Android teams share similar architecture

## Module Organization

### Android Modules

#### Core Modules (rib-base)
Base classes and interfaces for RIBs:
- `Rib`: Global RIBs configuration
- `Interactor`: Business logic component
- `Router`: Navigation and lifecycle management
- `Builder`: Dependency injection and RIB construction
- `Worker`: Background task execution
- `RibEvent`: Lifecycle event tracking

#### Android-Specific Modules
- `rib-android`: Android UI framework integration
- `rib-android-core`: Core Android implementations
- `rib-android-compose`: Jetpack Compose support
- `rib-router-navigator`: Advanced routing navigation
- `rib-screen-stack-base`: Screen stack management

#### Utility Modules
- `rib-workflow`: State management and reactive patterns
- `rib-test`: Testing utilities and mocks
- `rib-debug-utils`: Debugging and memory leak detection
- `rib-compiler-*`: Annotation processing for code generation

### iOS Modules

#### RIBs Framework
Main iOS implementation with Rx integration for reactive programming.

## Dependency Flow

```
Application Code
       ↓
rib-android / RIBs (iOS)
       ↓
rib-android-core (Android) / Core iOS Framework
       ↓
rib-base / Base Interfaces
```

## Integration Points

### Dependency Injection
- Components define dependencies needed from parent RIBs
- Builder pattern ensures hierarchical scoping
- Parent components fulfill child dependencies

### Reactive Streams
- RxJava (Android) / RxSwift (iOS) for reactive patterns
- Observable streams for inter-component communication
- Subject-based event patterns for state changes

### Lifecycle Management
- RibEvent system tracks component lifecycle
- Automatic resource cleanup on RIB detachment
- Worker scope binding to RIB lifecycle

## Next Steps

Refer to module-specific documentation for detailed information on each component:
- [rib-base Research](android/RIB_BASE.md)
- [rib-android Research](android/RIB_ANDROID.md)
- [iOS RIBs Research](ios/RIBs_iOS.md)
