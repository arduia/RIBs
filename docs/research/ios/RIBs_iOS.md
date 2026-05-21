# RIBs iOS Framework Research

## Framework Purpose

The RIBs iOS framework provides the core architecture for building scalable iOS applications with a hierarchical, reactive approach to business logic and UI management.

**Location**: `ios/RIBs/`

## Core Architecture

The iOS RIBs implementation follows the same core principles as Android but adapted for Swift and Rx conventions.

### Key Differences from Android

1. **Swift instead of Kotlin**: Type-safe, protocol-based design
2. **RxSwift instead of RxJava**: Reactive streams in Swift
3. **UIKit/SwiftUI integration**: iOS-specific UI frameworks
4. **Protocol-oriented design**: Extensive use of Swift protocols
5. **AutoLayout/SwiftUI**: Declarative or constraint-based layouts

## Core Components

### 1. RIB Protocol
Base protocol for all RIBs.

**Key Protocol**:
```swift
protocol Rib: AnyObject {
    var interactable: Interactable { get }
    var viewableRouter: ViewableRouter? { get }
}
```

**Responsibilities**:
- Define RIB interface
- Provide interactable reference
- Optional view router reference

### 2. Interactor
Handles business logic and lifecycle management.

**Key Protocol**:
```swift
protocol Interactable: AnyObject {
    var isActive: BehaviorRelay<Bool> { get }
    var interactorScope: InteractorScope { get }
}
```

**Interactor Responsibilities**:
- Manage business logic state
- Handle user actions from presenter
- Coordinate with router
- Manage child interactors
- Bind lifecycle to dependencies

**Example**:
```swift
class HomeInteractor: Interactor, HomePresentable {
    weak var router: HomeRouting?
    let presenter: HomePresentable
    
    override func didBecomeActive() {
        super.didBecomeActive()
        
        presenter.userActions
            .subscribe(onNext: { [weak self] action in
                self?.handleUserAction(action)
            })
            .disposed(by: disposeBag)
    }
}
```

### 3. Router
Manages navigation and child RIB lifecycle.

**Router Key Responsibilities**:
- Attach/detach child RIBs
- Manage navigation logic
- Coordinate view attachment
- Handle routing state

**Protocol Pattern**:
```swift
protocol HomeRouting: AnyObject {
    func routeToDetail(item: Item)
    func routeToSettings()
}

class HomeRouter: Router<HomeInteractor, HomeViewController>, HomeRouting {
    
    func routeToDetail(item: Item) {
        let detailRib = detailBuilder.build(item: item)
        attachChild(detailRib)
        presentViewController(detailRib.viewControllable)
    }
}
```

### 4. Builder
Dependency injection and RIB construction factory.

**Builder Pattern**:
```swift
protocol HomeBuildable: AnyObject {
    func build(withListener listener: HomeListener) -> HomeRouting
}

class HomeBuilder: HomeBuildable {
    
    func build(withListener listener: HomeListener) -> HomeRouting {
        let viewController = HomeViewController()
        let interactor = HomeInteractor(presenter: viewController)
        interactor.listener = listener
        
        let router = HomeRouter(
            interactor: interactor,
            viewController: viewController
        )
        return router
    }
}
```

### 5. Presenter/ViewControllable
View layer abstraction.

**Presenter Protocol**:
```swift
protocol HomePresentable: AnyObject {
    var userActions: Observable<UserAction> { get }
    func updateUI(with state: HomeState)
}

protocol HomeViewController: ViewControllable, HomePresentable {}
```

**View Controller Implementation**:
```swift
class HomeViewController: UIViewController, HomeViewController {
    let userActionsSubject = PublishSubject<UserAction>()
    
    var userActions: Observable<UserAction> {
        return userActionsSubject.asObservable()
    }
    
    func updateUI(with state: HomeState) {
        // Update view
    }
}
```

### 6. Listener Pattern
Parent-child RIB communication.

**Pattern**:
```swift
protocol HomeListener: AnyObject {
    func homeDidFinish()
    func homeDidSelectItem(_ item: Item)
}

class HomeInteractor: Interactor, HomePresentable {
    weak var listener: HomeListener?
    
    func userSelectedItem(_ item: Item) {
        listener?.homeDidSelectItem(item)
    }
}
```

### 7. Component & Dependency Injection
Dagger-like dependency management using Swift protocols.

**Component Definition**:
```swift
protocol HomeDependency: AnyObject {
    var networkService: NetworkService { get }
    var analyticsService: AnalyticsService { get }
}

protocol HomeComponent: AnyObject {
    var homeViewController: HomeViewController { get }
    var interactor: HomeInteractor { get }
}

class HomeComponent: Component<HomeDependency>, HomeComponent {
    
    var homeViewController: HomeViewController {
        let vc = HomeViewController()
        return vc
    }
    
    var interactor: HomeInteractor {
        return HomeInteractor(
            presenter: homeViewController,
            networkService: dependency.networkService
        )
    }
}
```

## iOS-Specific Patterns

### 1. Navigation Patterns

#### UINavigationController
```swift
class NavigationRouter: Router<Interactor, UINavigationController> {
    
    func pushViewController(_ viewController: UIViewController) {
        self.viewControllable?.uiviewController.pushViewController(
            viewController,
            animated: true
        )
    }
}
```

#### UITabBarController
```swift
class TabRouter: Router<Interactor, UITabBarController> {
    
    func selectTab(_ index: Int) {
        self.viewControllable?.uiviewController.selectedIndex = index
    }
}
```

#### Modal Presentation
```swift
class ModalRouter: Router<Interactor, UIViewController> {
    
    func presentModal(_ viewController: UIViewController) {
        self.viewControllable?.uiviewController.present(
            viewController,
            animated: true
        )
    }
}
```

### 2. Lifecycle Management

**RxSwift Disposables**:
```swift
class HomeInteractor: Interactor {
    let disposeBag = DisposeBag()
    
    override func didBecomeActive() {
        super.didBecomeActive()
        
        setupSubscriptions()
    }
    
    private func setupSubscriptions() {
        presenter.userActions
            .subscribe(onNext: { [weak self] action in
                self?.handleAction(action)
            })
            .disposed(by: disposeBag)
    }
}
```

### 3. RxSwift Integration

**Observable Streams**:
```swift
// State stream
let stateStream: Observable<AppState> = 
    userInputs
        .withLatestFrom(currentState)
        .map { input, state in
            self.reducer(input, state)
        }

// Presenter subscription
stateStream
    .subscribe(onNext: { [weak self] state in
        self?.presenter.render(state)
    })
    .disposed(by: disposeBag)
```

## Architecture Benefits

### 1. Testability
```swift
class HomeInteractorTests: XCTestCase {
    var interactor: HomeInteractor!
    var presenter: MockHomePresentable!
    var router: MockHomeRouter!
    
    override func setUp() {
        super.setUp()
        presenter = MockHomePresentable()
        router = MockHomeRouter()
        interactor = HomeInteractor(presenter: presenter)
        interactor.router = router
    }
    
    func testUserActionTriggersNavigation() {
        interactor.handleUserAction(.itemSelected(item))
        XCTAssertTrue(router.didRouteToDetail)
    }
}
```

### 2. Reusability
- RIBs are composable and reusable
- Protocol-based design enables mocking
- Dependency injection allows configuration

### 3. Scalability
- Hierarchical structure scales to large apps
- Clear separation of concerns
- Independent testing of components

## Comparison with SwiftUI

### Traditional UIKit Pattern
```swift
class HomeViewController: UIViewController, HomePresentable {
    @IBOutlet weak var tableView: UITableView!
    
    func updateUI(with state: HomeState) {
        tableView.reloadData()
    }
}
```

### SwiftUI Integration
```swift
struct HomeView: View, HomePresentable {
    @State var state: HomeState
    
    var body: some View {
        List(state.items) { item in
            Text(item.title)
        }
    }
    
    var userActions: Observable<UserAction> {
        userActionsSubject.asObservable()
    }
}
```

## Testing Strategies

### 1. Unit Testing Interactor
```swift
func testInteractorHandlesUserAction() {
    let router = MockHomeRouter()
    let presenter = MockHomePresentable()
    
    let interactor = HomeInteractor(presenter: presenter)
    interactor.router = router
    
    interactor.handleAction(.viewTapped)
    
    XCTAssertTrue(router.routeWasCalled)
}
```

### 2. Integration Testing
```swift
func testRibAttachmentDetachment() {
    let childRib = childBuilder.build()
    parentRouter.attachChild(childRib)
    
    XCTAssertTrue(childRib.interactable.isActive.value)
    
    parentRouter.detachChild(childRib)
    
    XCTAssertFalse(childRib.interactable.isActive.value)
}
```

## Key Files Structure

```
RIBs/
├── Core/
│   ├── Rib.swift
│   ├── Router.swift
│   ├── Interactor.swift
│   └── ViewControllable.swift
├── ReactiveComponentArchitecture/
│   ├── Interactor+
│   └── Router+
├── Utilities/
│   ├── WeakProxy.swift
│   └── DisposeBagExtension.swift
└── Protocols/
    ├── Buildable.swift
    ├── Presentable.swift
    └── Interactable.swift
```

## Integration Points

### 1. RxSwift
- Reactive streams throughout
- Observable subscriptions with DisposeBag
- Subject-based event emission

### 2. UIKit/SwiftUI
- ViewControllable protocol for UI abstraction
- Support for both UIKit and SwiftUI
- Lifecycle binding to view controllers

### 3. Dependency Injection
- Protocol-based components
- Type-safe dependency provision
- Hierarchical scoping

## Best Practices

### 1. Protocol Design
- Use protocols for all interfaces
- Small, focused protocols
- Testability through protocol mocking

### 2. Memory Management
- Use weak references in listeners
- Dispose subscriptions properly
- Clear child references on detach

### 3. Error Handling
- Propagate errors through observables
- Handle errors at appropriate levels
- Log errors for debugging

### 4. Testing
- Mock all dependencies
- Test in isolation
- Verify state changes through observables

## Related Resources

- iOS tutorials in `ios/tutorials/`
- Example implementations in samples
- IDE tooling in `ios/tooling/`
