# .NET Core MVC Cheatsheet — Quick Learning & Revision

Goal: concise explanations, runnable snippets, best practices, and real-world use cases for ASP.NET Core MVC (6/7/8).

---

## Table of contents
- Quick setup & templates
- Program.cs & startup essentials
- Controllers & action results
- Routing (conventional & attribute)
- Views, Razor & Tag Helpers
- Model binding & validation
- ViewModels & DTOs
- Dependency Injection & services
- EF Core integration & patterns
- Filters (authorization, exception, action, resource)
- Authentication & Authorization
- Areas & modular apps
- Logging, diagnostics & correlation
- Testing (unit & integration)
- Deployment & hosting
- Best practices & real-world examples

---

## Quick setup & templates
- Create MVC app: dotnet new mvc -o MyMvcApp
- Add packages as needed (EF Core, Identity, AutoMapper, FluentValidation, Serilog).

---

## Program.cs & startup essentials
- Minimal hosting pattern:
```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews();
builder.Services.AddDbContext<AppDbContext>(opt => opt.UseSqlServer(conn));
builder.Services.AddAuthentication(/*...*/);
var app = builder.Build();
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllerRoute(name: "default", pattern: "{controller=Home}/{action=Index}/{id?}");
app.Run();
```

Best practice: keep configuration and wiring in Program.cs; move complex configuration to extension methods.

---

## Controllers & action results
- Controllers return IActionResult or specific ActionResult<T>.
```csharp
public class ProductsController : Controller {
  private readonly IProductService _svc;
  public ProductsController(IProductService svc) => _svc = svc;

  public async Task<IActionResult> Index() {
    var vm = await _svc.GetAllViewModelsAsync();
    return View(vm);
  }

  public async Task<IActionResult> Details(int id) {
    var dto = await _svc.GetAsync(id);
    if (dto == null) return NotFound();
    return View(dto);
  }

  [HttpPost]
  public async Task<IActionResult> Create(CreateProductModel model) {
    if (!ModelState.IsValid) return View(model);
    var created = await _svc.CreateAsync(model);
    return RedirectToAction(nameof(Details), new { id = created.Id });
  }
}
```

Best practice: controllers orchestrate only — keep business logic in services.

---

## Routing
- Conventional routing example (default): see MapControllerRoute above.
- Attribute routing:
```csharp
[Route("api/products")]
public class ApiProductsController : ControllerBase {
  [HttpGet("{id:int}")]
  public IActionResult Get(int id) => Ok();
}
```
- Use route constraints and names for clarity.

---

## Views, Razor & Tag Helpers
- Razor syntax:
```cshtml
@model IEnumerable<ProductViewModel>
@foreach(var p in Model) {
  <div>@p.Name - @p.Price.ToString("C")</div>
}
```
- Tag Helpers: <form asp-action="Create">, <a asp-route-id="@item.Id" asp-controller="Products">View</a>
- Partial views and ViewComponents for reusable UI logic.

Best practice: keep views simple, move logic to ViewComponents or services.

---

## Model binding & validation
- Model binding supports form fields, route, query, JSON bodies.
```csharp
public IActionResult Search([FromQuery] string q) { ... }
[HttpPost]
public IActionResult Save([FromForm] ProductModel m) { ... }
```
- Validation via DataAnnotations or FluentValidation; check ModelState.IsValid.
- Use IValidatableObject for cross-field validation.

---

## ViewModels & DTOs
- Use view-specific models to avoid over-posting and to shape data for UI.
```csharp
public class ProductViewModel { public int Id {get;set;} public string Name {get;set;} public decimal Price {get;set;} }
```
- Map between domain and view models with AutoMapper or manual mapping.

---

## Dependency Injection & services
- Register services with lifetimes:
```csharp
builder.Services.AddScoped<IProductService, ProductService>();
builder.Services.AddSingleton<ICacheService, MemoryCacheService>();
```
- Prefer constructor injection for testability.

---

## EF Core integration & patterns
- Use DbContext, async queries, AsNoTracking for reads.
- Repository vs direct DbContext: use repositories for abstraction where helpful; favor CQRS for complex apps.
- Example paging & projection:
```csharp
var page = await _context.Products
  .AsNoTracking()
  .OrderBy(p => p.Id)
  .Skip((page-1)*size)
  .Take(size)
  .Select(p => new ProductViewModel { Id = p.Id, Name = p.Name, Price = p.Price })
  .ToListAsync();
```

---

## Filters
- Global or action-level filters: AuthorizationFilter, ExceptionFilter, ActionFilter, ResourceFilter.
```csharp
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method)]
public class LogActionFilter : ActionFilterAttribute {
  public override void OnActionExecuting(ActionExecutingContext ctx) { /* ... */ }
}
```
- Use filters to centralize cross-cutting concerns.

---

## Authentication & Authorization
- Use ASP.NET Core Identity or JWT for APIs.
- Policy-based authorization:
```csharp
builder.Services.AddAuthorization(opts => {
  opts.AddPolicy("AdminOnly", p => p.RequireRole("Admin"));
});
[Authorize(Policy = "AdminOnly")]
public class AdminController : Controller { }
```
- Protect anti-forgery tokens in forms (ValidateAntiForgeryToken).

---

## Areas & modularization
- Use Areas to group big features (Admin, Sales). Configure area routes and controllers:
```csharp
[Area("Admin")]
public class DashboardController : Controller { public IActionResult Index() => View(); }
```

---

## Logging, diagnostics & correlation
- Use ILogger<T>. Add correlation ID middleware and include in logs.
- Use HealthChecks UI, Application Insights or OpenTelemetry for tracing.

---

## Testing
- Unit test controllers by mocking dependencies (xUnit, Moq).
- Integration tests using WebApplicationFactory<T> and TestServer to exercise middleware, routing and views.
- Razor view testing: use libraries or validate markup via integration tests.

---

## Deployment & hosting
- Serve static files from wwwroot. Use environment-specific appsettings and secrets management.
- Dockerfile (multi-stage) recommended. Host on Kestrel behind reverse proxy (Nginx/IIS/Azure App Service).
- Enable response caching, compression (middleware) and CDN for static assets.

---

## Performance & security best practices
- Use caching (ResponseCache, MemoryCache, DistributedCache).
- Use output caching (where safe), compression, and CDNs.
- Prevent XSS: encode outputs, use tag helpers and HtmlEncoder, avoid raw HTML in views.
- Prevent CSRF: verify antiforgery tokens for state-changing requests.
- Use HTTPS, HSTS, secure cookies, and validate user input server-side.

---

## Real-world examples & scenarios

1) Search page with server-side paging and debounce on client
- Server: endpoint returns paged results.
- Client: debounce inputs, request page when stable; update UI.

2) Admin area: secure feature module using Area + Policy
```csharp
[Area("Admin"), Authorize(Policy = "AdminOnly")]
public class UsersController : Controller { /* ... */ }
```

3) File upload with streaming
```csharp
[HttpPost]
public async Task<IActionResult> Upload(IFormFile file) {
  if (file.Length > 0) {
    using var stream = System.IO.File.Create(path);
    await file.CopyToAsync(stream);
  }
  return Ok();
}
```

4) Background tasks with IHostedService
```csharp
public class CleanupService : BackgroundService {
  protected override async Task ExecuteAsync(CancellationToken ct) {
    while (!ct.IsCancellationRequested) {
      await Task.Delay(TimeSpan.FromHours(1), ct);
      // perform cleanup
    }
  }
}
builder.Services.AddHostedService<CleanupService>();
```

5) Secure edit form (prevent over-posting)
```csharp
[HttpPost]
public async Task<IActionResult> Edit(int id, [Bind("Name,Price")] ProductEditModel model) { ... }
```

---

Keep this cheatsheet as a single-page quick reference for ASP.NET Core MVC; expand with project-specific patterns and snippets as you build features.
