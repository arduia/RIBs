# rib-test Module Research

## Module Purpose

`rib-test` provides testing utilities, mock implementations, and helper classes for unit testing RIBs-based code.

**Location**: `android/libraries/rib-test/`

## Key Testing Concepts

### RIBs Testing Philosophy

- **Unit test in isolation**: Mock external dependencies
- **Test contracts**: Verify router/interactor interfaces
- **Fast tests**: No Android instrumentation needed for logic tests
- **Deterministic**: Avoid timing issues and flakiness

## Core Testing Components

### 1. TestRouter & RouterTestDouble
Mock implementations for testing routers.

**TestRouter Pattern**:
```kotlin
class MyRouterTest {
    private lateinit var router: MyRouter
    private lateinit var interactor: MyInteractor
    private lateinit var component: Component
    
    @Before
    fun setup() {
        interactor = mock(MyInteractor::class.java)
        component = mock(Component::class.java)
        router = MyRouter(interactor, component)
    }
    
    @Test
    fun testAttachChild() {
        val childRouter = mock(TestRouter::class.java)
        router.attachChild(childRouter)
        
        verify(router).attachChild(childRouter)
    }
}
```

### 2. TestInteractor
Mock implementations for testing interactors.

**Interactor Mocking**:
- Mock presenter/router dependencies
- Verify business logic execution
- Test lifecycle callbacks

**Example**:
```kotlin
class MyInteractorTest {
    private lateinit var interactor: MyInteractor
    private lateinit var presenter: MyPresenter
    private lateinit var router: MyRouter
    
    @Before
    fun setup() {
        presenter = mock(MyPresenter::class.java)
        router = mock(MyRouter::class.java)
        interactor = MyInteractor(presenter, router)
    }
    
    @Test
    fun testHandleUserAction() {
        interactor.handleAction(UserAction.Clicked)
        
        verify(router).navigateToNextScreen()
    }
}
```

### 3. TestPresenter
Mock implementations for testing presenters.

**Presenter Testing**:
- Verify view updates
- Test event emission
- Mock view dependencies

### 4. TestRib & RibTestDouble
Helper for testing complete RIB hierarchies.

**Usage**:
```kotlin
class MyRibTest {
    private lateinit var rib: TestRib
    
    @Before
    fun setup() {
        rib = TestRib(MyRouter(mock(), mock()))
    }
    
    @Test
    fun testRibLifecycle() {
        rib.attach()
        assertEquals(RibEvent.ATTACHED, rib.lastEvent)
        
        rib.detach()
        assertEquals(RibEvent.DETACHED, rib.lastEvent)
    }
}
```

### 5. RxTestUtils
RxJava testing utilities for observable subscriptions.

**Common Utilities**:
- TestObserver/TestSubscriber
- BlockingObservable for synchronous testing
- Observable.test() for RxJava 2

**Example**:
```kotlin
@Test
fun testObservableEmission() {
    val observable = Observable.just("test")
    
    val testObserver = observable.test()
    
    testObserver
        .assertValue("test")
        .assertNoErrors()
        .assertComplete()
}
```

### 6. LifecycleTestUtils
Utilities for testing RIB lifecycle events.

**Lifecycle Testing**:
```kotlin
@Test
fun testInteractorLifecycle() {
    val lifecycle = TestLifecycle()
    val interactor = MyInteractor(presenter, router)
    
    lifecycle.onCreate()
    interactor.onAttach(lifecycle)
    
    verify(presenter).setupUI()
    
    lifecycle.onDestroy()
    // Verify cleanup
}
```

## Testing Patterns

### 1. Unit Testing Router
```kotlin
class MyRouterTest {
    private lateinit var router: MyRouter
    private lateinit var interactor: MyInteractor
    
    @Before
    fun setup() {
        interactor = mock(MyInteractor::class.java)
        router = MyRouter(interactor, mock(Component::class.java))
    }
    
    @Test
    fun testAttachChild() {
        val child = mock(TestRouter::class.java)
        router.attachChild(child)
        assertNotNull(router.attachedChildren.find { it === child })
    }
    
    @Test
    fun testDetachChild() {
        val child = mock(TestRouter::class.java)
        router.attachChild(child)
        router.detachChild(child)
        assertNull(router.attachedChildren.find { it === child })
    }
}
```

### 2. Unit Testing Interactor
```kotlin
class MyInteractorTest {
    private lateinit var interactor: MyInteractor
    private lateinit var presenter: MyPresenter
    private lateinit var router: MyRouter
    
    @Before
    fun setup() {
        presenter = mock(MyPresenter::class.java)
        router = mock(MyRouter::class.java)
        interactor = MyInteractor(presenter, router)
    }
    
    @Test
    fun testHandleActionTriggersNavigation() {
        interactor.handleUserAction(UserAction.Clicked)
        verify(router).showDetails()
    }
    
    @Test
    fun testLifecycleBinding() {
        val lifecycle = TestLifecycle()
        interactor.onAttach(lifecycle)
        
        // Verify subscriptions set up
        verify(presenter).setupActionHandlers()
    }
}
```

### 3. Unit Testing Presenter
```kotlin
class MyPresenterTest {
    private lateinit var presenter: MyPresenter
    private lateinit var view: MyView
    
    @Before
    fun setup() {
        view = mock(MyView::class.java)
        presenter = MyPresenter()
        presenter.setView(view)
    }
    
    @Test
    fun testUpdateState() {
        presenter.updateState(MyState(title = "New Title"))
        verify(view).setTitle("New Title")
    }
    
    @Test
    fun testUserActionEmission() {
        val observer = presenter.userActions().test()
        
        presenter.onUserAction(UserAction.Clicked)
        
        observer.assertValue(UserAction.Clicked)
    }
}
```

### 4. Integration Testing
```kotlin
class MyRibIntegrationTest {
    private lateinit var router: MyRouter
    private lateinit var interactor: MyInteractor
    
    @Before
    fun setup() {
        val component = createTestComponent()
        router = component.router()
        interactor = router.interactor()
    }
    
    @Test
    fun testFullFlow() {
        router.attach()
        
        interactor.handleUserAction(UserAction.Clicked)
        
        verify(router).showNextScreen()
    }
}
```

## Mock & Stub Patterns

### 1. Mock Dependencies
```kotlin
val mockPresenter = mock(MyPresenter::class.java)
val mockRouter = mock(MyRouter::class.java)

// Stub return values
whenever(mockRouter.getTitle()).thenReturn("Test")

// Verify interactions
verify(mockPresenter).updateState(any())
```

### 2. Spy on Real Objects
```kotlin
val realInteractor = MyInteractor(presenter, router)
val spyInteractor = spy(realInteractor)

// Verify method was called on real object
verify(spyInteractor).onAttach(any())
```

### 3. Argument Capture
```kotlin
val captor = argumentCaptor<MyState>()

interactor.handleAction(UserAction.Clicked)

verify(presenter).updateState(captor.capture())
assertEquals("Expected", captor.value.title)
```

## Testing Strategies

### 1. Isolated Component Testing
Test each component independently:

```
Interactor Test
├── Mock Router
├── Mock Presenter
└── Verify business logic

Router Test
├── Mock Interactor
├── Mock Builder
└── Verify navigation logic

Presenter Test
├── Mock View
└── Verify UI updates
```

### 2. Contract Testing
Verify components implement expected contracts:

```kotlin
@Test
fun testInteractorImplementsContract() {
    assertTrue(interactor is Interactor)
    assertTrue(interactor is MyInteractorContract)
}
```

### 3. Lifecycle Testing
Verify proper lifecycle management:

```
onCreate() → Component created
onAttach() → Setup subscriptions
onDetach() → Cleanup subscriptions
onDestroy() → Final cleanup
```

## Key Test Utilities

| Utility | Purpose |
|---------|---------|
| mock() | Create mock object |
| spy() | Wrap real object with mocking |
| whenever() | Stub return values |
| verify() | Assert method calls |
| argumentCaptor | Capture arguments |
| TestObserver | Test RxJava streams |

## Best Practices

### 1. Test Isolation
- Each test should be independent
- Use @Before/@After for setup/cleanup
- No test order dependencies

### 2. Mock External Dependencies
- Always mock Android components
- Mock inter-RIB communication
- Use real implementations for business logic

### 3. Readable Tests
- Clear test names describing what's tested
- Arrange-Act-Assert pattern
- One assertion per test when possible

### 4. Lifecycle Management
- Test attach/detach flows
- Verify cleanup on lifecycle events
- Test error conditions

### 5. Rx Testing
- Use TestObserver for async testing
- Verify observable subscription/disposal
- Test error and completion cases

## Related Modules

- **rib-base**: Core RIB contracts being tested
- **rib-android**: Android-specific test utilities
- **rib-workflow-test**: Workflow-specific testing utilities

## Common Test Patterns

### 1. Router Navigation Test
```kotlin
@Test
fun testNavigationOnUserAction() {
    router.showDetails(item)
    verify(interactor).loadDetails(item)
}
```

### 2. Presenter State Test
```kotlin
@Test
fun testPresenterUpdatesViewOnStateChange() {
    presenter.setState(MyState(updated = true))
    verify(view).updateUI(any())
}
```

### 3. Lifecycle Event Test
```kotlin
@Test
fun testCleanupOnDetach() {
    interactor.onDetach()
    assertTrue(subscription.isDisposed)
}
```
