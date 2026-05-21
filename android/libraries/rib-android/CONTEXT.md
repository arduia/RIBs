# rib-android Module Context

## Quick Summary
Android-specific implementation adding View hierarchy management, Activity/Fragment lifecycle integration, and UI-based RIB patterns. Bridges Android framework with RIBs architecture.

## Location
`android/libraries/rib-android/`

## Main Purpose
- Integrate RIBs with Android View system
- Manage Activity/Fragment lifecycle binding
- Provide ViewRouter and ViewInteractor abstractions
- Handle View attachment/detachment
- Support traditional Android UI patterns

## Key Components

### View-Based RIBs
- **ViewRouter** - Router with View management capability
- **ViewInteractor** - Interactor communicating with presenter
- **ViewPresenter** - Presenter updating Views reactively
- **RibView** - Base interface for all RIB views

### Lifecycle Management
- **Activity/Fragment lifecycle** - RIB lifecycle tied to UI lifecycle
- **ViewLifecycleBinding** - Synchronizes View and RIB lifecycles
- **LifecycleDisposeBag** - Auto-cleanup on View destruction

### Memory Leak Detection
- **RibRefWatcher** - Detects RIBs not properly detached
- **MemoryLeakWatcher** - LeakCanary integration

## Quick Patterns

```kotlin
// View Router Pattern
class MyRouter(
    interactor: MyInteractor,
    component: MyComponent
) : ViewRouter<MyView, MyComponent>(interactor, component) {
    override fun inflateView(): MyView = MyView(context)
    
    fun showDetailRib(item: Item) {
        val detailRib = detailBuilder.build(item)
        attachChild(detailRib)
        attachChildView(detailRib, myView.container)
    }
}

// Presenter with Observable Events
class MyPresenter : ViewPresenter<MyView>() {
    private val userActionsSubject = PublishSubject.create<UserAction>()
    override fun didLoad() { view.setupUI() }
    fun userActions(): Observable<UserAction> = userActionsSubject
}

// Interactor Handling Events
override fun didBecomeActive() {
    presenter.userActions()
        .subscribe { action -> router.handleAction(action) }
        .disposed(by: disposeBag)
}
```

## Integration Points
- **rib-base**: Extends Router/Interactor with View support
- **rib-android-compose**: Alternative with Compose
- **rib-router-navigator**: Advanced navigation patterns
- **rib-workflow**: State machine workflows
- All Android RIB implementations depend on this

## Files to Know
- `com.uber.rib.core.ViewRouter` - Base Router with View management
- `com.uber.rib.core.ViewInteractor` - Base Interactor for UI
- `com.uber.rib.core.ViewPresenter` - Base Presenter for View updates
- `com.uber.rib.core.RibActivity` - Activity lifecycle support
- `com.uber.rib.core.RibFragment` - Fragment lifecycle support

## Common Tasks
- ✅ Creating new View-based RIB → Use ViewRouter + ViewInteractor
- ✅ Handling user input → Emit from Presenter via Observable
- ✅ Updating View → Subscribe in Presenter to state changes
- ✅ Managing child Views → Use attachChildView/detachChildView

## See Also
→ **Full Research**: [docs/research/android/RIB_ANDROID.md](../../../../docs/research/android/RIB_ANDROID.md)
→ **Compose Alternative**: [docs/research/android/RIB_ANDROID_COMPOSE.md](../../../../docs/research/android/RIB_ANDROID_COMPOSE.md)
→ **Testing**: [docs/research/TESTING_GUIDE.md](../../../../docs/research/TESTING_GUIDE.md)
