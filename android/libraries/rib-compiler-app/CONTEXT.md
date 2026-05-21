# rib-compiler-app Module Context

## Quick Summary
Annotation processor and code generation tooling for RIBs. Generates boilerplate code for RIB builders and dependency injection.

## Location
`android/libraries/rib-compiler-app/`

## Main Purpose
- Generate RIB Builder scaffolding
- Create Dagger components
- Generate dependency injection boilerplate
- Reduce manual coding effort
- Ensure consistent builder patterns

## Key Components

### Code Generation
- **Builder Generator** - Auto-generate Builder classes
- **Component Generator** - Auto-generate Dagger components
- **Router Generator** - Auto-generate Router classes
- **Interactor Generator** - Auto-generate Interactor classes

### Annotation Processing
- **@GenerateRib** - Trigger RIB generation
- **@GenerateBuilder** - Generate builder specifically
- **@GenerateComponent** - Generate component

### Templates
- Builder patterns
- Component dependencies
- Lifecycle bindings

## Quick Patterns

```kotlin
// Use annotation to trigger code generation
@GenerateRib
class MyInteractor(val presenter: MyPresenter) : Interactor<MyPresenter, MyRouter>

// Generated code automatically creates:
// - MyBuilder (Builder implementation)
// - MyComponent (Dagger component)
// - MyInteractorModuleHello (Dagger module)
// - Proper dependency injection

// Usage of generated builder
val router: MyRouter = MyBuilder(parentComponent)
    .build(input: MyBundle)
```

## Integration Points
- **rib-base**: Generates code extending base classes
- **rib-android**: Generates View-based RIBs
- **rib-compiler-test**: Test code generation

## Generated Files
- `*Builder.kt` - Builder implementations
- `*Component.kt` - Dagger components
- `*Module.kt` - Dagger modules
- Supporting generated code

## Common Code Generation Tasks
- ✅ New RIB → Annotate Interactor with @GenerateRib
- ✅ Custom builder → @GenerateBuilder
- ✅ Component setup → Handled automatically
- ✅ Dependencies → Dagger injection

## Code Generation Benefits
- Reduces boilerplate code
- Ensures consistent patterns
- Catches errors at compile time
- Improves development speed

## See Also
→ **Builder Pattern**: [docs/research/android/RIB_BASE.md](../../../../docs/research/android/RIB_BASE.md)
→ **Dependency Injection**: [docs/research/ARCHITECTURE_OVERVIEW.md](../../../../docs/research/ARCHITECTURE_OVERVIEW.md)
