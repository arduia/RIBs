# rib-android-compose Module Research

## Module Purpose

`rib-android-compose` provides integration between RIBs architecture and Jetpack Compose, enabling modern declarative UI development while maintaining RIBs business logic patterns.

**Location**: `android/libraries/rib-android-compose/`

## Key Concepts

### Compose + RIBs Integration

Unlike traditional Android Views, Compose is declarative and composable:

- **Traditional Views**: RIB contains mutable View state, Presenter mutates View
- **Compose**: RIB holds immutable state, Compose recomposes from state changes

This module bridges the gap by:
1. Maintaining RIBs architecture for business logic
2. Using Compose for UI rendering
3. State flows from Interactor → Presenter → Compose

## Core Components

### 1. ComposePresenter
Extends ViewPresenter for Compose-based UIs.

**Key Differences from ViewPresenter**:
- Works with Compose state flows instead of View references
- Uses State<T> or StateFlow<T> for observable state
- Composables are defined separately from Presenter

**Example**:
```kotlin
class MyComposePresenter : ComposePresenter<MyState>() {
    private val stateSubject = BehaviorSubject.create<MyState>()
    
    override val state: Observable<MyState> = stateSubject
    
    fun updateState(newState: MyState) {
        stateSubject.onNext(newState)
    }
}
```

### 2. Composable Integration
Consume presenter state in Composables.

**Pattern**:
```kotlin
@Composable
fun MyComposable(presenter: MyComposePresenter) {
    val state by presenter.state
        .toComposeState(rememberCoroutineScope())
    
    MyContent(state)
}

@Composable
fun MyContent(state: MyState) {
    Column {
        Text(state.title)
        Button(onClick = { /* trigger presenter action */ }) {
            Text("Action")
        }
    }
}
```

### 3. State Management in Compose RIBs
Observable to Compose State conversion.

**Key Utilities**:
- `Observable<T>.toComposeState()`: Convert Rx Observable to Compose State
- State updates trigger recomposition
- Compose lifecycle management integrated with RIB lifecycle

### 4. ComposeView Container
Android View that hosts Compose content within traditional View hierarchy.

**Usage**:
```kotlin
// ViewRouter still uses traditional container
class MyRouter : ViewRouter<ComposeView> {
    override fun inflateView(): ComposeView {
        return ComposeView(context).apply {
            setContent {
                MyComposable(presenter)
            }
        }
    }
}
```

## Architecture Patterns

### 1. Unidirectional Data Flow
```
Interactor
    ↓ (state changes)
ComposePresenter (holds state observable)
    ↓ (observable streams)
Composable UI (recomposes on state change)
    ↓ (user events)
ComposePresenter (subject emissions)
    ↓ (event streams)
Interactor (handles business logic)
```

### 2. Lifecycle Integration
Compose lifecycle managed within RIB lifecycle:
```
RIB Attach
    ↓
ComposeView.setContent() called
    ↓
Composables start recomposing on state changes
    ↓
RIB Detach
    ↓
Lifecycle scope cleanup
```

### 3. State Flow Pattern
```kotlin
class MyComposePresenter : ComposePresenter<MyState>() {
    private val _state = MutableStateFlow(MyState())
    val state: StateFlow<MyState> = _state
    
    fun updateUI(newState: MyState) {
        _state.value = newState
    }
}
```

## Integration with View System

### Nested Compose + Traditional Views
RIBs containing both Compose and traditional Views:

```kotlin
class MyRouter : ViewRouter<MyView> {
    
    override fun inflateView(): MyView {
        return MyView(context).apply {
            // Mix traditional View children
            addView(traditionalChild)
            
            // And Compose content
            findViewById<ComposeView>(R.id.compose_container).apply {
                setContent {
                    MyComposable(presenter)
                }
            }
        }
    }
}
```

### Fragment + Compose
```kotlin
class MyComposeFragment : Fragment() {
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        return ComposeView(requireContext()).apply {
            setContent {
                MyComposable(presenter)
            }
        }
    }
}
```

## Key Advantages

### 1. Declarative UI
- Express UI as function of state
- Automatic recomposition on state change
- Less boilerplate than imperative View updates

### 2. Compose Ecosystem
- Material Design 3 components
- Built-in animation framework
- Modifier system for styling

### 3. Maintained RIBs Structure
- Business logic still follows RIBs patterns
- Interactor, Router, Builder intact
- Testability preserved

## Testing Patterns

### 1. Composable Unit Testing
```kotlin
@Test
fun testMyComposable() {
    val state = MyState(title = "Test")
    
    composeTestRule.setContent {
        MyContent(state)
    }
    
    composeTestRule.onNodeWithText("Test")
        .assertIsDisplayed()
}
```

### 2. Presenter Testing
```kotlin
@Test
fun testPresenterStateChanges() {
    val presenter = MyComposePresenter()
    
    val states = mutableListOf<MyState>()
    presenter.state.subscribe { states.add(it) }
    
    presenter.updateUI(MyState(newTitle = "Updated"))
    
    assertEquals("Updated", states.last().title)
}
```

### 3. Integration Testing
```kotlin
@Test
fun testRibWithCompose() {
    val router = MyRouter(interactor, component)
    
    router.attachChild(childRib)
    
    composeTestRule.waitUntil {
        // Verify compose content updates
    }
}
```

## Performance Considerations

### 1. State Management
- Use minimal state in Compose state
- Keep derived state in Presenter
- Avoid unnecessary recompositions

### 2. RIB Scope
- One Presenter per RIB
- One Observable<State> per Presenter
- Subscriptions cleaned up on detach

### 3. Memory
- Compose instances cleaned up on RIB detach
- No memory leaks from unmanaged Compose state
- Lifecycle binding handles cleanup

## Key Classes

| Class | Purpose |
|-------|---------|
| ComposePresenter | Presenter base for Compose UIs |
| Composable functions | UI composable definitions |
| ComposeView | Android View hosting Compose |
| StateFlow/Observable | State reactive streams |

## Related Modules

- **rib-android**: Base Android integration
- **rib-base**: Core RIBs contracts
- **rib-router-navigator**: Advanced routing with Compose

## Common Patterns

### 1. Simple State-Driven UI
```kotlin
@Composable
fun MyScreen(state: MyState, onAction: (Action) -> Unit) {
    Column {
        Text(state.title)
        Button(onClick = { onAction(Action.Clicked) }) {
            Text("Click me")
        }
    }
}
```

### 2. Observable to Compose
```kotlin
fun <T> Observable<T>.toComposeState(scope: CoroutineScope): State<T?> {
    val state = remember { mutableStateOf<T?>(null) }
    
    LaunchedEffect(Unit) {
        subscribe { value ->
            state.value = value
        }
    }
    
    return state
}
```

### 3. Lifecycle-Aware Subscription
```kotlin
@Composable
fun rememberObservable<T>(observable: Observable<T>): State<T?> {
    val state = remember { mutableStateOf<T?>(null) }
    val lifecycle = LocalLifecycleOwner.current.lifecycle
    
    LaunchedEffect(observable) {
        // Handle subscription within Compose lifecycle
    }
    
    return state
}
```
