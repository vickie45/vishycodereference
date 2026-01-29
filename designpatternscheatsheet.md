# Design Patterns Cheatsheet — Quick Learning & Revision

Goal: concise patterns, intent, minimal examples, and when to use.

## Creational
- Singleton: single instance (prefer DI singletons).
- Factory Method / Abstract Factory: delegate creation.
- Builder: step-by-step construction.
- Prototype: clone objects.

Example (Factory):
```csharp
public interface IParser { void Parse(); }
public class JsonParser : IParser { public void Parse() {} }
public static class ParserFactory { public static IParser Create(string type) => type=="json"? new JsonParser() : null; }
```

## Structural
- Adapter, Facade, Decorator, Proxy, Composite
- Facade simplifies subsystems.

## Behavioral
- Strategy, Observer, Command, Chain of Responsibility, State, Template Method
- Strategy example:
```csharp
public interface IDiscount { decimal Apply(decimal total); }
public class SeasonalDiscount : IDiscount { public decimal Apply(decimal total) => total * 0.9m; }
```

## More patterns & guidance

### Repository + Unit of Work
- Repository abstracts data access; Unit of Work groups operations in a transaction.
```csharp
public interface IUnitOfWork { Task<int> CommitAsync(); }
public interface IRepository<T> { Task<T> GetAsync(int id); }
```
- Use with DI and EF Core DbContext as UoW (DbContext implements UoW behavior).

### Adapter & Decorator examples
- Adapter: translate interface for legacy libs.
- Decorator: add behavior (logging, caching) without modifying class:
```csharp
public class CachingRepo : IRepository {
  private readonly IRepository _inner; public async Task<Item> Get(int id) {
    var cached = _cache.Get(id); if (cached != null) return cached;
    var item = await _inner.Get(id); _cache.Set(id, item); return item;
  }
}
```

### Anti-patterns
- Overuse of Singleton for mutable state — leads to hidden coupling.
- God objects that break SRP — split responsibilities early.

### When to use
- Choose pattern by problem: creation issues -> factory/builder; behavioral flexibility -> strategy/command; composition of behavior -> decorator/proxy.

- Prefer composition over inheritance.
- Use patterns to communicate intent; avoid overengineering.
