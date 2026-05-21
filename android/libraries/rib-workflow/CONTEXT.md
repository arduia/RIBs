# rib-workflow Module Context

## Quick Summary
State machine and workflow implementation for multi-step flows and complex business logic coordination. Enables sequential workflows with conditional branching and error handling.

## Location
`android/libraries/rib-workflow/`

## Main Purpose
- Define workflow state machines for multi-step processes
- Manage conditional routing based on workflow state
- Handle workflow cancellation and completion
- Coordinate complex RIB transitions
- Provide observable-based workflow execution

## Key Components

### Workflow Patterns
- **Workflow<T>** - Abstract base for state machines
- **WorkflowRouter** - Router managing workflow-driven transitions
- **Actionable<T>** - RIB output interface for workflow participation
- **WorkflowListener** - Observe workflow events

### Flow Control
- **Sequential steps** - Execute steps one after another
- **Conditional branching** - Route based on conditions
- **Error handling** - Graceful failure scenarios
- **Cancellation** - Allow users to exit workflows

## Quick Patterns

```kotlin
// Sequential Workflow
class SignupWorkflow : Workflow<SignupResult> {
    override fun invoke(input: Unit): Observable<SignupResult> {
        return emailStep()
            .flatMap { email -> passwordStep(email) }
            .flatMap { password -> confirmationStep(password) }
            .map { confirmed -> SignupResult.Success(confirmed) }
            .onErrorReturn { error -> SignupResult.Error(error) }
    }
}

// Conditional Branching
class OnboardingWorkflow : Workflow<OnboardingResult> {
    override fun invoke(input: Unit): Observable<OnboardingResult> {
        return selectTypeStep()
            .flatMap { type ->
                when (type) {
                    UserType.DRIVER -> driverOnboardingStep()
                    UserType.RIDER -> riderOnboardingStep()
                }
            }
    }
}

// Router Handling Workflow Output
class ParentRouter : WorkflowRouter<WorkflowOutput> {
    fun startWorkflow() {
        workflow.invoke(Unit)
            .subscribe { output ->
                when (output) {
                    is EmailStep -> showPasswordScreen(output.email)
                    is PasswordStep -> showConfirmation(output.password)
                    is Complete -> finishWorkflow()
                }
            }
    }
}

// Child RIB Emitting Output
class EmailRouter : ViewRouter<EmailView>, EmailScreenActionable {
    override fun onContinueClicked(email: String) {
        publishSubject.onNext(email)
    }
}
```

## Integration Points
- **rib-base**: Uses Interactor/Router as workflow foundations
- **rib-android**: View-based workflow screens
- **rib-router-navigator**: Complex navigation patterns
- **rib-workflow-test**: Testing workflows

## Files to Know
- `com.uber.rib.workflow.core.Workflow` - Workflow base class
- `com.uber.rib.workflow.WorkflowRouter` - Router for workflows
- `com.uber.rib.workflow.Actionable` - Output interface

## Common Tasks
- ✅ Multi-step onboarding → Sequential workflow steps
- ✅ Checkout process → Conditional branching workflow
- ✅ Error recovery → onErrorResumeNext handling
- ✅ User cancellation → Catch and emit cancelled result
- ✅ Nested workflows → Sub-workflow execution

## Workflow Types

### Sequential Flow
```
Step 1 → Step 2 → Step 3 → Complete
```

### Conditional Flow
```
Decision → Branch A → Result A
       → Branch B → Result B
```

### Error Handling Flow
```
Step 1 → Error → Retry Step → Step 2
```

## See Also
→ **Full Research**: [docs/research/android/RIB_WORKFLOW.md](../../../../docs/research/android/RIB_WORKFLOW.md)
→ **Advanced Navigation**: [docs/research/android/RIB_ROUTER_NAVIGATOR.md](../../../../docs/research/android/RIB_ROUTER_NAVIGATOR.md)
→ **Testing**: [docs/research/TESTING_GUIDE.md](../../../../docs/research/TESTING_GUIDE.md)
