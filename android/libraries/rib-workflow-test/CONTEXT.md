# rib-workflow-test Module Context

## Quick Summary
Testing utilities and test helpers for workflow state machines. Enables testing of multi-step workflows, conditional branching, and error handling scenarios.

## Location
`android/libraries/rib-workflow-test/`

## Main Purpose
- Provide workflow test utilities and helpers
- Support testing of workflow steps
- Enable workflow output validation
- Test error and cancellation flows
- Verify step transitions

## Key Components

### Workflow Testing
- **WorkflowTestObserver** - Observe workflow execution
- **WorkflowTestRunner** - Execute and validate workflows
- **StepTestDouble** - Mock workflow steps
- **TestObservable** - Controlled observable for testing

### Step Testing
- **StepEmulator** - Simulate step execution
- **StepValidator** - Validate step output
- **ConditionalBranchTester** - Test conditional logic

## Quick Patterns

```kotlin
// Test Workflow Completion
@Test
fun testWorkflowCompletion() {
    val observer = workflow.invoke(Unit).test()
    
    emailStep.complete("user@example.com")
    passwordStep.complete("password123")
    
    observer.assertComplete()
    observer.assertValue { it is SignupResult.Success }
}

// Test Conditional Branching
@Test
fun testConditionalBranching() {
    val observer = workflow.invoke(Unit).test()
    
    selectType.emit(UserType.DRIVER)
    
    observer.assertValue { it is DriverOnboardingResult }
}

// Test Error Handling
@Test
fun testErrorRecovery() {
    val observer = workflow.invoke(Unit).test()
    
    paymentStep.error(NetworkError())
    retryStep.complete(paymentInfo)
    
    observer.assertComplete()
}

// Test Cancellation
@Test
fun testWorkflowCancellation() {
    val observer = workflow.invoke(Unit).test()
    
    workflow.cancel()
    
    observer.assertValue { it is WorkflowResult.Cancelled }
}

// Test Step Sequence
@Test
fun testStepSequence() {
    val stepOrder = mutableListOf<String>()
    workflow.onStepExecuted { step -> stepOrder.add(step.name) }
    
    workflow.invoke(Unit).test()
    
    assertEquals(listOf("email", "password", "confirm"), stepOrder)
}
```

## Integration Points
- **rib-workflow**: Core workflow implementations
- **rib-test**: Base testing utilities
- **rib-android**: View-based workflow testing

## Files to Know
- `com.uber.rib.workflow.WorkflowTestObserver` - Workflow observer
- `com.uber.rib.workflow.WorkflowTestRunner` - Workflow executor

## Common Testing Tasks

### Test Sequential Steps
```kotlin
@Test
fun testSequentialFlow() {
    // Verify steps execute in order
}
```

### Test Error Handling
```kotlin
@Test
fun testErrorRecovery() {
    // Verify error handling and recovery
}
```

### Test Cancellation
```kotlin
@Test
fun testUserCancellation() {
    // Verify cancellation works
}
```

### Test Step Output
```kotlin
@Test
fun testStepOutput() {
    // Verify step output values
}
```

## See Also
→ **Workflow Documentation**: [docs/research/android/RIB_WORKFLOW.md](../../../../docs/research/android/RIB_WORKFLOW.md)
→ **Testing Guide**: [docs/research/TESTING_GUIDE.md](../../../../docs/research/TESTING_GUIDE.md)
→ **Testing Utilities**: [rib-test](../rib-test/CONTEXT.md)
