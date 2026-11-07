Comprehensive testing patterns for .NET applications using xUnit, and modern testing practices.

## Test Framework Stack

| Tool | Purpose |
|---|---|
| **xUnit v3** | Test framework (preferred for .NET) |
| **Moq** | Mocking dependencies |
| **WebApplicationFactory** | ASP.NET Core integration tests |

## Running Tests

```bash
# Run all tests
dotnet test

# Run with coverage
dotnet test 
  --configuration Release \
  --verbosity normal \
  --results-directory TestResults \
  --coverage \
  --coverage-output-format cobertura,xml

# Run specific project
dotnet test tests/MyApp.UnitTests/

# Filter by test name
dotnet test --filter "FullyQualifiedName~OrderService"

# Watch mode during development
dotnet watch test --project tests/MyApp.UnitTests/
```
