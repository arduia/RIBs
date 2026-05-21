# rib-base Module Research

## Module Purpose

`rib-base` contains the foundational interfaces and abstract classes that form the core of the RIBs architecture. This module is platform-agnostic and provides the contracts that all RIB implementations must follow.

**Location**: `android/libraries/rib-base/`

## Core Components

### 1. Rib Class
**File**: `com/uber/rib/core/Rib.kt`

Global configuration point for the RIBs framework.

**Key Features**:
- Singleton configuration management
- Non-fatal error/warning handling callbacks
- Debug message routing
- Default configuration with crash-on-error behavior

**Usage**:
```kotlin
Rib.setConfiguration(object : Rib.Configuration {
    override fun handleNonFatalError(msg: String, throwable: Throwable?) { }
    override fun handleNonFatalWarning(msg: String, throwable: Throwable?) { }
    override fun handleDebugMessage(format: String, vararg args: Any?) { }
})
```

### 2. Interactor & RibInteractor
**Files**: 
- `com/uber/rib/core/Interactor.kt`
- `com/uber/rib/core/RibInteractor.kt`
- `com/uber/rib/core/BasicInteractor.kt`

Handles business logic and state management for a RIB.

**Interactor Responsibilities**:
- Execute business logic
- Manage interactor lifecycle
- Communicate with parent/child interactors
- Handle user interactions from presenter
- Trigger routing changes

**Key Methods**:
- `onAttach()`: Called when interactor becomes active
- `onDetach()`: Called when interactor is being removed
- Lifecycle aware observable streams

**Interactor Lifecycle**:
- Created with parent scope
- Attached when child RIB is added
- Detached when child RIB is removed
- Can be tested independently with mock presenters/routers

### 3. Router & BasicRouter
**Files**:
- `com/uber/rib/core/Router.kt`
- `com/uber/rib/core/BasicRouter.kt`

Manages navigation, child RIB attachment/detachment, and navigation logic.

**Router Responsibilities**:
- Handle routing logic
- Attach/detach child RIBs
- Manage child RIB lifecycle
- Respond to interactor commands
- Communicate navigation state

**Key Operations**:
- `attachChild(child: Rib)`: Add and initialize child RIB
- `detachChild(child: Rib)`: Remove and cleanup child RIB
- Hierarchical RIB tree management

### 4. Builder & RibBuilder
**Files**:
- `com/uber/rib/core/Builder.kt`
- `com/uber/rib/core/RibBuilder.kt`

Factory pattern implementation for RIB construction with dependency injection.

**Builder Pattern**:
- Accepts parent component/dependencies
- Returns fully constructed RIB
- Manages component dependency graph
- Type-safe dependency provision

**Components Created**:
- Interactor instance
- Router instance
- Presenter (if UI component)
- View (if UI component)

### 5. Presenter
**File**: `com/uber/rib/core/Presenter.kt`

Interface for view presentation logic (optional for non-UI RIBs).

**Presenter Responsibilities**:
- Update view based on state
- Handle view user interactions
- Communicate events to interactor via subjects
- Manage view lifecycle independently from business logic

**Key Pattern**:
- Reactive streams (Rx) for event emission
- Subjects for user input capture

### 6. Worker System
**Files**:
- `com/uber/rib/core/Worker.kt`
- `com/uber/rib/core/WorkerBinder.kt`
- `com/uber/rib/core/WorkerScopeProvider.kt`

Background task execution with lifecycle binding.

**Worker Pattern**:
- Bounded to RIB lifecycle
- Automatic cleanup on RIB detachment
- Executes on provided scheduler
- No view access

**Usage Example**:
```kotlin
class MyWorker(
    val someService: SomeService
) : Worker {
    override fun onAttach(nodeLifecycle: NodeLifecycle) {
        nodeLifecycle.onBeforeDetach.subscribe {
            // Cleanup when RIB detaches
        }
        // Start background work
    }
}
```

### 7. Bundle
**File**: `com/uber/rib/core/Bundle.kt`

Data container for passing configuration to RIB builders.

**Purpose**:
- Encapsulate configuration data
- Type-safe parameter passing
- Enable testing with mock data

### 8. RibEvent & Lifecycle
**Files**:
- `com/uber/rib/core/RibEvent.kt`
- `com/uber/rib/core/RibEventType.kt`
- `com/uber/rib/core/RibEvents.kt`
- `com/uber/rib/core/lifecycle/*`

Lifecycle event tracking system.

**RibEvent Types**:
- `CREATED`: Component instantiated
- `ATTACHED`: Component attached to parent
- `DETACHED`: Component removed from parent
- `DESTROYED`: Component cleanup complete

**Lifecycle Events**:
- InteractorEvent
- PresenterEvent
- WorkerEvent

### 9. Component Interfaces
**Files**:
- `com/uber/rib/core/InteractorBaseComponent.kt`
- `com/uber/rib/core/InteractorComponent.kt`
- `com/uber/rib/core/InteractorAndViewModule.kt`

Dagger/dependency injection component definitions.

**Component Hierarchy**:
- `InteractorBaseComponent`: Parent dependency provider
- `InteractorComponent`: Scoped component for RIB
- View-specific modules for UI RIBs

## Key Architecture Patterns

### 1. Dependency Injection Pattern
Each RIB explicitly declares what dependencies it needs from parent:
```kotlin
interface Component {
    fun interactor(): MyInteractor
    fun router(): MyRouter
}
```

### 2. Lifecycle Binding
All components tied to interactor lifecycle:
- Workers auto-cleanup on detach
- Subscriptions auto-dispose on detach
- Resources released predictably

### 3. Hierarchical Scoping
Parent components scope dependencies for children:
- Constructor injection of parent component
- Child builders access parent dependencies
- Clear dependency resolution order

## Testing Considerations

### Unit Testing
- Interactor: Mock router, presenter, parent interactor
- Router: Mock builder, interactor, parent
- Worker: Mock lifecycle, dependencies

### Integration Testing
- Real builder with mock dependencies
- Component tree structure validation
- Lifecycle event sequence verification

## Integration with Other Modules

- **rib-android**: Extends base with Android View integration
- **rib-workflow**: Uses Interactor/Router for state management
- **rib-test**: Provides testing utilities based on base contracts
- **rib-compiler-***: Processes annotations for code generation

## Key Files Summary

| File | Purpose |
|------|---------|
| Rib.kt | Global configuration |
| Interactor.kt | Business logic contract |
| Router.kt | Navigation contract |
| Builder.kt | Construction contract |
| Worker.kt | Background task contract |
| RibEvent.kt | Lifecycle tracking |
| Bundle.kt | Configuration data |
