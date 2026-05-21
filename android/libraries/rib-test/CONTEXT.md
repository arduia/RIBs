# rib-test Module Context

## Quick Summary
Testing utilities and mock implementations for unit and integration testing RIBs-based code. Provides test doubles, fixtures, and helpers for fast, isolated testing.

## Location
`android/libraries/rib-test/`

## Main Purpose
- Provide mock/stub implementations for testing
- Enable fast unit tests without Android instrumentation
- Offer test fixtures and builders for test data
- Support deterministic, flake-free testing
- Provide RxJava testing utilities

## Key Components

### Mock Implementations
- **TestRouter** - Mock router for testing
- **TestInteractor** - Mock interactor for testing
- **TestPresenter** - Mock presenter for testing
- **TestRib** - Complete RIB test double

### Testing Utilities
- **RecordingObserver** - Record Observable emissions
- **TestObserver** - RxJava test observer
- **TestScheduler** - Controlled time scheduling
- **DisposeBagTestUtils** - Verify disposables

### Test Builders
- **Test data builders** - Consistent test data
- **Component builders** - Test component factories
- **Fixture builders** - Reusable test setups

## Quick Patterns

```kotlin
// Mock Dependencies
val mockPresenter = mock(MyPresenter::class.java)
val mockRouter = mock(MyRouter::class.java)

// Stub Return Values
whenever(mockRouter.getTitle()).thenReturn("Test Title")

// Create Interactor for Testing
val interactor = MyInteractor(mockPresenter, mockRouter)

// Test Business Logic
interactor.handleUserAction(UserAction.Clicked)
verify(mockRouter).navigateToDetail()

// Test with TestObserver
val observable = Observable.just("test")
val observer = observable.test()
observer.assertValue("test").assertComplete()

// Test Data Builder
fun testState() = MyState(
    title = "Test Title",
    items = listOf(testItem()),
    isLoading = false
)

// Router Testing
val router = MyRouter(interactor, component)
val child = mockRouter()
router.attachChild(child)
assertTrue(router.attachedChildren.contains(child))

// Presenter Testing
val presenter = MyPresenter()
presenter.setView(mockView)
presenter.updateState(testState())
verify(mockView).renderState(testState())
```

## Integration Points
- **rib-base**: Test doubles for base classes
- **rib-android**: Test utilities for View-based RIBs
- **rib-workflow**: Workflow testing helpers
- All modules depend on this for testing

## Files to Know
- `com.uber.rib.core.TestRouter` - Router test double
- `com.uber.rib.core.TestInteractor` - Interactor test double
- `com.uber.rib.core.RxTestUtils` - Observable test helpers
- `com.uber.rib.core.TestObserver` - Observer for testing

## Common Testing Tasks

### Unit Testing Router
```kotlin
@Test
fun testChildAttachment() {
    val child = mockRouter()
    router.attachChild(child)
    assertTrue(router.attachedChildren.contains(child))
}
```

### Unit Testing Interactor
```kotlin
@Test
fun testNavigationTriggering() {
    interactor.handleAction(UserAction.Clicked)
    verify(router).navigateToDetail()
}
```

### Unit Testing Presenter
```kotlin
@Test
fun testStateRendering() {
    presenter.updateState(testState())
    verify(view).renderState(testState())
}
```

### Integration Testing
```kotlin
@Test
fun testRibLifecycle() {
    val lifecycle = TestLifecycle()
    interactor.onAttach(lifecycle)
    verify(presenter).setupUI()
}
```

## Testing Best Practices

### 1. Isolation
- Mock all external dependencies
- Test each component independently
- No inter-component coupling in tests

### 2. Determinism
- Use TestScheduler for time-dependent code
- Use TestObserver for async testing
- Avoid Thread.sleep() and timing assumptions

### 3. Clarity
- Clear test names describing intent
- Arrange-Act-Assert pattern
- One logical assertion per test

### 4. Lifecycle
- Verify lifecycle callbacks
- Test attach/detach flows
- Verify cleanup on destruction

## See Also
→ **Full Research**: [docs/research/android/RIB_TEST.md](../../../../docs/research/android/RIB_TEST.md)
→ **Testing Guide**: [docs/research/TESTING_GUIDE.md](../../../../docs/research/TESTING_GUIDE.md)
→ **Workflow Testing**: [docs/research/android/RIB_WORKFLOW.md](../../../../docs/research/android/RIB_WORKFLOW.md)
