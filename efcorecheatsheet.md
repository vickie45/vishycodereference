# EF Core Cheatsheet — Quick Learning & Revision

Goal: concise EF Core patterns, snippets, best practices and real-world examples.

## Setup
- Packages: Microsoft.EntityFrameworkCore.SqlServer, Microsoft.EntityFrameworkCore.Design
- Tools: dotnet-ef CLI (dotnet tool install --global dotnet-ef)

## DbContext & models
```csharp
public class AppDbContext : DbContext {
  public DbSet<Product> Products { get; set; }
  public AppDbContext(DbContextOptions<AppDbContext> opts) : base(opts) {}
}
public class Product { public int Id {get;set;} public string Name {get;set;} public decimal Price {get;set;} }
```

## Migrations
- Add: dotnet ef migrations add Initial
- Apply: dotnet ef database update
- Best: review and keep migrations small.

## Querying
- LINQ + EF:
```csharp
var cheap = await ctx.Products.Where(p => p.Price < 10).ToListAsync();
var dto = await ctx.Products.Where(...).Select(p => new ProductDto { Id = p.Id, Name = p.Name }).ToListAsync();
```
- Use AsNoTracking() for read-only queries.

## Change tracking & updates
- Attach & modify:
```csharp
ctx.Attach(p).State = EntityState.Modified;
await ctx.SaveChangesAsync();
```
- Prefer explicit property updates to avoid over-posting.

## Relationships & loading
- Configure via Fluent API or attributes; use Include/ThenInclude for eager loading:
```csharp
var order = await ctx.Orders.Include(o => o.Items).ThenInclude(i => i.Product).FirstOrDefaultAsync(id);
```
- Avoid N+1 queries.

## Performance & patterns
- Use projection, pagination (Skip/Take), batching, compiled queries.
- Use Value Converters, owned types, and AsNoTracking where appropriate.

## Transactions & concurrency
- Use explicit transactions for multi-step operations:
```csharp
using var tx = await ctx.Database.BeginTransactionAsync();
await ctx.SaveChangesAsync();
await tx.CommitAsync();
```
- Use RowVersion (byte[]) for optimistic concurrency.

## Testing
- Use SQLite in-memory or InMemory provider for tests; prefer SQLite for closer SQL behavior.

## Real-world example
- Upsert:
```csharp
var existing = await ctx.Products.FirstOrDefaultAsync(p => p.Sku == sku);
if (existing == null) ctx.Products.Add(new Product{Sku=sku, Name=name});
else existing.Price = price;
await ctx.SaveChangesAsync();
```

## Advanced topics & deep-dive

### Change tracking behaviors
- Track vs no-track:
  - ChangeTracker.QueryTrackingBehavior = QueryTrackingBehavior.NoTracking for read-mostly contexts to reduce memory.
  - Use AsNoTrackingWithIdentityResolution() when you need identity resolution without full tracking.
- DetectChanges:
  - Avoid frequent automatic DetectChanges in tight loops; call ChangeTracker.DetectChanges() explicitly when needed.

### Raw SQL & performance
- Execute raw SQL for complex queries not supported by LINQ:
```csharp
var results = await ctx.Products.FromSqlRaw("SELECT * FROM dbo.Products WHERE Price < {0}", 10).ToListAsync();
await ctx.Database.ExecuteSqlRawAsync("UPDATE dbo.Products SET Price = Price * 1.1 WHERE CategoryId = {0}", catId);
```
- Parameterize raw SQL to avoid injection; prefer FromSqlInterpolated for safety.

### Compiled queries
- Use compiled queries in hot paths to reduce LINQ-to-SQL overhead:
```csharp
private static readonly Func<AppDbContext, decimal, Task<List<Product>>> _cheapQuery =
  EF.CompileAsyncQuery((AppDbContext c, decimal max) => c.Products.Where(p => p.Price < max).ToList());
var cheap = await _cheapQuery(ctx, 10m);
```

### Global query filters & soft delete
- Configure once to exclude soft-deleted rows:
```csharp
modelBuilder.Entity<Entity>().HasQueryFilter(e => !e.IsDeleted);
```
- Remember filters apply to all queries unless explicitly ignored (IgnoreQueryFilters()).

### Shadow properties & owned types
- Shadow properties hold DB-only values:
```csharp
modelBuilder.Entity<Product>().Property<DateTime>("CreatedAt");
```
- Owned types for value objects:
```csharp
modelBuilder.Entity<Order>().OwnsOne(o => o.ShippingAddress);
```

### Migrations & schema management
- Keep migration commits small and descriptive.
- Use idempotent scripts for environments where direct migrations are not allowed.
- For large backfills: add nullable column, backfill in batches, then make column non-nullable.

### Concurrency
- RowVersion:
```csharp
public byte[] RowVersion { get; set; }
modelBuilder.Entity<Entity>().Property(e => e.RowVersion).IsRowVersion();
```
- Handle DbUpdateConcurrencyException and implement merge/resolve strategy.
