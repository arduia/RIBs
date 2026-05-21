# rib-android-compose Module Context

## Quick Summary
Jetpack Compose integration for RIBs, enabling declarative UI development while maintaining business logic separation. Bridges modern Compose with traditional RIBs patterns.

## Location
`android/libraries/rib-android-compose/`

## Main Purpose
- Integrate Jetpack Compose with RIBs architecture
- Manage state flow from Interactor through Presenter to Composables
- Provide ComposePresenter for state-driven UI
- Handle Compose lifecycle within RIB lifecycle
- Enable reactive state updates via Observable streams

## Key Components

### Compose-Specific Classes
- **ComposePresenter** - Presenter base class for Compose UIs
- **Composable** - Declarative UI functions
- **ComposableViewHost** - Container for Compose within View hierarchy
- **StateObservable** - Observable to Compose State conversion

### State Management
- **Observable<State>** - Reactive state stream from Interactor
- **Compose State** - Mutable state within Composables
- **Unidirectional Data Flow** - State → UI recomposition → Events

### Lifecycle Integration
- **Compose lifecycle** - Tied to RIB lifecycle
- **DisposeBag** - Auto-cleanup on RIB detach
- **Effect hooks** - Setup subscriptions in Compose

## Quick Patterns

```kotlin
// ComposePresenter with Observable State
class MyComposePresenter : ComposePresenter<MyState>() {
    private val stateSubject = BehaviorSubject.create<MyState>()
    override val state: Observable<MyState> = stateSubject
    
    fun updateState(newState: MyState) {
        stateSubject.onNext(newState)
    }
}

// Composable Consuming State
@Composable
fun MyScreen(presenter: MyComposePresenter) {
    val state by presenter.state.toComposeState()
    
    MyContent(state = state, onAction = { action ->
        presenter.handleAction(action)
    })
}

// Unidirectional Data Flow
Interactor (state changes)
    ↓ (via Observable)
ComposePresenter (holds state)
    ↓ (via Composable recomposition)
Composable UI (renders state)
    ↓ (user interaction)
Observable Subject (event emission)
    ↓ (to Interactor)
Business Logic
```

## Integration Points
- **rib-android**: Extends ViewPresenter for Compose
- **rib-base**: Uses Interactor/Router lifecycle
- **rib-workflow**: Compose-based workflow UIs
- **rib-router-navigator**: Navigation with Compose

## Files to Know
- `com.uber.rib.core.compose.ComposePresenter` - Presenter base class
- `com.uber.rib.core.compose.StateObservable` - Observable conversion utilities
- `com.uber.rib.core.compose.ComposableViewHost` - Compose in View container

## Common Tasks
- ✅ Creating Compose UI → Use ComposePresenter + @Composable
- ✅ State updates → Emit from Presenter Observable
- ✅ User events → Create Subject in Presenter
- ✅ Recomposition → Automatic on state change
- ✅ Lifecycle binding → Handled by RIB lifecycle

## Architecture Pattern
```
Interactor (business logic)
    ↓
ComposePresenter (state holder + event emitter)
    ↓
@Composable functions (declarative UI)
    ↓
User actions (reflow to Interactor)
```

## See Also
→ **Full Research**: [docs/research/android/RIB_ANDROID_COMPOSE.md](../../../../docs/research/android/RIB_ANDROID_COMPOSE.md)
→ **Traditional Alternative**: [docs/research/android/RIB_ANDROID.md](../../../../docs/research/android/RIB_ANDROID.md)
→ **Testing Composables**: [docs/research/TESTING_GUIDE.md](../../../../docs/research/TESTING_GUIDE.md)
