# rib-screen-stack-base Module Context

## Quick Summary
Foundation for screen stack-based navigation patterns. Provides base classes and utilities for managing stacks of screens/RIBs with back stack navigation.

## Location
`android/libraries/rib-screen-stack-base/`

## Main Purpose
- Provide screen stack management foundation
- Support traditional back stack navigation
- Manage screen lifecycle within stacks
- Enable push/pop navigation patterns
- Support screen history tracking

## Key Components

### Screen Stack Management
- **ScreenStack** - Stack of screens/RIBs
- **ScreenStackRouter** - Router managing screen stack
- **ScreenStackEvent** - Stack events (push, pop, etc.)
- **ScreenStackListener** - Observe stack changes

### Navigation Operations
- **Push** - Add screen to stack
- **Pop** - Remove screen from stack
- **Replace** - Replace top screen
- **Clear** - Clear entire stack

## Quick Patterns

```kotlin
// Screen Stack Router
class ScreenStackRouter(
    interactor: Interactor,
    component: Component
) : ViewRouter<ScreenStackView, Component> {
    
    private val screenStack = mutableListOf<Router>()
    
    fun push(screen: Router) {
        screenStack.add(screen)
        attachChild(screen)
        attachChildView(screen, view.screenContainer)
    }
    
    fun pop(): Router? {
        return screenStack.removeLastOrNull()?.also { screen ->
            detachChild(screen)
            detachChildView(screen)
        }
    }
    
    fun canGoBack(): Boolean = screenStack.size > 1
}

// Navigation Usage
fun navigateToDetail(itemId: String) {
    val detailRib = detailBuilder.build(itemId)
    screenStackRouter.push(detailRib)
}

fun goBack() {
    if (screenStackRouter.canGoBack()) {
        screenStackRouter.pop()
    }
}
```

## Integration Points
- **rib-android**: View-based screen stack
- **rib-router-navigator**: Advanced navigation patterns
- **rib-base**: Core RIB routing

## Files to Know
- Screen stack base classes
- Stack event definitions
- Router extensions for stacks

## Common Tasks
- ✅ Push screen → Add to stack and show
- ✅ Pop screen → Remove and hide
- ✅ Back navigation → Pop from stack
- ✅ Deep stack → Multiple screens active
- ✅ Stack state → Track and restore

## Stack Navigation Pattern

```
[Screen 1]                Initial state
  ↓ (push)
[Screen 2]
[Screen 1]                Two screens in stack
  ↓ (push)
[Screen 3]
[Screen 2]
[Screen 1]                Three screens in stack
  ↓ (pop)
[Screen 2]
[Screen 1]                Back to two screens
```

## See Also
→ **Navigation**: [docs/research/android/RIB_ROUTER_NAVIGATOR.md](../../../../docs/research/android/RIB_ROUTER_NAVIGATOR.md)
→ **Android Integration**: [docs/research/android/RIB_ANDROID.md](../../../../docs/research/android/RIB_ANDROID.md)
