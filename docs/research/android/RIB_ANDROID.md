# rib-android Module Research

## Module Purpose

`rib-android` provides Android-specific implementations and extensions of the base RIBs architecture, including View integration, Android lifecycle management, and Activity/Fragment support.

**Location**: `android/libraries/rib-android/`

## Key Responsibilities

- Android View hierarchy integration with RIB tree
- Activity and Fragment lifecycle management
- View binding and presenter integration
- Android-specific dependency injection
- Memory leak detection and debugging

## Core Components

### 1. ViewRouter & ViewRibRouter
Extends base Router with Android View management.

**Key Features**:
- `attachChildView(child, viewGroup)`: Attach child RIB's view to parent
- `detachChildView(child)`: Remove child view from parent
- Automatic view container management
- View lifecycle synced with RIB lifecycle

**Usage Pattern**:
```kotlin
class MyRouter(
    interactor: MyInteractor,
    component: Component
) : ViewRouter<MyView, MyComponent>(interactor, component) {
    
    override fun inflateView(): MyView = MyView(context)
    
    fun showChild(child: MyViewRouter) {
        attachChild(child)
        attachChildView(child, myView.childContainer)
    }
}
```

### 2. ViewInteractor
Extends base Interactor with View binding.

**Responsibilities**:
- Interacts with presenter
- Receives user input from presenter
- Maintains business logic state
- Triggers routing on logic changes

**Presenter Communication**:
```kotlin
class MyInteractor(
    val presenter: MyPresenter
) : ViewInteractor<MyPresenter, MyRouter> {
    
    override fun onAttach(lifecycle: NodeLifecycle) {
        presenter.userActions()
            .subscribe { action ->
                handleUserAction(action)
            }
    }
}
```

### 3. ViewPresenter
Android View controller pattern for reactive UI updates.

**Key Responsibilities**:
- Update view based on state changes
- Capture user interactions
- Emit events via Rx Subjects
- Manage view lifecycle

**Pattern**:
```kotlin
class MyPresenter : ViewPresenter<MyView>() {
    private val userActionSubject = PublishSubject.create<UserAction>()
    
    override fun didLoad() {
        view.setupUI()
    }
    
    fun userActions(): Observable<UserAction> = userActionSubject
    
    fun updateState(state: MyState) {
        view.render(state)
    }
}
```

### 4. ActivityLifecycle Integration
Manages RIB lifecycle tied to Activity lifecycle.

**Lifecycle Synchronization**:
- RIB created when Activity created
- RIB attached when Activity resumed
- RIB detached when Activity paused
- RIB destroyed when Activity destroyed

### 5. Fragment Support
Integration with Android Fragments for nested RIBs.

**Pattern**:
- Fragment hosts a RIB's view
- Fragment lifecycle drives RIB lifecycle
- Multiple RIBs can coexist in single Activity

## Android-Specific Patterns

### 1. View Inflation
Different approaches to creating views:

```kotlin
// XML layout inflation
override fun inflateView(): MyView {
    val view = LayoutInflater.from(router.context)
        .inflate(R.layout.my_view, null) as MyView
    return view
}

// Programmatic creation
override fun inflateView(): MyView = MyView(router.context)
```

### 2. Presenter Observable Patterns
User input capture using RxJava:

```kotlin
class MyPresenter : ViewPresenter<MyView>() {
    private val itemClickedSubject = PublishSubject.create<Item>()
    
    override fun didLoad() {
        view.itemClicks()
            .subscribe(itemClickedSubject)
    }
    
    fun itemClicks(): Observable<Item> = itemClickedSubject
}
```

### 3. State Management
Reactive state updates from interactor to presenter:

```kotlin
interactor.stateChanges()
    .subscribe { state ->
        presenter.updateState(state)
    }
```

## Integration Points

### Activity/Fragment Lifecycle
```
Activity.onCreate()
    ↓
RIB.create()
    ↓
Activity.onResume()
    ↓
RIB.attach()
    ↓
Activity.onPause()
    ↓
RIB.detach()
    ↓
Activity.onDestroy()
    ↓
RIB.destroy()
```

### View Hierarchy Synchronization
```
RIB Tree (Business Logic)
    ↓ (Presenter)
View Tree (Android UI)
```

## Testing in Android

### Unit Testing
- Mock Android Context/Resources
- Test Router without Activity
- Test Interactor independently
- Test Presenter with mock View

### Integration Testing
- Test with FragmentTestRule
- Verify Activity lifecycle drives RIB lifecycle
- Test navigation and view swapping

## Memory Management

### Leak Detection
- RibRefWatcher tracks RIB instances
- Detects RIBs not properly detached
- Integration with LeakCanary

### Auto-Cleanup
- Workers cleanup on detach
- Subscriptions auto-disposed on detach
- View references cleared on detach

## Key Classes

| Class | Purpose |
|-------|---------|
| ViewRouter | Base Router with view management |
| ViewInteractor | Base Interactor with view presenter |
| ViewPresenter | Base Presenter for view updates |
| RibView | Base interface for all rib views |
| RouterNavigator | Navigation between RIBs |

## Related Modules

- **rib-android-core**: Lower-level Android integration
- **rib-base**: Base contracts extended here
- **rib-android-compose**: Jetpack Compose integration
- **rib-router-navigator**: Advanced routing capabilities

## Common Patterns

### 1. View Routing
```kotlin
class MyRouter : ViewRouter<MyView> {
    fun showDetailRib(item: Item) {
        val detailRib = DetailBuilder(...)
            .build(item)
        attachChild(detailRib)
        attachChildView(detailRib, myView.detailContainer)
    }
}
```

### 2. Presenter Event Streams
```kotlin
presenter.userActions()
    .distinctUntilChanged()
    .switchMap { action -> interactor.handleAction(action) }
    .subscribe { result -> presenter.showResult(result) }
```

### 3. Lifecycle Binding
```kotlin
interactor.onAttach { lifecycle ->
    lifecycle.createScope()
        .subscribe { /* Rx work bounded to RIB lifecycle */ }
}
```
