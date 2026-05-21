# rib-router-navigator Module Context

## Quick Summary
Advanced navigation abstraction providing back stack management, destination routing, and complex navigation patterns. Higher-level navigation API built on top of RIBs routing.

## Location
`android/libraries/rib-router-navigator/`

## Main Purpose
- Provide high-level navigation abstraction
- Manage back stack and navigation history
- Support various navigation patterns (tabs, modals, stacks)
- Handle deep linking
- Decouple navigation logic from routing

## Key Components

### Navigation Abstractions
- **RouterNavigator** - High-level navigation manager
- **Destination** - Target of navigation (sealed classes)
- **NavigationState** - Current navigation state tracking
- **RouterStack** - Stack-based router management
- **BackStack** - Back stack with history

### Navigation Patterns
- **Tab navigation** - Multiple independent stacks
- **Modal navigation** - Overlays on main flow
- **Stack navigation** - Traditional back stack
- **Deep linking** - External navigation targets

## Quick Patterns

```kotlin
// Destination Definition
sealed class Destination {
    object Home : Destination()
    data class Detail(val itemId: String) : Destination()
    object Settings : Destination()
}

// Navigator Implementation
class AppNavigator {
    private val backStack = mutableListOf<Destination>()
    
    fun navigate(destination: Destination) {
        backStack.add(destination)
        executeNavigation(destination)
    }
    
    fun goBack(): Boolean {
        return if (backStack.size > 1) {
            backStack.removeAt(backStack.lastIndex)
            true
        } else false
    }
}

// Router Using Navigator
class AppRouter : Router<Interactor, Component> {
    fun navigate(destination: Destination) {
        when (destination) {
            is Destination.Detail -> showDetailRib(destination.itemId)
            is Destination.Settings -> showSettingsRib()
        }
    }
}

// Tab Navigation Pattern
class TabRouter {
    private val tabs = mutableMapOf<Tab, Router>()
    
    fun selectTab(tab: Tab) {
        tabs[tab]?.attach()
        tabs.filterKeys { it != tab }.values.forEach { it.detach() }
    }
}

// Back Stack Management
class BackStackNavigator {
    private val stack = mutableListOf<Destination>()
    
    fun push(destination: Destination) { stack.add(destination) }
    fun pop(): Destination? = if (stack.size > 1) stack.removeAt(stack.lastIndex) else null
    fun canGoBack(): Boolean = stack.size > 1
}
```

## Integration Points
- **rib-base**: Uses Router as foundation
- **rib-android**: View-based navigation
- **rib-workflow**: Complex multi-step flows
- **rib-router-navigator-test**: Navigation testing

## Files to Know
- `com.uber.rib.navigation.RouterNavigator` - Navigation manager
- `com.uber.rib.navigation.Destination` - Navigation targets
- `com.uber.rib.navigation.NavigationState` - State tracking
- `com.uber.rib.navigation.RouterStack` - Stack management

## Common Tasks
- ✅ Tab navigation → Tab router with multiple stacks
- ✅ Modal presentation → Modal router on main flow
- ✅ Back navigation → BackStack with canGoBack checking
- ✅ Deep linking → Parse path and rebuild stack
- ✅ Navigation history → Track in NavigationState

## Navigation Patterns Supported

### 1. Stack-Based (Traditional)
```
Home → Detail → SubDetail → (back) → Detail → (back) → Home
```

### 2. Tab-Based
```
Tab A: Home → Detail
Tab B: Settings
(switch tabs maintain state)
```

### 3. Modal-Based
```
Home
  ↓ present
Modal (can dismiss)
  ↓ dismiss
Home
```

### 4. Deep Link
```
External link → Parse path → Rebuild stack → Show target
```

## See Also
→ **Full Research**: [docs/research/android/RIB_ROUTER_NAVIGATOR.md](../../../../docs/research/android/RIB_ROUTER_NAVIGATOR.md)
→ **Workflows**: [docs/research/android/RIB_WORKFLOW.md](../../../../docs/research/android/RIB_WORKFLOW.md)
→ **Basic Routing**: [docs/research/android/RIB_ANDROID.md](../../../../docs/research/android/RIB_ANDROID.md)
→ **Testing**: [docs/research/TESTING_GUIDE.md](../../../../docs/research/TESTING_GUIDE.md)
