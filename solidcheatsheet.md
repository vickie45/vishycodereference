# SOLID Cheatsheet — Quick Learning & Revision

Goal: concise SOLID principles, examples and applied guidance.

## S — Single Responsibility Principle
- One reason to change.
```csharp
public class OrderRepository { public Task Save(Order o) {} }
public class OrderService { public Task Place(OrderDto d) {} }
```

## O — Open/Closed Principle
- Open for extension, closed for modification. Use interfaces/strategy.

## L — Liskov Substitution Principle
- Subtypes must behave as the base type without surprises.

## I — Interface Segregation
- Small, focused interfaces:
```csharp
public interface IPrinter { void Print(); }
public interface IScanner { void Scan(); }
```

## D — Dependency Inversion
- Depend on abstractions; inject interfaces.
```csharp
public class Controller { public Controller(IRepo repo) { ... } }
```

## Practical refactors & common violations

### SRP applied
- Instead of:
```csharp
public class ReportManager { public void Generate() { /* data access, formatting, saving */ } }
```
- Refactor to services: ReportDataProvider, ReportFormatter, ReportRepository.

### DIP with factories and IoC
- Constructor injection:
```csharp
public class OrderController {
  private readonly IOrderService _svc;
  public OrderController(IOrderService svc) { _svc = svc; }
}
```
- Use factory when creating types at runtime:
```csharp
public interface IHandlerFactory { IHandler Create(string type); }
```

### Interface segregation example
- Avoid:
```csharp
public interface IMulti { void Print(); void Fax(); void Scan(); }
```
- Split into IPrinter, IFax, IScanner and implement only needed ones.

### When SOLID trips you
- Over-application can lead to excessive indirection. Balance clarity and maintainability.
