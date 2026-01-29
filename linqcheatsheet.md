# LINQ Cheatsheet — Quick Learning & Revision

Goal: concise LINQ operators, examples, execution model and best practices.

## Basics
- Query and method syntax:
```csharp
var q = from p in products where p.Price < 10 select p;
var m = products.Where(p => p.Price < 10).Select(p => p.Name);
```
- Deferred execution: queries run when enumerated.

## Common operators
- Filtering: Where
- Projection: Select, SelectMany
- Ordering: OrderBy, ThenBy
- Aggregates: Count, Sum, Average, Min, Max
- Partition: Skip, Take
- Element: First/FirstOrDefault, Single/SingleOrDefault

## Joins & Grouping
```csharp
var j = from c in customers join o in orders on c.Id equals o.CustomerId select new {c, o};
var g = orders.GroupBy(o => o.CustomerId).Select(g => new { CustomerId = g.Key, Total = g.Sum(x => x.Total) });
```

## Performance tips
- Prefer projection to limit transferred data.
- Materialize with ToList/ToArray before multiple enumerations.
- For EF use IQueryable so operators translate to SQL; avoid client-side evaluation.

## Parallel & async
- PLINQ: AsParallel() for CPU-bound.
- Async: use ToListAsync, FirstOrDefaultAsync with EF Core.

## Real-world example
- Top-selling product ids:
```csharp
var top = orders.SelectMany(o => o.Items)
  .GroupBy(i => i.ProductId)
  .OrderByDescending(g => g.Sum(x => x.Quantity))
  .Take(10)
  .Select(g => new { ProductId = g.Key, Qty = g.Sum(x=>x.Quantity) })
  .ToList();
```

## Deeper concepts

### IQueryable vs IEnumerable
- IQueryable builds an expression tree that LINQ providers (EF) translate to SQL; use for remote queries.
- IEnumerable executes in-memory; good for local collections after materialization.

### Expression Trees vs Delegates
- Expression<Func<T, bool>> can be inspected/transformed by providers.
- Func<T,bool> is a compiled delegate executed in-memory.

### Client vs Server evaluation
- For EF, prefer expressions that can translate to SQL. Client evaluation pulls data into memory and may cause large transfers.
- Check warnings about client evaluation in logs; rewrite or pre-load needed data.

### Useful advanced operators
- SelectMany for flattening collections:
```csharp
var allItems = orders.SelectMany(o => o.Items);
```
- Aggregate for custom reductions:
```csharp
var checksum = items.Aggregate(0, (acc, i) => acc ^ i.Id);
```
- DistinctBy (NET 6+ or via GroupBy fallback):
```csharp
var unique = items.GroupBy(i => i.Key).Select(g => g.First());
```

### Performance tips
- Avoid repeated enumeration; materialize with ToList when needed.
- Use indexed access for arrays instead of LINQ in hot loops.
- Use caching for expensive queries when data is stable.
