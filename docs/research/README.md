# RIBs Research Documentation

This directory contains comprehensive research documentation for each module in the RIBs project. These documents provide detailed context about module purposes, core components, architectural patterns, and integration points.

## Document Organization

### Overview Documents

- **[ARCHITECTURE_OVERVIEW.md](ARCHITECTURE_OVERVIEW.md)** - High-level architecture overview, core concepts, and module organization

### Android Module Research

#### Core Modules
- **[RIB_BASE.md](android/RIB_BASE.md)** - Foundation classes and interfaces (`rib-base`)
  - Interactor, Router, Builder patterns
  - Lifecycle management
  - Worker system
  - Dependency injection

#### Android Integration
- **[RIB_ANDROID.md](android/RIB_ANDROID.md)** - Android UI framework integration (`rib-android`)
  - ViewRouter and ViewInteractor
  - ViewPresenter patterns
  - Activity/Fragment lifecycle
  - View hierarchy synchronization
  - Memory leak detection

#### Modern Android Development
- **[RIB_ANDROID_COMPOSE.md](android/RIB_ANDROID_COMPOSE.md)** - Jetpack Compose support (`rib-android-compose`)
  - Compose + RIBs integration
  - ComposePresenter pattern
  - State management with Compose
  - Unidirectional data flow
  - Lifecycle integration

#### Advanced Features
- **[RIB_WORKFLOW.md](android/RIB_WORKFLOW.md)** - Workflow and state machine management (`rib-workflow`)
  - Workflow pattern for multi-step flows
  - State machine implementation
  - Conditional branching
  - Error and cancellation handling
  - Complex user flows

- **[RIB_ROUTER_NAVIGATOR.md](android/RIB_ROUTER_NAVIGATOR.md)** - Advanced navigation (`rib-router-navigator`)
  - Navigation abstraction
  - Back stack management
  - Tab navigation
  - Modal navigation
  - Deep linking
  - Screen stack patterns

#### Testing & Utilities
- **[RIB_TEST.md](android/RIB_TEST.md)** - Testing framework (`rib-test`)
  - Unit testing patterns
  - Mock/stub implementations
  - Integration testing
  - Rx testing utilities
  - Lifecycle testing

### iOS Framework Research

- **[RIBs_iOS.md](ios/RIBs_iOS.md)** - iOS framework core (`RIBs`)
  - Swift protocol-based architecture
  - RxSwift integration
  - ViewControllable pattern
  - UIKit and SwiftUI support
  - Component and DI patterns
  - Navigation patterns
  - Lifecycle management
  - Testing strategies

## Quick Reference by Purpose

### Understanding Core Architecture
Start with [ARCHITECTURE_OVERVIEW.md](ARCHITECTURE_OVERVIEW.md) for the big picture.

### Building Android Apps
1. [RIB_BASE.md](android/RIB_BASE.md) - Understand core concepts
2. [RIB_ANDROID.md](android/RIB_ANDROID.md) - Implement with views
3. Choose based on UI framework:
   - [RIB_ANDROID_COMPOSE.md](android/RIB_ANDROID_COMPOSE.md) for Compose
   - [RIB_ANDROID.md](android/RIB_ANDROID.md) for traditional Views

### Building iOS Apps
- [RIBs_iOS.md](ios/RIBs_iOS.md) - Complete iOS implementation guide

### Complex Navigation
- [RIB_WORKFLOW.md](android/RIB_WORKFLOW.md) - Multi-step flows
- [RIB_ROUTER_NAVIGATOR.md](android/RIB_ROUTER_NAVIGATOR.md) - Advanced routing

### Testing
- [RIB_TEST.md](android/RIB_TEST.md) - Android testing patterns
- See iOS document for iOS testing

## Module Dependency Map

```
┌─────────────────────────────────────┐
│       Application Code              │
└──────────────┬──────────────────────┘
               │
      ┌────────┴────────┐
      │                 │
┌─────▼──────┐    ┌─────▼──────────┐
│ rib-android│    │  RIBs (iOS)    │
└─────┬──────┘    └────────┬───────┘
      │                    │
┌─────▼──────────┐    ┌────▼────────────┐
│ rib-android-*  │    │ Swift Protocols │
│ rib-workflow   │    │ RxSwift         │
│ rib-router-nav │    └─────────────────┘
└─────┬──────────┘
      │
┌─────▼──────────┐
│   rib-base     │
└────────────────┘
```

## Key Concepts Index

### Architecture Patterns
- **MVC vs RIBs**: [ARCHITECTURE_OVERVIEW.md](ARCHITECTURE_OVERVIEW.md#core-concepts)
- **Dependency Injection**: [RIB_BASE.md](android/RIB_BASE.md#key-architecture-patterns)
- **Lifecycle Binding**: [RIB_BASE.md](android/RIB_BASE.md#2-lifecycle-binding)
- **Hierarchical Scoping**: [RIB_BASE.md](android/RIB_BASE.md#3-hierarchical-scoping)

### Component Patterns
- **Interactor Pattern**: [RIB_BASE.md](android/RIB_BASE.md#2-interactor--ribinteractor)
- **Router Pattern**: [RIB_BASE.md](android/RIB_BASE.md#3-router--basicrouter)
- **Builder Pattern**: [RIB_BASE.md](android/RIB_BASE.md#4-builder--ribbuilder)
- **Worker Pattern**: [RIB_BASE.md](android/RIB_BASE.md#6-worker-system)

### UI Integration
- **Android Views**: [RIB_ANDROID.md](android/RIB_ANDROID.md)
- **Jetpack Compose**: [RIB_ANDROID_COMPOSE.md](android/RIB_ANDROID_COMPOSE.md)
- **iOS UIKit/SwiftUI**: [RIBs_iOS.md](ios/RIBs_iOS.md)

### Advanced Topics
- **State Machines**: [RIB_WORKFLOW.md](android/RIB_WORKFLOW.md#core-components)
- **Navigation Stacks**: [RIB_ROUTER_NAVIGATOR.md](android/RIB_ROUTER_NAVIGATOR.md#back-stack-management)
- **Deep Linking**: [RIB_ROUTER_NAVIGATOR.md](android/RIB_ROUTER_NAVIGATOR.md#3-deep-linking)

### Testing
- **Unit Testing**: [RIB_TEST.md](android/RIB_TEST.md#testing-patterns)
- **Integration Testing**: [RIB_TEST.md](android/RIB_TEST.md#4-integration-testing)
- **Lifecycle Testing**: [RIB_TEST.md](android/RIB_TEST.md#4-lifecycle-management)

## Using These Documents with Agents

These research documents are designed to provide context for AI agents and developers:

1. **Code Generation**: Agents can reference these docs when generating new RIBs
2. **Code Review**: Use as standards for reviewing RIBs implementations
3. **Architecture Decisions**: Document rationale for architectural choices
4. **Onboarding**: New team members can understand the architecture
5. **Maintenance**: Reference when fixing bugs or refactoring

### Agent Context System
See **[AGENT_CONTEXT_SYSTEM.md](AGENT_CONTEXT_SYSTEM.md)** for detailed explanation of:
- How agents discover and load research documentation
- Workflow examples for common tasks
- Quality assurance checklist
- Continuous improvement process

## Version Information

- **RIBs Version**: 0.12.0 (Android), 0.9+ (iOS)
- **Research Document Date**: May 2026
- **Documentation Scope**: Complete architecture overview with examples

## Cross-References

### Android-iOS Comparison Tables
See module-specific documents for platform comparison tables.

### Example Implementations
- Android examples in `android/tutorials/`
- iOS examples in `ios/tutorials/`
- Code samples embedded in research documents

## Contributing to Research Docs

When updating RIBs or adding new modules:
1. Update relevant research document
2. Add cross-references in this README
3. Include code examples
4. Document integration points

## Related Resources

- Official RIBs Wiki: https://github.com/uber/RIBs/wiki
- Android Docs: `android/README.md` files in each module
- iOS Docs: `ios/tutorials/` examples
- Blog Posts: See main project README for engineering blog links

---

**Last Updated**: May 21, 2026
**Created for**: Mental Alignment Context Compression
