# rib-android-core Module Context

## Quick Summary
Low-level Android integration layer providing core utilities for Activity, Fragment, and Android component lifecycle management. Foundation for higher-level View-based RIBs.

## Location
`android/libraries/rib-android-core/`

## Main Purpose
- Provide Android-specific utilities and helpers
- Manage Activity and Fragment lifecycle integration
- Handle Android context and resource access
- Support Android lifecycle callbacks
- Enable proper lifecycle cleanup

## Key Components

### Lifecycle Integration
- **ActivityEventProvider** - Activity lifecycle events
- **FragmentEventProvider** - Fragment lifecycle events
- **AndroidLifecycleOwner** - Lifecycle provider
- **AndroidLifecycleDisposable** - Lifecycle-aware disposal

### Context and Resources
- **Android Context** - App/Activity context access
- **Resource loading** - Android resources
- **Config handling** - App configuration

### Utilities
- **Android utilities** - Common Android helpers
- **View utilities** - View system helpers
- **Activity utilities** - Activity helpers

## Quick Patterns

```kotlin
// Activity Lifecycle Binding
class MyActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Create root RIB tied to activity lifecycle
        val rib = rootBuilder.build(this)
        rib.attach()
    }
    
    override fun onDestroy() {
        rib.detach()
        super.onDestroy()
    }
}

// Fragment Lifecycle Binding
class MyFragment : Fragment() {
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        val rib = ribBuilder.build(this)
        rib.attach()
        return rib.viewableRouter?.viewController?.view ?: View(context)
    }
}

// Context Access
val context = androidContext // Access within RIB
val resources = context.resources // Resource access

// Lifecycle Aware Disposal
lifecycle.onBeforeDetach
    .subscribe { disposables.dispose() }
```

## Integration Points
- **rib-base**: Provides lifecycle management
- **rib-android**: Higher-level View integration
- **Android Framework**: AppCompatActivity, Fragment, etc.

## Files to Know
- Core lifecycle event providers
- Android context utilities
- Fragment/Activity support classes
- Android-specific lifecycle binding

## Common Tasks
- ✅ Activity-based RIB root → Lifecycle binding
- ✅ Fragment-based RIBs → Fragment lifecycle integration
- ✅ Context access → Use androidContext
- ✅ Resource loading → Access via context
- ✅ Lifecycle cleanup → onBeforeDetach handling

## See Also
→ **Android Integration**: [docs/research/android/RIB_ANDROID.md](../../../../docs/research/android/RIB_ANDROID.md)
→ **Lifecycle Management**: [docs/research/android/RIB_BASE.md](../../../../docs/research/android/RIB_BASE.md)
