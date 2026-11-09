---
name: csharp-testing
description: "Use when c# and .NET testing patterns with xUnit v3, mocking, integration tests, and test organization best practices. Only for C#/.NET — not for other languages. Triggers on \"csharp-testing\", \"csharp testing\", \"testing\"."
---

# C# Testing Patterns

Comprehensive testing patterns for .NET applications using xUnit, Assert, and modern testing practices.

## When to Activate

- Writing new tests for C# code
- Reviewing test quality and coverage
- Setting up test infrastructure for .NET projects
- Debugging flaky or slow tests

## Test Framework Stack

| Tool | Purpose |
|---|---|
| **xUnit** | Test framework (preferred for .NET) |
| **Moq** | Mocking dependencies |
| **WebApplicationFactory** | ASP.NET Core integration tests |

## Unit Test Structure

### Arrange-Act-Assert

```csharp
public sealed class OrderServiceTests
{
    private readonly Mock<IOrderRepository> _repository = new Mock<IOrderRepository>();
    private readonly ILogger<OrderService> _logger = new Mock<ILogger<OrderService>>();
    private readonly OrderService _orderService;

    public OrderServiceTests()
    {
        _orderService = new OrderService(_repository.Object, _logger.Object);
    }

    [Fact]
    public async Task PlaceOrderAsync_ReturnsSuccess_WhenRequestIsValid()
    {
        var request = new CreateOrderRequest
        {
            CustomerId = "cust-123",
            Items = [new OrderItem("SKU-001", 2, 29.99m)]
        };

        var result = await _orderService.PlaceOrderAsync(request, CancellationToken.None);

        Assert.True(result.IsSuccess);
        Assert.NotNull(result.Value);
        Assert.Equal("cust-123", result.Value!.CustomerId);
    }
}
```

### Determinism and isolation

New tests must produce the same result on every machine and every run.

- Do not use `Guid.NewGuid()`, `DateTime.Now`, `DateTime.UtcNow`, random values,
  or other changing input when asserting output. Use a fixed value, for example
  `Guid.Parse("11111111-2222-3333-4444-555555555555")` or an explicit
  `DateTime` with an explicit `DateTimeKind`.
- Formatting code may depend on the current culture. When asserting formatted
  strings, pin `CultureInfo.CurrentCulture = CultureInfo.InvariantCulture` and
  restore the original culture in a `finally` block. The same applies to
  time-zone-sensitive values.
- Do not depend on live SSH hosts, IPMI endpoints, network services, or external
  commands in a unit test. If a LIVE something is required try to write the test to use a docker container that provides the necessary functionality.
- Use `Path.GetTempPath()` for file-system tests and delete temporary files and
  directories in a `finally` block. 
- Platform-specific behavior must be guarded. Use
  `RuntimeInformation.IsOSPlatform(...)`; use `Assert.Inconclusive` if the test
  cannot run on the current platform so the suite reports it as not applicable
  rather than silently passing.
- Tests that require external infrastructure like docker containers should be marked with a trait or category `IntegrationTest`
  or moved to an explicitly
  separate integration-test suite. Do not add new live-service tests to the
  normal unit-test path.

### patterns to use

#### XUnit
For xUnit, use `[Fact]` for simple tests and `[Theory]` with `[InlineData]` for parameterized tests:

```csharp
[Fact]
public void ProcessCallExecuteTest()
{
    var sut = new Something(...);
}

[Theory]
[InlineData(true)]
[InlineData(false)]
public void ProcessCallExecuteTest(bool useShell)
{
    var sut = new Something(...);
}
```

Complex output should be asserted exactly rather than printed:

```csharp
string result = value.Dump(new JsonFormatter());

Assert.AreEqual("{\"Name\":\"test\",\"Count\":5,\"Enabled\":true}", result);
```

When fixing a test that previously used non-deterministic data, replace the
data with a fixed value and assert the resulting output exactly.


## Coverage expectations

- Cover the public API and overloads that changed, including the non-generic and
  generic variants when both exist.
- Include null, empty/default, nested/complex, collection, and invalid-argument
  cases where they are meaningful.
- Test caching, factory selection, and other stateful behavior explicitly.
- Do not write tests that only exercise framework behavior or mirror the
  implementation line by line.
- Aim for at least 80% coverage

### Testing complete checklist

1. Every new or changed test has at least one meaningful assertion.
2. The test is deterministic and does not depend on the current time, culture,
   random values, or external services.
3. Temporary files and resources are cleaned up.
4. The focused tests pass.
5. The full test suite passes.
6. No //Arrange, //Act, //Assert comments remain in the code.
7. No code analyzer warnings from either xunit or other anylyzers present in tests introduced
