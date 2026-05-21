# rib-compiler-test Module Context

## Quick Summary
Testing support for annotation-processor-based code generation. Validates code generation correctness and provides test utilities for generated code.

## Location
`android/libraries/rib-compiler-test/`

## Main Purpose
- Test code generation correctness
- Validate generated RIB builders
- Verify component generation
- Test generated dependency injection
- Ensure generated code works correctly

## Key Components

### Code Generation Testing
- **CompilationTest** - Test source code compilation
- **GeneratedCodeValidator** - Validate generated files
- **BuilderTest** - Test builder generation
- **ComponentTest** - Test component generation

### Test Utilities
- **GeneratedCodeChecker** - Check generated code
- **CompilationAssertion** - Assert compilation results
- **GeneratedFileValidator** - Validate file contents

## Quick Patterns

```kotlin
// Test that code generation succeeds
@Test
fun testGenerateBuilderFromAnnotation() {
    val result = compile("""
        @GenerateRib
        class MyInteractor(val presenter: MyPresenter) : Interactor<MyPresenter, MyRouter>
    """)
    
    assertTrue(result.isSuccess)
    assertTrue(result.hasFile("MyBuilder.kt"))
}

// Verify generated builder correctness
@Test
fun testGeneratedBuilderPattern() {
    val generated = compile(mySourceCode).getFile("MyBuilder.kt")
    
    assertTrue(generated.contains("class MyBuilder"))
    assertTrue(generated.contains("fun build()"))
    assertTrue(generated.contains("MyRouter"))
}

// Test component generation
@Test
fun testComponentGeneration() {
    val result = compile(annotatedCode)
    
    assertTrue(result.hasFile("MyComponent.kt"))
    assertTrue(result.hasFile("MyModule.kt"))
}
```

## Integration Points
- **rib-compiler-app**: Tests compilation logic
- **rib-base**: Validates generated base classes
- **rib-test**: Uses test utilities

## Files to Know
- Compilation test infrastructure
- Code generation validators
- Test utilities for generated code

## Common Testing Tasks
- ✅ Builder generation → Test compilation and output
- ✅ Component generation → Verify Dagger components
- ✅ Error handling → Test compilation errors
- ✅ Code patterns → Verify generated patterns

## What Gets Tested

### Code Generation
- Source code compiles correctly
- Required files are generated
- Generated code is syntactically valid

### Generated Code Quality
- Generated classes extend correct bases
- Dependencies are properly injected
- Pattern conventions are followed

### Integration
- Generated builders can instantiate RIBs
- Components provide correct dependencies
- Lifecycle handling is preserved

## See Also
→ **Code Generation**: [rib-compiler-app](../rib-compiler-app/CONTEXT.md)
→ **Builder Pattern**: [docs/research/android/RIB_BASE.md](../../../../docs/research/android/RIB_BASE.md)
→ **Testing Guide**: [docs/research/TESTING_GUIDE.md](../../../../docs/research/TESTING_GUIDE.md)
