# rib-workflow Module Research

## Module Purpose

`rib-workflow` provides state machine and workflow management for coordinating complex business logic flows, multi-step processes, and state transitions across RIBs.

**Location**: `android/libraries/rib-workflow/`

## Key Concepts

### Workflows vs Routing

- **Routing**: Simple parent-child RIB navigation
- **Workflow**: Complex multi-step state machines with conditional logic

Workflows are useful for:
- Multi-screen flows (onboarding, checkout)
- State-dependent branching
- Coordinating multiple RIBs
- Handling cancellation and completion

## Core Components

### 1. Workflow<T>
Base class for defining state machine workflows.

**Generic Type T**: Output type when workflow completes

**Key Methods**:
```kotlin
abstract class Workflow<T> {
    abstract fun invoke(input: Unit): Observable<T>
}
```

**Pattern**:
```kotlin
class SignupWorkflow : Workflow<SignupResult> {
    override fun invoke(input: Unit): Observable<SignupResult> {
        return emailStep()
            .flatMap { email -> passwordStep(email) }
            .flatMap { password -> confirmationStep(password) }
            .map { confirmed -> SignupResult.Success(confirmed) }
            .onErrorReturn { error -> SignupResult.Error(error) }
    }
}
```

### 2. WorkflowRouter
Router that manages workflow-based child RIB transitions.

**Responsibilities**:
- Execute workflow observable
- Route based on workflow output
- Manage child RIB transitions
- Handle cancellation

**Key Methods**:
```kotlin
class SignupRouter : WorkflowRouter<SignupResult> {
    
    fun startSignup() {
        signupWorkflow()
            .subscribe { result ->
                when (result) {
                    is SignupResult.Success -> showSuccess()
                    is SignupResult.Error -> showError()
                    is SignupResult.Cancelled -> showHome()
                }
            }
    }
}
```

### 3. Actionable<T>
Interface for RIBs that handle workflow actions.

**Purpose**: RIBs that are part of workflow emit output to drive workflow forward

**Example**:
```kotlin
interface EmailScreenActionable : Actionable<String> {
    // RIB emits email string when user continues
}

class EmailScreenRouter : 
    ViewRouter<EmailView>,
    EmailScreenActionable {
    
    override fun onContinueClicked(email: String) {
        publishSubject.onNext(email)
    }
}
```

### 4. WorkflowListener
Observes workflow events and lifecycle.

**Events Tracked**:
- Workflow started
- Step transitions
- Workflow completed
- Workflow cancelled/errored

## Architecture Patterns

### 1. Sequential Workflow
Multiple steps execute one after another:

```kotlin
class CheckoutWorkflow : Workflow<OrderResult> {
    override fun invoke(input: Unit): Observable<OrderResult> {
        return selectItemsStep()
            .flatMap { items -> enterAddressStep(items) }
            .flatMap { address -> enterPaymentStep(address) }
            .flatMap { payment -> confirmOrderStep(payment) }
            .map { order -> OrderResult.Success(order) }
    }
}
```

### 2. Conditional Branching
Workflow routing based on conditions:

```kotlin
class OnboardingWorkflow : Workflow<OnboardingResult> {
    override fun invoke(input: Unit): Observable<OnboardingResult> {
        return selectTypeStep()
            .flatMap { type ->
                when (type) {
                    UserType.DRIVER -> driverOnboardingStep()
                    UserType.RIDER -> riderOnboardingStep()
                }
            }
            .map { profile -> OnboardingResult.Success(profile) }
    }
}
```

### 3. Error Handling
Graceful error handling in workflows:

```kotlin
class PaymentWorkflow : Workflow<PaymentResult> {
    override fun invoke(input: PaymentInfo): Observable<PaymentResult> {
        return processPaymentStep(input)
            .onErrorResumeNext { error ->
                when (error) {
                    is NetworkError -> showRetryStep()
                    is InvalidCardError -> showReplaceCardStep()
                    else -> Observable.error(error)
                }
            }
    }
}
```

### 4. Cancellation Handling
Allow users to cancel workflow:

```kotlin
class ModalWorkflow : Workflow<ModalResult> {
    private val cancelSubject = PublishSubject.create<ModalResult>()
    
    override fun invoke(input: Unit): Observable<ModalResult> {
        return Observable.merge(
            workflowSteps(),
            cancelSubject
        )
    }
    
    fun cancel() {
        cancelSubject.onNext(ModalResult.Cancelled)
    }
}
```

## Integration with RIBs

### 1. Workflow-Based Router
```kotlin
class MyRouter(
    interactor: MyInteractor,
    component: Component
) : WorkflowRouter<WorkflowOutput> {
    
    private val workflow = MyWorkflow()
    
    fun startWorkflow() {
        workflow.invoke(Unit)
            .subscribe { output ->
                handleWorkflowOutput(output)
            }
    }
}
```

### 2. Child RIB in Workflow
RIB that emits output to advance workflow:

```kotlin
class EmailScreenRouter(
    interactor: Interactor,
    component: Component
) : ViewRouter<EmailView>, EmailScreenActionable {
    
    private val outputSubject = PublishSubject.create<String>()
    
    override fun onContinueClicked(email: String) {
        outputSubject.onNext(email)
    }
    
    override fun asObservable(): Observable<String> = outputSubject
}
```

### 3. Parent Handles Workflow
```kotlin
class OnboardingRouter : WorkflowRouter<OnboardingOutput> {
    
    fun startOnboarding() {
        onboardingWorkflow()
            .subscribe { output ->
                when (output) {
                    is EmailOutput -> showPasswordScreen(output.email)
                    is PasswordOutput -> showConfirmationScreen(output.password)
                    is CompleteOutput -> finish()
                }
            }
    }
}
```

## Advanced Patterns

### 1. Nested Workflows
Workflows that contain sub-workflows:

```kotlin
class CheckoutWorkflow : Workflow<OrderResult> {
    override fun invoke(input: Unit): Observable<OrderResult> {
        return shippingWorkflow()
            .flatMap { shipping ->
                billingWorkflow(shipping)
            }
            .flatMap { billing ->
                paymentWorkflow(billing)
            }
    }
}
```

### 2. Workflow with Side Effects
Perform actions alongside workflow execution:

```kotlin
class DownloadWorkflow : Workflow<DownloadResult> {
    override fun invoke(input: URL): Observable<DownloadResult> {
        return validateURLStep(input)
            .flatMap { url ->
                downloadFile(url)
                    .doOnNext { bytes -> cacheFile(bytes) }
            }
            .map { result -> DownloadResult.Success(result) }
    }
}
```

### 3. Resumable Workflows
Save and resume workflow state:

```kotlin
class ResumeableWorkflow : Workflow<WorkflowResult> {
    private var savedState: WorkflowState? = null
    
    override fun invoke(input: Unit): Observable<WorkflowResult> {
        val startPoint = savedState?.let { resumeFrom(it) } 
            ?: initialStep()
        return startPoint.flatMap { /* ... */ }
    }
    
    fun saveState(state: WorkflowState) {
        savedState = state
    }
}
```

## Testing Patterns

### 1. Workflow Unit Testing
```kotlin
@Test
fun testSignupWorkflow() {
    val observer = TestObserver<SignupResult>()
    
    signupWorkflow()
        .test()
        .assertValue { result ->
            result is SignupResult.Success
        }
}
```

### 2. Step Testing
```kotlin
@Test
fun testEmailStep() {
    emailWorkflow()
        .test()
        .assertValue { email -> email.isNotEmpty() }
}
```

### 3. Router with Workflow
```kotlin
@Test
fun testWorkflowRouter() {
    val router = MyRouter(interactor, component)
    
    router.startWorkflow()
    
    verify(router).handleNextStep()
}
```

## Key Classes

| Class | Purpose |
|-------|---------|
| Workflow<T> | Base workflow state machine |
| WorkflowRouter | Router managing workflow |
| Actionable<T> | RIB output interface |
| WorkflowListener | Event observer |

## Related Modules

- **rib-base**: Core RIBs Router/Interactor used in workflows
- **rib-android**: View Router extensions for workflows
- **rib-workflow-test**: Testing utilities for workflows

## Common Use Cases

### 1. Multi-Screen Onboarding
Email → Password → Confirmation → Success

### 2. Purchase Flow
Select Items → Address → Payment → Confirmation

### 3. Form Wizards
Multiple sequential steps with validation

### 4. User Flows
Complex user interaction sequences with branching

## Best Practices

1. **Keep workflows focused**: Single responsibility per workflow
2. **Error handling**: Always handle failure cases
3. **Cancellation**: Allow users to exit workflows
4. **Testability**: Test workflow steps independently
5. **Memory**: Clean up subscriptions on workflow completion
