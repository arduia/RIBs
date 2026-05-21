# rib-router-navigator Module Research

## Module Purpose

`rib-router-navigator` provides advanced navigation capabilities and router composition patterns for managing complex navigation flows, back stack management, and routing state.

**Location**: `android/libraries/rib-router-navigator/`

## Key Concepts

### Navigation vs Routing

- **Routing**: Direct parent-child RIB attachment (basic RIBs pattern)
- **Navigation**: Higher-level abstraction managing complex flows, history, and state

Navigation is useful for:
- Back stack management
- Deep linking
- Navigation history
- Complex routing logic
- Screen stack patterns

## Core Components

### 1. RouterNavigator
Higher-level navigation manager composing multiple routers.

**Key Responsibilities**:
- Manage back stack
- Handle navigation commands
- Coordinate router transitions
- Track navigation history

**Pattern**:
```kotlin
class AppNavigator {
    private val backStack = mutableListOf<Router>()
    
    fun navigate(destination: Destination) {
        when (destination) {
            is HomeDestination -> showHome()
            is DetailDestination -> showDetail(destination.id)
            is SettingsDestination -> showSettings()
        }
    }
    
    fun goBack() {
        backStack.removeAt(backStack.lastIndex)?.detach()
    }
}
```

### 2. Destination Pattern
Sealed classes defining possible navigation destinations.

**Definition**:
```kotlin
sealed class Destination {
    object Home : Destination()
    data class Detail(val itemId: String) : Destination()
    object Settings : Destination()
}
```

**Usage**:
```kotlin
navigator.navigate(Destination.Detail("item-123"))
```

### 3. NavigationState
Tracks current navigation state.

**State Information**:
- Current destination
- Navigation history
- Back stack depth
- Can-go-back state

### 4. RouterStack
Manages a stack of routers for screen stack pattern.

**Pattern** (like Android navigation stack):
```kotlin
class ScreenRouter(
    interactor: Interactor,
    component: Component
) {
    private val routerStack = mutableListOf<Router>()
    
    fun push(screen: Router) {
        routerStack.add(screen)
        attachChild(screen)
    }
    
    fun pop(): Router? {
        return routerStack.removeLastOrNull().also { router ->
            router?.detach()
        }
    }
}
```

## Navigation Patterns

### 1. Tab Navigation
Multiple tabs with independent back stacks:

```kotlin
class TabRouter {
    private val tabs = mutableMapOf<Tab, Router>()
    
    fun selectTab(tab: Tab) {
        tabs[tab]?.attach()
        tabs.filterKeys { it != tab }
            .values.forEach { it.detach() }
    }
}
```

### 2. Modal Navigation
Modal presentations on top of main stack:

```kotlin
class ModalRouter {
    private var currentModal: Router? = null
    
    fun presentModal(modal: Router) {
        currentModal = modal
        attachChild(modal)
    }
    
    fun dismissModal() {
        currentModal?.detach()
        currentModal = null
    }
}
```

### 3. Deep Linking
Reconstruct navigation state from deep links:

```kotlin
class DeepLinkNavigator {
    fun navigate(deepLink: String) {
        val path = parseDeepLink(deepLink)
        
        // Rebuild back stack
        for (segment in path) {
            when (segment) {
                "home" -> pushHome()
                "detail" -> pushDetail(segment.id)
                "settings" -> pushSettings()
            }
        }
    }
}
```

### 4. Conditional Navigation
Route based on state:

```kotlin
class ConditionalRouter {
    fun navigateAfterAuth(user: User?) {
        if (user != null) {
            navigateToHome(user)
        } else {
            navigateToLogin()
        }
    }
}
```

## Integration with RIBs

### 1. Navigator in Router
```kotlin
class AppRouter(
    interactor: Interactor,
    component: Component
) {
    private val navigator = AppNavigator()
    
    fun navigate(destination: Destination) {
        navigator.navigate(destination)
    }
}
```

### 2. Interactor Requests Navigation
```kotlin
class HomeInteractor(
    val presenter: HomePresenter,
    val router: HomeRouter
) : Interactor<HomePresenter, HomeRouter> {
    
    override fun onAttach(lifecycle: NodeLifecycle) {
        presenter.itemClicks()
            .subscribe { item ->
                router.navigate(Destination.Detail(item.id))
            }
    }
}
```

### 3. Router Handles Navigation
```kotlin
class HomeRouter(
    interactor: HomeInteractor,
    component: Component
) : Router<HomeInteractor, Component> {
    
    fun navigate(destination: Destination) {
        when (destination) {
            is Destination.Detail -> {
                val detailRouter = detailBuilder.build(destination.itemId)
                attachChild(detailRouter)
                attachChildView(detailRouter, view.container)
            }
        }
    }
}
```

## Advanced Navigation Patterns

### 1. Navigation Commands
Decouple navigation logic from routing:

```kotlin
interface NavigationCommand {
    fun execute(router: Router)
}

data class PushScreenCommand(val screen: Router) : NavigationCommand {
    override fun execute(router: Router) {
        router.push(screen)
    }
}

// Usage
val command = PushScreenCommand(detailRouter)
command.execute(mainRouter)
```

### 2. Navigation Interceptors
Intercept and modify navigation:

```kotlin
interface NavigationInterceptor {
    fun onBeforeNavigate(destination: Destination): Boolean
    fun onAfterNavigate(destination: Destination)
}

class AuthInterceptor : NavigationInterceptor {
    override fun onBeforeNavigate(destination: Destination): Boolean {
        return if (destination.requiresAuth && !isAuthenticated()) {
            navigateToLogin()
            false
        } else {
            true
        }
    }
}
```

### 3. Navigation History
Track and replay navigation:

```kotlin
class NavigationHistory {
    private val history = mutableListOf<Destination>()
    
    fun record(destination: Destination) {
        history.add(destination)
    }
    
    fun replay(count: Int): List<Destination> {
        return history.takeLast(count)
    }
}
```

### 4. Observable Navigation
Emit navigation events:

```kotlin
class ObservableNavigator {
    private val navigationSubject = PublishSubject.create<Destination>()
    
    fun navigate(destination: Destination) {
        navigationSubject.onNext(destination)
    }
    
    fun navigationEvents(): Observable<Destination> = navigationSubject
}
```

## Back Stack Management

### 1. Simple Back Stack
```kotlin
class BackStackNavigator {
    private val backStack = mutableListOf<Destination>()
    
    fun push(destination: Destination) {
        backStack.add(destination)
    }
    
    fun pop(): Destination? {
        return if (backStack.size > 1) {
            backStack.removeAt(backStack.lastIndex)
        } else null
    }
    
    fun canGoBack(): Boolean = backStack.size > 1
}
```

### 2. Exclusive Back Stack
Only one instance of each destination type:

```kotlin
class ExclusiveBackStack {
    private val backStack = mutableListOf<Destination>()
    
    fun push(destination: Destination) {
        backStack.removeAll { it::class == destination::class }
        backStack.add(destination)
    }
    
    fun pop(): Destination? = backStack.removeLastOrNull()
}
```

## Testing Navigation

### 1. Navigator Unit Testing
```kotlin
@Test
fun testNavigationToDetail() {
    val navigator = AppNavigator()
    
    navigator.navigate(Destination.Detail("123"))
    
    assertEquals(Destination.Detail("123"), navigator.currentDestination)
}
```

### 2. Back Stack Testing
```kotlin
@Test
fun testBackStackPop() {
    val backStack = BackStackNavigator()
    backStack.push(Destination.Home)
    backStack.push(Destination.Detail("1"))
    
    backStack.pop()
    
    assertEquals(Destination.Home, backStack.current())
}
```

### 3. Router Navigation Testing
```kotlin
@Test
fun testRouterNavigates() {
    val router = AppRouter(interactor, component)
    val destination = Destination.Detail("123")
    
    router.navigate(destination)
    
    verify(router).attachChild(any())
}
```

## Best Practices

### 1. Clear Navigation Hierarchy
- Define all destinations upfront
- Use sealed classes for type safety
- Document navigation rules

### 2. Back Stack Management
- Always allow back to home
- Limit back stack depth
- Clear stack on app backgrounding

### 3. State Preservation
- Save navigation state on pause
- Restore on resume
- Handle process death gracefully

### 4. Testing
- Test each navigation path
- Test back stack behavior
- Test edge cases (multiple pops, empty stack)

## Key Classes

| Class | Purpose |
|-------|---------|
| RouterNavigator | High-level navigation manager |
| Destination | Navigation target definition |
| NavigationState | Current navigation state |
| RouterStack | Stack-based router management |
| BackStackNavigator | Back stack management |

## Related Modules

- **rib-base**: Base Router contracts
- **rib-android**: Android View integration
- **rib-workflow**: Alternative for complex flows

## Common Use Cases

### 1. Tab Navigation with Independent Stacks
Each tab maintains its own back stack.

### 2. Modal Flows
Modals presented on top of main flow.

### 3. Deep Linking
Reconstruct navigation from external links.

### 4. Conditional Navigation
Route based on authentication, permissions, etc.

### 5. Screen Stack Navigation
Traditional stack-based navigation like Android native.
