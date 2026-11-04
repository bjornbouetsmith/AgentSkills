## Mocking with moq

### Hard rules
- *Do not* mock the class under test. Only mock dependencies.
- *Do not* use strict mocks. Use loose mocks instead.
- *Do not* use `VerifyNoOtherCalls()` or `VerifyAll()`. These are brittle and unnecessary.
- Only setup mocks for the specific behavior that is relevant to the test. Do not setup every method on a mock. Rely on the default behavior of loose mocks for methods that are not relevant to the test.

```csharp
// BAD causes brittle tests that break if new methods are called on the interface
var mockDependency = new Mock<IDependency>(MockBehavior.Strict);
```

```csharp
// Good
var mockDependency = new Mock<IDependency>(MockBehavior.Loose);
```

```csharp
var mockDependency = new Mock<IDependency>();
mockDependency.Setup(d => d.MethodA()).Returns(42);

mockDependency.Verify(d => d.MethodA(), Times.Once);

// BAD causes brittle tests that break if new methods are called on the interface
mockDependency.VerifyNoOtherCalls();
```