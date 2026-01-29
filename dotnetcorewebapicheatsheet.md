# .NET Core Web API Cheatsheet — Quick Learning & Revision

Goal: concise explanations, runnable snippets, best practices and real-world use cases for modern .NET (6/7/8) Web APIs.

---

## Table of contents
- Quick setup
- Minimal Program.cs (startup)
- Controllers & routing
- Models, DTOs & mapping
- Dependency Injection & services
- Middleware & global error handling
- EF Core & migrations
- Validation & FluentValidation
- Authentication & Authorization (JWT)
- Versioning, CORS, health checks
- Logging & correlation IDs
- Testing (unit & integration)
- Deployment (Docker, CI)
- Performance & security best practices
- Real-world examples / snippets

---

## Quick setup
- Create: dotnet new webapi -o MyApi
- Add packages commonly used:
  - Microsoft.EntityFrameworkCore.SqlServer
  - Microsoft.EntityFrameworkCore.Design
  - Swashbuckle.AspNetCore
  - Microsoft.AspNetCore.Authentication.JwtBearer
  - AutoMapper.Extensions.Microsoft.DependencyInjection
  - FluentValidation.AspNetCore
  - xunit / Moq for tests

---

## Minimal Program.cs (recommended for .NET 6+)
- Register services, middlewares, controllers and run app.
```csharp
// Minimal Program.cs
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
// DI, DB, Auth, CORS, HealthChecks...
builder.Services.AddDbContext<AppDbContext>(opt => opt.UseSqlServer(builder.Configuration.GetConnectionString("Default")));
builder.Services.AddScoped<IItemRepository, ItemRepository>();
builder.Services.AddAutoMapper(typeof(Program));
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme).AddJwtBearer(opt => { /* configure */ });

var app = builder.Build();
if (app.Environment.IsDevelopment()) { app.UseSwagger(); app.UseSwaggerUI(); }
app.UseHttpsRedirection();
app.UseMiddleware<ErrorHandlingMiddleware>();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

Best practice: keep Program.cs small — wire up services and delegate logic to modules/services.

---

## Controllers & routing
- Use [ApiController] and conventional routing.
- Return IActionResult or typed ActionResult<T>.
```csharp
[ApiController]
[Route("api/[controller]")]
public class ItemsController : ControllerBase {
  private readonly IItemRepository _repo;
  public ItemsController(IItemRepository repo) => _repo = repo;

  [HttpGet]
  public async Task<ActionResult<IEnumerable<ItemDto>>> GetAll() {
    var items = await _repo.GetAllAsync();
    return Ok(items);
  }

  [HttpGet("{id:int}", Name = "GetItem")]
  public async Task<ActionResult<ItemDto>> Get(int id) {
    var item = await _repo.GetAsync(id);
    if (item == null) return NotFound();
    return Ok(item);
  }

  [HttpPost]
  public async Task<ActionResult> Create(CreateItemDto dto) {
    var created = await _repo.CreateAsync(dto);
    return CreatedAtRoute("GetItem", new { id = created.Id }, created);
  }
}
```

Best practice: use ActionResult<T> for clear responses and CreatedAtRoute for POST.

---

## Models, DTOs & mapping
- Never expose EF entities directly; use DTOs.
- AutoMapper or manual mapping.
```csharp
public class Item { public int Id {get;set;} public string Name {get;set;} }
public class ItemDto { public int Id {get;set;} public string Name {get;set;} }
public class CreateItemDto { public string Name {get;set;} }

// AutoMapper Profile
public class MappingProfile : Profile {
  public MappingProfile() { CreateMap<Item, ItemDto>(); CreateMap<CreateItemDto, Item>(); }
}
```

---

## Dependency Injection & services
- Register services with appropriate lifetimes:
  - Singleton: long-lived, thread-safe (e.g., configuration cache)
  - Scoped: per-request (DbContext, repositories)
  - Transient: lightweight, stateless
```csharp
builder.Services.AddScoped<IItemRepository, ItemRepository>();
```

---

## Middleware & global error handling
- Centralize error handling and logging in middleware.
```csharp
public class ErrorHandlingMiddleware {
  private readonly RequestDelegate _next;
  public ErrorHandlingMiddleware(RequestDelegate next) => _next = next;
  public async Task InvokeAsync(HttpContext ctx, ILogger<ErrorHandlingMiddleware> log) {
    try { await _next(ctx); }
    catch(Exception ex) {
      log.LogError(ex, "Unhandled error");
      ctx.Response.ContentType = "application/json";
      ctx.Response.StatusCode = 500;
      await ctx.Response.WriteAsJsonAsync(new { error = "An unexpected error occurred" });
    }
  }
}
```

---

## EF Core & migrations
- Add DbContext and register.
```csharp
public class AppDbContext : DbContext {
  public DbSet<Item> Items { get; set; }
  public AppDbContext(DbContextOptions<AppDbContext> opt) : base(opt) {}
}
```
- Migrations:
  - dotnet ef migrations add InitialCreate
  - dotnet ef database update

Best practice: keep DbContext small; use DTOs; avoid heavy business logic in DbContext.

---

## Validation & FluentValidation
- Validate incoming models using DataAnnotations or FluentValidation.
```csharp
public class CreateItemDtoValidator : AbstractValidator<CreateItemDto> {
  public CreateItemDtoValidator() {
    RuleFor(x => x.Name).NotEmpty().MaximumLength(200);
  }
}
```
- Register FluentValidation in DI and integrate with MVC.

---

## Authentication & Authorization (JWT)
- Configure JWT authentication and Authorization policies.
```csharp
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
  .AddJwtBearer(options => { options.TokenValidationParameters = new TokenValidationParameters { /* issuer, key, audience */ }; });

[Authorize(Roles = "Admin")]
public class AdminController : ControllerBase { /* ... */ }
```

Best practice: validate tokens, refresh securely, store secrets in Key Vault or env vars.

---

## Versioning, CORS, Health Checks
- API versioning:
  - Add package Microsoft.AspNetCore.Mvc.Versioning and configure.
- CORS:
```csharp
builder.Services.AddCors(opt => opt.AddPolicy("default", p => p.WithOrigins("https://app.example.com").AllowAnyHeader().AllowAnyMethod()));
app.UseCors("default");
```
- Health Checks: builder.Services.AddHealthChecks(); app.MapHealthChecks("/health");

---

## Logging & correlation IDs
- Use ILogger<T> everywhere.
- Add middleware to attach Correlation-Id header and log with scopes.
```csharp
using (logger.BeginScope(new Dictionary<string, object>{{"CorrelationId", corrId}})) { /* ... */ }
```

---

## Testing
- Unit tests: xUnit + Moq for services and controllers (mock repositories).
- Integration tests: WebApplicationFactory<TEntryPoint> to spin up in-memory host; use in-memory DB or test container.
```csharp
public class ItemsControllerTests : IClassFixture<WebApplicationFactory<Program>> {
  private readonly HttpClient _client;
  public ItemsControllerTests(WebApplicationFactory<Program> factory) => _client = factory.CreateClient();
  [Fact] public async Task GetAll_ReturnsOk() { var res = await _client.GetAsync("/api/items"); res.EnsureSuccessStatusCode(); }
}
```

---

## Deployment (Docker & CI)
- Simple Dockerfile (multi-stage)
```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY . .
RUN dotnet publish -c Release -o /app

FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app .
ENTRYPOINT ["dotnet", "MyApi.dll"]
```
- CI: run dotnet restore, build, test, docker build/push.

---

## Performance & security best practices
- Use async/await for I/O; avoid blocking calls.
- Paginate heavy queries; use projections (Select) rather than returning full entities.
- Protect from over-posting: bind DTOs only to allowed fields.
- Rate limit, enable HTTPS, use HSTS, secure cookies, validate inputs, sanitize outputs.
- Keep dependencies up to date.

---

## Real-world snippets

1) Repository pattern (simple)
```csharp
public interface IItemRepository {
  Task<IEnumerable<ItemDto>> GetAllAsync();
  Task<ItemDto> GetAsync(int id);
  Task<ItemDto> CreateAsync(CreateItemDto dto);
}
```

2) Cancellation support in controllers
```csharp
[HttpGet]
public async Task<ActionResult> GetAll(CancellationToken ct) {
  var items = await _repo.GetAllAsync(ct);
  return Ok(items);
}
```

3) Paging & projection with EF Core
```csharp
var items = await _context.Items
  .AsNoTracking()
  .Where(i => i.IsActive)
  .OrderBy(i => i.Id)
  .Skip((page-1)*pageSize)
  .Take(pageSize)
  .Select(i => new ItemDto { Id = i.Id, Name = i.Name })
  .ToListAsync();
```

4) Graceful shutdown & DB connection cleanup handled by generic host automatically.

---

## Further reading / next steps
- Deep dive: middleware pipeline, advanced EF Core performance, distributed tracing (OpenTelemetry), resilient networking (Polly), secure token handling.
- Explore Microsoft docs for security guidelines and Azure deployment patterns.

---

Keep this file as your .NET Core Web API single-page quick reference; add project-specific snippets as you build real services.
