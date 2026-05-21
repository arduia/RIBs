# Android Libraries - Module Context Index

Quick reference for all Android RIBs library modules with compact context documentation.

## Core Modules

### [rib-base](rib-base/CONTEXT.md) 
Foundation module with core RIBs interfaces and base classes.
- **Provides**: Interactor, Router, Builder, Worker patterns
- **Defines**: Core contracts for all RIBs
- **See**: [CONTEXT.md](rib-base/CONTEXT.md) | [Full Research](../../docs/research/android/RIB_BASE.md)

### [rib-android](rib-android/CONTEXT.md)
Android-specific implementation with View hierarchy integration.
- **Provides**: ViewRouter, ViewInteractor, ViewPresenter
- **Handles**: Activity/Fragment lifecycle binding
- **See**: [CONTEXT.md](rib-android/CONTEXT.md) | [Full Research](../../docs/research/android/RIB_ANDROID.md)

### [rib-android-core](rib-android-core/CONTEXT.md)
Low-level Android integration and lifecycle utilities.
- **Provides**: Android lifecycle helpers
- **Handles**: Context and resource access
- **See**: [CONTEXT.md](rib-android-core/CONTEXT.md)

## Advanced Features

### [rib-android-compose](rib-android-compose/CONTEXT.md)
Jetpack Compose integration for declarative UI.
- **Provides**: ComposePresenter, state management
- **Handles**: Compose + RIBs state flow
- **See**: [CONTEXT.md](rib-android-compose/CONTEXT.md) | [Full Research](../../docs/research/android/RIB_ANDROID_COMPOSE.md)

### [rib-workflow](rib-workflow/CONTEXT.md)
State machine and multi-step workflow management.
- **Provides**: Workflow<T>, WorkflowRouter
- **Handles**: Sequential flows, conditional branching
- **See**: [CONTEXT.md](rib-workflow/CONTEXT.md) | [Full Research](../../docs/research/android/RIB_WORKFLOW.md)

### [rib-router-navigator](rib-router-navigator/CONTEXT.md)
Advanced navigation abstraction and back stack management.
- **Provides**: RouterNavigator, Destination pattern
- **Handles**: Tab navigation, modals, deep linking
- **See**: [CONTEXT.md](rib-router-navigator/CONTEXT.md) | [Full Research](../../docs/research/android/RIB_ROUTER_NAVIGATOR.md)

### [rib-screen-stack-base](rib-screen-stack-base/CONTEXT.md)
Foundation for screen stack-based navigation.
- **Provides**: ScreenStack, push/pop navigation
- **Handles**: Back stack management
- **See**: [CONTEXT.md](rib-screen-stack-base/CONTEXT.md)

## Testing & Utilities

### [rib-test](rib-test/CONTEXT.md)
Testing utilities and mock implementations.
- **Provides**: Test doubles, fixtures, observers
- **Handles**: Fast, isolated unit testing
- **See**: [CONTEXT.md](rib-test/CONTEXT.md) | [Full Research](../../docs/research/android/RIB_TEST.md)

### [rib-workflow-test](rib-workflow-test/CONTEXT.md)
Testing utilities for workflow state machines.
- **Provides**: Workflow test observers and runners
- **Handles**: Step and flow validation
- **See**: [CONTEXT.md](rib-workflow-test/CONTEXT.md)

### [rib-debug-utils](rib-debug-utils/CONTEXT.md)
Debugging and diagnostic utilities.
- **Provides**: Memory leak detection, tree inspection
- **Handles**: Development-time diagnostics
- **See**: [CONTEXT.md](rib-debug-utils/CONTEXT.md)

## Code Generation

### [rib-compiler-app](rib-compiler-app/CONTEXT.md)
Annotation processor for code generation.
- **Provides**: Builder and component generation
- **Handles**: Boilerplate reduction
- **See**: [CONTEXT.md](rib-compiler-app/CONTEXT.md)

### [rib-compiler-test](rib-compiler-test/CONTEXT.md)
Testing for code generation.
- **Provides**: Compilation testing utilities
- **Handles**: Generated code validation
- **See**: [CONTEXT.md](rib-compiler-test/CONTEXT.md)

## Module Dependency Map

```
┌─────────────────────────────────────┐
│    Application Code                 │
└──────────────┬──────────────────────┘
               │
     ┌─────────┴─────────┐
     │                   │
┌────▼────────┐   ┌──────▼────────┐
│ rib-android │   │ rib-workflow  │
│ rib-android-│   │               │
│ compose     │   └──────┬────────┘
└────┬────────┘          │
     │          ┌────────┴────────┐
     │          │                 │
┌────▼──────────▼────┐  ┌─────────▼──────┐
│  rib-android-core  │  │rib-router-nav  │
│  rib-screen-stack  │  │rib-screen-stack│
└────┬────────┬──────┘  └────────┬───────┘
     │        │                  │
     └────────┴──────┬───────────┘
                     │
            ┌────────▼────────┐
            │   rib-base      │
            └─────────────────┘

Testing/Debug Path:
┌────────────────────┐
│ Each module above  │
└────────┬───────────┘
         │
    ┌────▼──────────────┐
    │ rib-test          │ (all use)
    │ rib-workflow-test │ (workflows use)
    │ rib-debug-utils   │ (debug use)
    └───────────────────┘

Code Generation Path:
┌────────────────────┐
│ Source annotations │
└────────┬───────────┘
         │
    ┌────▼──────────────┐
    │ rib-compiler-app  │
    │ rib-compiler-test │
    └───────────────────┘
```

## Quick Selection Guide

### For a new UI RIB: Use
- ✅ [rib-base](rib-base/CONTEXT.md) - Core patterns
- ✅ [rib-android](rib-android/CONTEXT.md) - Android Views
- ✅ [rib-test](rib-test/CONTEXT.md) - Testing

### For Compose: Use
- ✅ [rib-base](rib-base/CONTEXT.md) - Core patterns
- ✅ [rib-android-compose](rib-android-compose/CONTEXT.md) - Compose support
- ✅ [rib-test](rib-test/CONTEXT.md) - Testing

### For Complex Navigation: Use
- ✅ [rib-router-navigator](rib-router-navigator/CONTEXT.md) - Advanced routing
- ✅ [rib-workflow](rib-workflow/CONTEXT.md) - Multi-step flows
- ✅ [rib-screen-stack-base](rib-screen-stack-base/CONTEXT.md) - Stack navigation

### For Multi-Step Flows: Use
- ✅ [rib-workflow](rib-workflow/CONTEXT.md) - Workflows
- ✅ [rib-workflow-test](rib-workflow-test/CONTEXT.md) - Testing workflows

### For Debugging: Use
- ✅ [rib-debug-utils](rib-debug-utils/CONTEXT.md) - Diagnostics
- ✅ [rib-test](rib-test/CONTEXT.md) - Unit testing

## Cross-Module Integration

Each module has CONTEXT.md documenting:
- Quick summary (1-2 sentences)
- Main purpose
- Key components
- Quick patterns with code examples
- Integration points with other modules
- Files to know
- Common tasks
- Link to full research documentation

## Full Documentation

For comprehensive details, see:
- **Research Docs**: [docs/research/android/](../../docs/research/android/)
- **CLAUDE.md**: [CLAUDE.md](../../CLAUDE.md) with task-specific guidance
- **Architecture Overview**: [ARCHITECTURE_OVERVIEW.md](../../docs/research/ARCHITECTURE_OVERVIEW.md)

## Accessing This Document

📍 **Direct Link**: `android/libraries/MODULES_CONTEXT.md`
📍 **From Project Root**: Navigate to `android/libraries/` directory
📍 **From Any Module**: Look for `CONTEXT.md` in module root

---

**Last Updated**: May 21, 2026
**Part of**: Mental Alignment Initiative - Context Compression
