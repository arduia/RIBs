# rib-base Module Context

## Quick Summary
Foundation module providing core RIBs interfaces and abstract base classes. Defines the contracts for Interactor, Router, Builder, and Worker patterns used across all RIBs implementations.

## Location
`android/libraries/rib-base/`

## Main Purpose
- Define core RIB contracts (interfaces and abstract classes)
- Provide lifecycle management utilities
- Enable dependency injection patterns
- Support Worker-based background tasks
- Track RIB events and lifecycle

## Key Components

### Core Classes
- **Interactor** - Business logic component
- **Router** - Navigation and child RIB management
- **Builder** - Factory for RIB construction
- **Worker** - Background task execution
- **Rib** - Global configuration holder

### Lifecycle Management
- **RibEvent** - Lifecycle events (CREATED, ATTACHED, DETACHED, DESTROYED)
- **NodeLifecycle** - Component lifecycle binding
- **InteractorEvent**, **PresenterEvent**, **WorkerEvent** - Specific lifecycle tracking

### Dependency Injection
- **InteractorComponent** - Dagger component contracts
- **Bundle** - Configuration data container

## Quick Patterns

```kotlin
// Basic RIB Components
class MyInteractor(val presenter: MyPresenter) : Interactor<MyPresenter, MyRouter>
class MyRouter(interactor: MyInteractor, component: MyComponent) : Router<MyInteractor, MyComponent>(interactor, component)
class MyBuilder : Builder<MyRouter, MyComponent> { override fun build(component: MyComponent) = MyRouter(...) }

// Lifecycle Binding
interactor.onAttach { lifecycle ->
    someService.startWork()
    lifecycle.onBeforeDetach.subscribe { someService.cleanup() }
}

// Worker Pattern
class MyWorker(val service: MyService) : Worker {
    override fun onAttach(nodeLifecycle: NodeLifecycle) {
        nodeLifecycle.onBeforeDetach.subscribe { service.cleanup() }
    }
}
```

## Integration Points
- **rib-android**: Extends with ViewRouter/ViewInteractor
- **rib-workflow**: Uses Interactor/Router for state machines
- **rib-test**: Provides testing utilities
- All other modules depend on this

## Files to Know
- `com.uber.rib.core.Rib` - Global config
- `com.uber.rib.core.Interactor` - Business logic interface
- `com.uber.rib.core.Router` - Navigation interface
- `com.uber.rib.core.Builder` - Factory interface
- `com.uber.rib.core.Worker` - Background task interface

## See Also
→ **Full Research**: [docs/research/android/RIB_BASE.md](../../../../docs/research/android/RIB_BASE.md)
→ **Testing**: [docs/research/TESTING_GUIDE.md](../../../../docs/research/TESTING_GUIDE.md)
