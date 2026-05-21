# RIBs Testing Guide

Comprehensive testing strategies for RIBs architecture across Android and iOS platforms.

## Testing Philosophy

### Core Principles
1. **Test in Isolation**: Mock external dependencies
2. **Fast Execution**: Unit tests should run in milliseconds
3. **Deterministic**: No timing issues or flakiness
4. **Clear Intent**: Test names clearly describe what's being tested
5. **Minimal Setup**: Only mock what's necessary

### Test Pyramid
```
        △
       /|\
      / | \
     /  |  \  UI/Integration Tests (10%)
    /   |   \
   /────┼────\
  /     |     \ Component Tests (30%)
 /──────┼──────\
/       |       \ Unit Tests (60%)
─────────────────
```

## Unit Testing Patterns

### 1. Testing Interactor

**What to Test**:
- Business logic execution
- Lifecycle callbacks
- Event emission
- State management

**Mock Dependencies**:
- Presenter/View
- Router
- Parent Interactor
- External services

**Example - Android**:
```kotlin
class HomeInteractorTest {
    private lateinit var interactor: HomeInteractor
    private lateinit var presenter: MockHomePresenter
    private lateinit var router: MockHomeRouter
    
    @Before
    fun setup() {
        presenter = MockHomePresenter()
        router = MockHomeRouter()
        interactor = HomeInteractor(presenter, router)
    }
    
    @Test
    fun testLoadDataOnAttach() {
        interactor.onAttach(TestLifecycle())
        
        presenter.verify().showLoadingState()
    }
    
    @Test
    fun testNavigateOnItemClick() {
        interactor.handleItemClick(Item(id = "123"))
        
        router.verify().showDetail("123")
    }
}
```

**Example - iOS**:
```swift
class HomeInteractorTests: XCTestCase {
    var interactor: HomeInteractor!
    var presenter: MockHomePresentable!
    var router: MockHomeRouting!
    
    override func setUp() {
        super.setUp()
        presenter = MockHomePresentable()
        router = MockHomeRouting()
        interactor = HomeInteractor(presenter: presenter)
        interactor.router = router
    }
    
    func testLoadDataOnBecomeActive() {
        interactor.didBecomeActive()
        
        XCTAssertTrue(presenter.didLoadCalled)
    }
    
    func testRouteToDetailOnItemSelect() {
        interactor.selectItem(Item(id: "123"))
        
        XCTAssertTrue(router.routeToDetailCalled)
        XCTAssertEqual(router.selectedItemID, "123")
    }
}
```

### 2. Testing Router

**What to Test**:
- Child attachment/detachment
- Navigation logic
- View attachment/detachment
- Lifecycle management

**Mock Dependencies**:
- Interactor
- Builder/Factory
- Parent Router
- ViewContainer

**Example - Android**:
```kotlin
class HomeRouterTest {
    private lateinit var router: HomeRouter
    private lateinit var interactor: MockHomeInteractor
    private lateinit var component: MockComponent
    
    @Before
    fun setup() {
        interactor = MockHomeInteractor()
        component = MockComponent()
        router = HomeRouter(interactor, component)
    }
    
    @Test
    fun testAttachChildRib() {
        val child = MockRouter()
        router.attachChild(child)
        
        assertTrue(router.attachedChildren.contains(child))
    }
    
    @Test
    fun testDetachChildRib() {
        val child = MockRouter()
        router.attachChild(child)
        router.detachChild(child)
        
        assertFalse(router.attachedChildren.contains(child))
    }
    
    @Test
    fun testNavigateToDetail() {
        router.routeToDetail("item-123")
        
        verify(interactor).detailRequested("item-123")
    }
}
```

**Example - iOS**:
```swift
class HomeRouterTests: XCTestCase {
    var router: HomeRouter!
    var interactor: MockHomeInteractor!
    var viewController: MockHomeViewController!
    
    override func setUp() {
        super.setUp()
        interactor = MockHomeInteractor()
        viewController = MockHomeViewController()
        router = HomeRouter(interactor: interactor, viewController: viewController)
    }
    
    func testAttachChild() {
        let child = MockRib()
        router.attachChild(child)
        
        XCTAssertTrue(router.children.contains(where: { $0 === child }))
    }
    
    func testDetachChild() {
        let child = MockRib()
        router.attachChild(child)
        router.detachChild(child)
        
        XCTAssertFalse(router.children.contains(where: { $0 === child }))
    }
}
```

### 3. Testing Presenter/View Layer

**What to Test**:
- View updates on state changes
- Event emission on user actions
- View lifecycle callbacks
- Presenter state management

**Mock Dependencies**:
- UIView/UIViewController
- Business services
- Analytics

**Example - Android**:
```kotlin
class HomePresenterTest {
    private lateinit var presenter: HomePresenter
    private lateinit var view: MockHomeView
    
    @Before
    fun setup() {
        view = MockHomeView()
        presenter = HomePresenter()
        presenter.setView(view)
    }
    
    @Test
    fun testRenderState() {
        val state = HomeState(items = listOf(Item(id = "1")))
        
        presenter.renderState(state)
        
        verify(view).displayItems(state.items)
    }
    
    @Test
    fun testEmitUserAction() {
        val observer = presenter.userActions().test()
        
        presenter.onItemClicked(Item(id = "1"))
        
        observer.assertValue { it is ItemClickedAction }
    }
}
```

### 4. Testing Workers

**What to Test**:
- Task execution
- Lifecycle cleanup
- Error handling
- Resource management

**Example - Android**:
```kotlin
class MyWorkerTest {
    private lateinit var worker: MyWorker
    private lateinit var service: MockService
    
    @Before
    fun setup() {
        service = MockService()
        worker = MyWorker(service)
    }
    
    @Test
    fun testWorkerStartsTask() {
        val lifecycle = TestLifecycle()
        
        worker.onAttach(lifecycle)
        
        verify(service).startTask()
    }
    
    @Test
    fun testWorkerCleansUpOnDetach() {
        val lifecycle = TestLifecycle()
        worker.onAttach(lifecycle)
        
        lifecycle.triggerDetach()
        
        verify(service).cancelTask()
    }
}
```

## Integration Testing Patterns

### 1. RIB Component Integration

**Test**:
- Router and Interactor together
- Full RIB lifecycle
- Component creation
- Child RIB integration

**Example - Android**:
```kotlin
class HomeRibIntegrationTest {
    private lateinit var router: HomeRouter
    
    @Before
    fun setup() {
        val component = createTestComponent()
        router = HomeRouter(
            interactor = component.homeInteractor(),
            component = component
        )
    }
    
    @Test
    fun testRibLifecycle() {
        val lifecycle = TestLifecycle()
        
        router.interactor.onAttach(lifecycle)
        assertTrue(router.interactor.isAttached)
        
        router.interactor.onDetach()
        assertFalse(router.interactor.isAttached)
    }
    
    @Test
    fun testChildAttachmentFlow() {
        val parent = HomeRouter(...)
        val child = DetailRouter(...)
        
        parent.attachChild(child)
        
        // Verify both parent and child are properly scoped
        verify(child.interactor).onAttach(any())
    }
}
```

### 2. Workflow Integration Testing

**Example - Android**:
```kotlin
class CheckoutWorkflowIntegrationTest {
    private lateinit var workflow: CheckoutWorkflow
    
    @Test
    fun testWorkflowCompletion() {
        val observer = workflow.invoke(Unit).test()
        
        // Simulate flow progression
        emailStep.complete("user@example.com")
        passwordStep.complete("password123")
        confirmStep.complete(true)
        
        observer.assertComplete()
        observer.assertValueCount(1)
    }
}
```

## Testing Strategy by Module

### rib-base Testing

```
Interactor Tests
├── Lifecycle callbacks
├── Event emission
└── Business logic

Router Tests
├── Child attachment/detachment
├── Navigation routing
└── Lifecycle management

Builder Tests
├── Component creation
└── Dependency injection

Worker Tests
├── Task execution
└── Lifecycle cleanup
```

### rib-android Testing

```
ViewRouter Tests
├── View attachment/detachment
├── Activity/Fragment integration
└── View hierarchy updates

ViewInteractor Tests
├── Presenter communication
├── View lifecycle binding
└── Activity lifecycle sync

ViewPresenter Tests
├── View rendering
├── Event emission
└── View updates
```

### rib-android-compose Testing

```
ComposePresenter Tests
├── State observable updates
├── Recomposition triggers
└── Event emission

Composable Tests
├── UI rendering
├── State-driven composition
└── User interaction handling
```

### rib-workflow Testing

```
Workflow Tests
├── Step execution order
├── Conditional branching
├── Error handling
└── Cancellation

WorkflowRouter Tests
├── Workflow output handling
├── Step transitions
└── Result mapping
```

### rib-router-navigator Testing

```
Navigator Tests
├── Destination routing
├── Back stack management
├── History tracking

BackStack Tests
├── Push/pop operations
├── Stack state
└── Edge cases
```

## Test Data Builders

### Purpose
Create consistent, reusable test data.

**Example - Android**:
```kotlin
class TestDataBuilders {
    companion object {
        fun homeState() = HomeState(
            items = listOf(testItem()),
            isLoading = false,
            error = null
        )
        
        fun testItem() = Item(
            id = "test-id",
            title = "Test Item",
            description = "Test Description"
        )
    }
}
```

**Example - iOS**:
```swift
class TestDataBuilders {
    static func makeHomeState(
        items: [Item] = [makeItem()]
    ) -> HomeState {
        return HomeState(
            items: items,
            isLoading: false,
            error: nil
        )
    }
    
    static func makeItem() -> Item {
        return Item(
            id: "test-id",
            title: "Test Item",
            description: "Test Description"
        )
    }
}
```

## Mock Implementations

### Creating Mocks

**Android - Using Mockito**:
```kotlin
val mockPresenter = mock(HomePresenter::class.java)
whenever(mockPresenter.userActions())
    .thenReturn(Observable.just(UserAction.Clicked))

verify(mockPresenter, times(1)).setupUI()
```

**iOS - Manual Mocks**:
```swift
class MockHomePresenter: HomePresentable {
    var didLoadCalled = false
    var displayedState: HomeState?
    
    func didLoad() {
        didLoadCalled = true
    }
    
    func render(state: HomeState) {
        displayedState = state
    }
}
```

## Testing Utilities Summary

### Android
- **Mockito**: Mocking framework
- **TestObserver**: RxJava test observer
- **TestRule**: JUnit test rules
- **Robolectric**: Android runtime mocking

### iOS
- **XCTest**: Apple's testing framework
- **Nimble**: Assertion library
- **Quick**: BDD-style testing
- **RxBlocking**: RxSwift test utilities

## Best Practices

### 1. Test Isolation
```kotlin
@Test
fun eachTestIsIndependent() {
    // Each test should not depend on others
    // Setup fresh mocks in @Before
}
```

### 2. Descriptive Names
```kotlin
@Test
fun whenUserClicksDetailButton_thenNavigateToDetailScreen() {
    // Clear what's being tested
}
```

### 3. Single Responsibility
```kotlin
@Test
fun testNavigationTriggering() {
    // Test one thing: does this trigger navigation?
    // Don't test UI rendering in the same test
}
```

### 4. Arrange-Act-Assert
```kotlin
@Test
fun testExample() {
    // Arrange: Setup test data
    val interactor = createInteractor()
    
    // Act: Perform the action
    interactor.handleAction(action)
    
    // Assert: Verify results
    verify(router).navigate()
}
```

### 5. Avoid Test Interdependence
```kotlin
@Before
fun setup() {
    // Fresh setup for each test
    // No shared mutable state
}
```

## Continuous Integration

### Test Execution
- Run unit tests on every commit
- Integration tests on release branches
- Code coverage tracking
- Flakiness detection

### Test Metrics
- Target: >80% code coverage
- Execution time: <5 minutes for full suite
- Flakiness rate: <1%

## Troubleshooting Tests

### Common Issues

**Timing Issues**:
- Use `TestObserver` instead of `Thread.sleep()`
- Block on observables in tests
- Use time-controlled schedulers

**Memory Leaks**:
- Dispose subscriptions in teardown
- Clear mock references
- Check lifecycle cleanup

**Flaky Tests**:
- Avoid timing assumptions
- Mock time-dependent code
- Use deterministic test data

## References

- [Android Testing Guide](https://developer.android.com/training/testing)
- [iOS Testing with XCTest](https://developer.apple.com/documentation/xctest)
- [RxJava Testing](https://github.com/ReactiveX/RxJava/wiki/Testing)
- [RxSwift Testing](https://github.com/ReactiveX/RxSwift/blob/main/Documentation/Testing.md)

---

**Document Version**: 1.0
**Updated**: May 21, 2026
