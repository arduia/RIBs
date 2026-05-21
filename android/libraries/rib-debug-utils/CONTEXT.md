# rib-debug-utils Module Context

## Quick Summary
Debugging and diagnostic utilities for RIBs development. Provides memory leak detection, RIB tree inspection, and debugging tools.

## Location
`android/libraries/rib-debug-utils/`

## Main Purpose
- Detect memory leaks in RIBs
- Inspect RIB tree structure
- Provide debugging information
- Track RIB lifecycle events
- Enable development-time diagnostics

## Key Components

### Memory Leak Detection
- **RibRefWatcher** - Track RIB instances
- **LeakCanary integration** - Memory leak detection
- **RIB validator** - Verify proper cleanup

### RIB Tree Inspection
- **RibTreeDumper** - Print RIB hierarchy
- **RibTreeInspector** - Inspect structure
- **RibDebugger** - Debug utilities

### Lifecycle Tracking
- **Event logging** - Log lifecycle events
- **State tracking** - Track RIB state
- **Attachment tracking** - Monitor attach/detach

## Quick Patterns

```kotlin
// Memory Leak Detection
Rib.setConfiguration(object : Rib.Configuration {
    override fun handleNonFatalError(msg: String, throwable: Throwable?) {
        if (BuildConfig.DEBUG) {
            Log.e("RibDebug", msg, throwable)
            // Could integrate with crash reporting
        }
    }
})

// Enable RIB Ref Watching
RibRefWatcher.enableIfDragsterPresent(context)

// Inspect RIB Tree
val debugInfo = RibTreeDumper.dumpRibTree(rootRib)
Log.d("RibTree", debugInfo)

// Track Lifecycle Events
ribEventListener.onEvent { event ->
    Log.d("RibLifecycle", "${event.rib}: ${event.type}")
}
```

## Integration Points
- **rib-base**: Core RIB lifecycle
- **rib-android**: View-based RIBs
- **Development**: Debug builds and testing

## Files to Know
- Memory leak detection utilities
- RIB tree inspection tools
- Lifecycle event logging
- Debug configuration helpers

## Common Debugging Tasks
- ✅ Find memory leaks → Use RibRefWatcher
- ✅ Inspect RIB tree → Use RibTreeDumper
- ✅ Track lifecycle → Monitor events
- ✅ Debug issues → Check event logs

## Debug Information Available

### RIB Tree
- Hierarchical structure
- Parent-child relationships
- Current state

### Lifecycle Events
- CREATED, ATTACHED, DETACHED, DESTROYED
- Timestamps
- Event chain

### Memory Usage
- Active RIBs count
- Detached but not destroyed RIBs
- Potential leaks

## See Also
→ **Architecture**: [docs/research/android/RIB_BASE.md](../../../../docs/research/android/RIB_BASE.md)
→ **Lifecycle**: [docs/research/ARCHITECTURE_OVERVIEW.md](../../../../docs/research/ARCHITECTURE_OVERVIEW.md)
