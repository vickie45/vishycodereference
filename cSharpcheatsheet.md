# C# Cheatsheet — Quick Learning & Revision

Goal: concise explanations, runnable code snippets, best practices, and real-world use cases.

---

## Table of contents
- Basics & environment
- Syntax & types
- Variables, scope & nullability
- Control flow
- Methods & pattern matching
- OOP: classes, records, structs, interfaces
- Properties, indexers & events
- Generics, delegates & lambdas
- Collections & LINQ
- Async/await & concurrency
- Memory & performance (span, ref, stackalloc)
- Error handling & disposal
- Interop & unsafe
- Tooling, testing & deployment
- Best practices & real-world snippets

---

## Basics & environment
- Run: dotnet new console -o App; dotnet run
- Project: .csproj controls SDK, target framework, package refs.
- Useful CLI: dotnet build, dotnet test, dotnet publish, dotnet add package

---

## Syntax & types
- Primitive types: int, long, float, double, decimal, bool, char, string
- Nullable reference types (C# 8+): enable in csproj -> <Nullable>enable</Nullable>
```csharp
int i = 0;
string? maybe = null; // nullable reference
decimal price = 9.99m;
```

---

## Variables, scope & nullability
- var for type inference, explicit types allowed.
- Prefer readonly for fields with fixed value.
- Use null-coalescing operators: ?? and ??=; null-conditional ?. and pattern matching is and is not.
```csharp
var s = maybe ?? "default";
obj?.Method();
```

---

## Control flow
- if/else, switch expressions, for, foreach, while, do/while
- Switch expression (concise):
```csharp
var result = value switch {
  1 => "one",
  > 1 => "many",
  _ => "none"
};
```

---

## Methods & pattern matching
- Local functions, default parameters, params, ref/out.
- Pattern matching in switch and is:
```csharp
int Add(int a, int b = 1) => a + b;
bool IsPoint(object o) => o is Point { X: > 0, Y: > 0 };
```

---

## OOP: classes, records, structs, interfaces
- Class: reference type, supports inheritance.
- Record: immutable by default, value-based equality (C# 9+).
- Struct: value types, use for small, immutable types.
```csharp
public class Person { public string Name { get; set; } }
public record PersonR(string Name);
public struct Point { public int X; public int Y; }
```
- Interface and explicit implementation:
```csharp
public interface IRepository<T> { Task<T> Get(int id); }
public class Repo : IRepository<Item> { public Task<Item> Get(int id) => ...; }
```

---

## Properties, indexers & events
- Auto-properties, init-only setters, expression bodies, indexers:
```csharp
public string Name { get; init; } // immutable initialization
public int this[int index] => _arr[index];
public event EventHandler? Changed;
```

---

## Generics, delegates & lambdas
- Generics for type-safe containers:
```csharp
public class Box<T> { public T Value { get; set; } }
Func<int,int> sq = x => x*x;
Action<string> log = s => Console.WriteLine(s);
```
- Covariance/contravariance: use in interfaces/delegates where applicable.

---

## Collections & LINQ
- Arrays, List<T>, Dictionary<TKey,TValue>, HashSet<T>, Queue/Stack.
- LINQ for querying:
```csharp
var evens = numbers.Where(n => n % 2 == 0).Select(n => n * 2).ToList();
var top = items.OrderByDescending(i => i.Score).Take(10);
var grouped = items.GroupBy(i => i.Category).ToDictionary(g => g.Key, g => g.ToList());
```
- Use AsEnumerable/AsQueryable when mixing LINQ-to-Objects and LINQ-to-Entities.

---

## Async/await & concurrency
- Task, Task<T>, ValueTask<T>, async/await. Avoid async void except event handlers.
- CancellationToken for cancelling operations.
```csharp
async Task<Data> GetAsync(string url, CancellationToken ct) {
  using var client = new HttpClient();
  var res = await client.GetAsync(url, ct);
  res.EnsureSuccessStatusCode();
  return await res.Content.ReadFromJsonAsync<Data>(cancellationToken: ct);
}
```
- Parallelism: Parallel.ForEach, Task.WhenAll, channels, TPL Dataflow for pipelines.
- Use ConfigureAwait(false) in libraries to avoid capturing context.

---

## Memory & performance
- Use Span<T>, ReadOnlySpan<T>, Memory<T> for zero-allocation slicing.
- ref returns and ref locals for performance-critical code.
- stackalloc for allocating on stack:
```csharp
Span<int> buffer = stackalloc int[128];
```
- Avoid allocations in hot paths; use pooling (ArrayPool<T>), StringBuilder for heavy string work.

---

## Error handling & disposal
- Exceptions: throw, try/catch/finally, use specific exceptions.
- IDisposable and using / await using:
```csharp
using var conn = new SqlConnection(connStr);
await using var stream = await file.OpenReadAsync();
```
- Implement IAsyncDisposable for async cleanup.

---

## Interop & unsafe
- P/Invoke with DllImport, use Marshal where needed.
- unsafe code for pointers if necessary; prefer safe APIs.

---

## Tooling, testing & deployment
- Unit testing: xUnit / NUnit / MSTest; mocking: Moq, NSubstitute
- Profiling: dotnet-counters, dotnet-trace, Visual Studio Profiler
- Code style: EditorConfig, analyzers (Roslyn), nullable annotations
- Packaging: NuGet (dotnet pack), CI with GitHub Actions/Azure DevOps
- Publish: dotnet publish -c Release -r <runtime> --self-contained

---

## Best practices
- Prefer immutability and small methods.
- Favor composition over inheritance.
- Keep async code free of blocking calls (avoid Task.Result/Wait).
- Keep exceptions exceptional — use Result/Option patterns for expected failures.
- Use DI for testability; register lifetimes appropriately (singleton/scoped/transient).
- Enable nullable reference types and address warnings.

---

## Real-world examples & snippets

1) Simple Web call with retry (polly):
```csharp
var policy = Policy.Handle<HttpRequestException>().WaitAndRetryAsync(3, i => TimeSpan.FromSeconds(2*i));
await policy.ExecuteAsync(() => httpClient.GetAsync(url));
```

2) Minimal DI + background service (worker):
```csharp
public class Worker : BackgroundService {
  private readonly ILogger<Worker> _log;
  public Worker(ILogger<Worker> log) => _log = log;
  protected override async Task ExecuteAsync(CancellationToken ct) {
    while (!ct.IsCancellationRequested) {
      _log.LogInformation("tick");
      await Task.Delay(TimeSpan.FromSeconds(5), ct);
    }
  }
}
```

3) LINQ + projection for DTO:
```csharp
var dtos = await _context.Orders
  .Where(o => o.Date >= since)
  .Select(o => new OrderDto { Id = o.Id, Total = o.Items.Sum(i => i.Price * i.Qty) })
  .ToListAsync();
```

4) Safe JSON parse:
```csharp
T? TryParseJson<T>(string json) {
  try { return JsonSerializer.Deserialize<T>(json); } catch { return default; }
}
```

5) Value-based equality with record:
```csharp
public record Money(decimal Amount, string Currency);
```

---

## Quick reference one-liners
- Null-coalescing assign: x ??= GetDefault();
- Pattern matching: if (obj is Person { Age: > 18 } p) ...
- Create list: var list = new List<int>{1,2,3};
- Async parallel: var results = await Task.WhenAll(tasks);

---

## Further reading / next steps
- Deep dive: Roslyn analyzers, advanced memory patterns (Span/Memory), source generators, high-performance C#.
- Learn patterns: CQRS, DDD, reactive streams (System.Threading.Channels), gRPC interop.

---

Keep this file as your single-page C# quick reference and expand with project-specific snippets as needed.
