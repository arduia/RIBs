# CLAUDE.md - Project Context & Agent Instructions

## Project Overview

This is the **RIBs** (Router, Interactor, Builder) framework - a cross-platform mobile architecture used at Uber for building scalable iOS and Android applications.

**Repository**: https://github.com/arduia/RIBs  
**Primary Branch**: main  
**Development Branches**: Feature branches with descriptive names

## Mental Alignment & Context Compression

This project uses **mental alignment** - a technique to ensure AI agents have complete architectural context when working on code. Research documentation has been created for every major module to provide comprehensive understanding.

### Key Concept
Before implementing features or fixing bugs, agents should:
1. **Read the relevant research documentation** from `docs/research/`
2. **Understand the architectural patterns** specific to the task
3. **Follow established patterns** for consistency
4. **Reference examples** in the research docs

## Research Documentation

Complete module research is available in `docs/research/`:

### Start Here
- **[ARCHITECTURE_OVERVIEW.md](docs/research/ARCHITECTURE_OVERVIEW.md)** - System architecture, core concepts
- **[docs/research/README.md](docs/research/README.md)** - Index of all research documents

### Android Modules
- **[RIB_BASE.md](docs/research/android/RIB_BASE.md)** - Core interfaces, Interactor/Router/Builder patterns
- **[RIB_ANDROID.md](docs/research/android/RIB_ANDROID.md)** - Android UI integration, ViewRouter, ViewInteractor
- **[RIB_ANDROID_COMPOSE.md](docs/research/android/RIB_ANDROID_COMPOSE.md)** - Jetpack Compose integration
- **[RIB_WORKFLOW.md](docs/research/android/RIB_WORKFLOW.md)** - Workflow state machines, multi-step flows
- **[RIB_ROUTER_NAVIGATOR.md](docs/research/android/RIB_ROUTER_NAVIGATOR.md)** - Advanced navigation, back stacks
- **[RIB_TEST.md](docs/research/android/RIB_TEST.md)** - Testing patterns and utilities

### iOS Framework
- **[RIBs_iOS.md](docs/research/ios/RIBs_iOS.md)** - Swift implementation, RxSwift, protocols

### Cross-Platform
- **[TESTING_GUIDE.md](docs/research/TESTING_GUIDE.md)** - Unit/integration testing strategies

## When to Use Research Docs

### Always Read Before:
- **Creating a new RIB**: Read ARCHITECTURE_OVERVIEW + module-specific docs
- **Working on Android features**: Read RIB_BASE + RIB_ANDROID + RIB_TEST
- **Working on iOS features**: Read RIBs_iOS + relevant module docs
- **Implementing navigation**: Read RIB_ROUTER_NAVIGATOR + RIB_WORKFLOW
- **Setting up testing**: Read TESTING_GUIDE + RIB_TEST
- **Working with Compose**: Read RIB_ANDROID_COMPOSE + RIB_ANDROID

### Task-Specific Guidance

#### New RIB Implementation
```
1. Read: ARCHITECTURE_OVERVIEW.md
2. Read: RIB_BASE.md (understand core patterns)
3. Read: RIB_ANDROID.md or RIBs_iOS.md (platform-specific)
4. Read: RIB_TEST.md (understand testing approach)
5. Implement following the patterns
6. Create tests based on TESTING_GUIDE.md
```

#### Bug Fixes
```
1. Read: ARCHITECTURE_OVERVIEW.md (quick refresh)
2. Read: Relevant module doc (e.g., RIB_ANDROID.md)
3. Check TESTING_GUIDE.md for test patterns
4. Fix the bug while maintaining architecture patterns
5. Add regression tests
```

#### Feature Addition
```
1. Determine which module is affected
2. Read that module's research document
3. Understand integration points from ARCHITECTURE_OVERVIEW.md
4. Implement following established patterns
5. Test thoroughly using TESTING_GUIDE.md patterns
```

#### Navigation Features
```
1. Read: RIB_ROUTER_NAVIGATOR.md
2. Read: RIB_WORKFLOW.md (for complex flows)
3. Check examples in the research docs
4. Implement using established patterns
```

## Code Examples in Research Docs

All research documents include code examples in:
- **Kotlin** for Android
- **Swift** for iOS
- **Kotlin + Mockito** for Android testing
- **Swift + XCTest** for iOS testing

Use these examples as templates when implementing similar functionality.

## Architecture Patterns to Follow

### Core Patterns (From RIB_BASE.md)
1. **Builder Pattern**: Constructor dependency injection
2. **Lifecycle Binding**: Tie resource cleanup to RIB lifecycle
3. **Hierarchical Scoping**: Parent provides dependencies to children
4. **Contract Interfaces**: Router/Interactor define clear contracts

### Android-Specific (From RIB_ANDROID.md)
1. **ViewRouter**: Manages View hierarchy alongside RIB tree
2. **ViewInteractor**: Communicates with Presenter via Rx
3. **ViewPresenter**: Updates View reactively based on state
4. **Lifecycle Sync**: View lifecycle tied to RIB lifecycle

### iOS-Specific (From RIBs_iOS.md)
1. **Protocol-Based Design**: All components are protocols
2. **ViewControllable**: View layer abstraction
3. **Listener Pattern**: Parent-child communication
4. **RxSwift Integration**: Reactive streams throughout

### Testing (From RIB_TEST.md & TESTING_GUIDE.md)
1. **Isolation**: Mock all dependencies
2. **Arrange-Act-Assert**: Clear test structure
3. **Single Responsibility**: One assertion per test
4. **Deterministic**: No timing assumptions

## Integration Points

Understand how modules interact:
```
Application Code
    ↓
rib-android / RIBs (iOS)
    ↓
rib-android-core / Swift Protocols
    ↓
rib-base / Base Interfaces
```

See ARCHITECTURE_OVERVIEW.md for detailed dependency map.

## Key Files by Task

### Adding New RIB
- Reference: RIB_BASE.md + platform-specific doc
- Test template: RIB_TEST.md
- Example path: `android/libraries/rib-base/src/main/kotlin/com/uber/rib/core/`

### Testing
- Unit test patterns: RIB_TEST.md
- Integration patterns: TESTING_GUIDE.md
- Example: See "Testing Patterns" sections in each module doc

### Navigation Changes
- Router patterns: RIB_ANDROID.md
- Advanced routing: RIB_ROUTER_NAVIGATOR.md
- Workflows: RIB_WORKFLOW.md

### Compose Integration
- Compose patterns: RIB_ANDROID_COMPOSE.md
- State management: RIB_ANDROID_COMPOSE.md ("Unidirectional Data Flow")

## Common Patterns & Their Location

| Pattern | Documentation |
|---------|---|
| Creating new RIB | RIB_BASE.md |
| View integration | RIB_ANDROID.md / RIBs_iOS.md |
| Dependency injection | RIB_BASE.md |
| Navigation routing | RIB_ROUTER_NAVIGATOR.md |
| Multi-step flows | RIB_WORKFLOW.md |
| Testing RIBs | RIB_TEST.md + TESTING_GUIDE.md |
| Compose integration | RIB_ANDROID_COMPOSE.md |
| Worker pattern | RIB_BASE.md |
| Event lifecycle | RIB_BASE.md |

## Questions to Ask Yourself

When working on RIBs code, use these questions to find the right documentation:

**"How do I implement X?"**
→ Look in ARCHITECTURE_OVERVIEW.md for overview, then module-specific doc

**"What's the pattern for Y?"**
→ Search "Patterns" section in relevant module doc

**"How do I test Z?"**
→ See TESTING_GUIDE.md + RIB_TEST.md

**"How do these modules work together?"**
→ See ARCHITECTURE_OVERVIEW.md dependency map

**"What's the lifecycle for component A?"**
→ Search "Lifecycle" in RIB_BASE.md or platform-specific doc

## Code Quality Standards

All code should:
1. Follow patterns documented in research docs
2. Be testable (follow TESTING_GUIDE.md)
3. Have clear separation of concerns (Router/Interactor/Presenter)
4. Use proper dependency injection
5. Bind to RIB lifecycle for resource management
6. Include unit tests with mocked dependencies

## References & Further Reading

- **Official RIBs Wiki**: https://github.com/uber/RIBs/wiki
- **Uber Engineering Blog**: Search "RIBs architecture"
- **Module Tutorials**: `android/tutorials/` and `ios/tutorials/`
- **Example Apps**: See platform-specific demos directory

## Version Information

- **RIBs Version**: 0.12.0 (Android), 0.9+ (iOS)
- **Research Doc Version**: 1.0 (May 2026)
- **Last Updated**: May 21, 2026

## For Agents: How to Use This File

This CLAUDE.md file is your guide to the RIBs codebase. When you're assigned a task:

1. **Read this file first** to understand the project structure
2. **Check "Task-Specific Guidance"** section for your task type
3. **Read the referenced research docs** before implementing
4. **Follow the patterns** shown in code examples
5. **Use the "Common Patterns" table** to find specific documentation
6. **Reference examples** in research docs when implementing

### Example Agent Workflow:
```
Task: Add new navigation flow to Android app
↓
Check CLAUDE.md → "Navigation Features"
↓
Read: RIB_ROUTER_NAVIGATOR.md
↓
Read: RIB_WORKFLOW.md (for complex flows)
↓
Check code examples in docs
↓
Implement using established patterns
↓
Test using TESTING_GUIDE.md patterns
↓
Commit and reference this file in commit message
```

## Quick Links for Common Tasks

- **New Android RIB**: [RIB_BASE.md](docs/research/android/RIB_BASE.md) → [RIB_ANDROID.md](docs/research/android/RIB_ANDROID.md)
- **New iOS RIB**: [RIBs_iOS.md](docs/research/ios/RIBs_iOS.md)
- **Add Navigation**: [RIB_ROUTER_NAVIGATOR.md](docs/research/android/RIB_ROUTER_NAVIGATOR.md)
- **Unit Testing**: [TESTING_GUIDE.md](docs/research/TESTING_GUIDE.md) + [RIB_TEST.md](docs/research/android/RIB_TEST.md)
- **Compose Feature**: [RIB_ANDROID_COMPOSE.md](docs/research/android/RIB_ANDROID_COMPOSE.md)
- **Workflow/State Machine**: [RIB_WORKFLOW.md](docs/research/android/RIB_WORKFLOW.md)

---

**Note**: This file is part of the "mental alignment" initiative to ensure all agents and developers work from the same architectural understanding.
