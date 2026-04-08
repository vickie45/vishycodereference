This tutorial will guide you through building a fully functional Blog API with ASP.NET Core 8, LINQ, and Entity Framework Core (EF Core) using SQL Server. You'll learn how to use LINQ for database operations, set up relationships, and implement all CRUD endpoints for a complete backend solution.

---

1. What is EF Core?

Entity Framework Core is an Object-Relational Mapper (ORM) that allows you to interact with a database using C# objects instead of writing raw SQL queries. With the Code-First approach, you define your database schema using C# classes, and EF Core automatically creates the database structure and handles data access for you.

For data querying, EF Core uses Language-Integrated Query (LINQ), which lets you write strongly-typed queries in C# that are then translated into SQL and executed against the database.

2. Project Setup

Step 1: Create a new project

```bash
dotnet new webapi -n BlogApi
cd BlogApi
```

Alternatively, in Visual Studio: Create a new project → ASP.NET Core Web API → Name it BlogApi → Use .NET 8.0 → Ensure Use controllers is selected (not minimal APIs).

Step 2: Install required NuGet packages

Run these commands to install EF Core for SQL Server and the tools for migrations:

```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet add package Microsoft.EntityFrameworkCore.Design
```

In Visual Studio's Package Manager Console:

```
Install-Package Microsoft.EntityFrameworkCore.SqlServer
Install-Package Microsoft.EntityFrameworkCore.Tools
```

These packages enable your app to connect to SQL Server and use EF Core's database migration features.

3. Creating Models

Blog Model (Principal entity)

```csharp
// Models/Blog.cs
namespace BlogApi.Models;

public class Blog
{
    public int Id { get; set; }
    public string Title { get; set; } = string.Empty;
    public string Content { get; set; } = string.Empty;
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public bool IsPublished { get; set; } = false;

    // Navigation property for one-to-many relationship with Post
    public ICollection<Post> Posts { get; set; } = new List<Post>();
}
```

Post Model (Dependent entity)

```csharp
// Models/Post.cs
namespace BlogApi.Models;

public class Post
{
    public int Id { get; set; }
    public string Title { get; set; } = string.Empty;
    public string Content { get; set; } = string.Empty;
    public DateTime PublishedAt { get; set; }

    // Foreign key for Blog relationship
    public int BlogId { get; set; }

    // Reference navigation property back to Blog
    public Blog Blog { get; set; } = null!;
}
```

In a one-to-many relationship, a single Blog can have many associated Posts, but each Post belongs to only one Blog. The foreign key BlogId on the Post entity makes the relationship required, ensuring every post is associated with a blog.

4. Setting Up the Database Context

```csharp
// Data/ApplicationDbContext.cs
using BlogApi.Models;
using Microsoft.EntityFrameworkCore;

namespace BlogApi.Data;

public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        // Configure the one-to-many relationship explicitly (optional, conventions handle it)
        modelBuilder.Entity<Post>()
            .HasOne(p => p.Blog)
            .WithMany(b => b.Posts)
            .HasForeignKey(p => p.BlogId)
            .OnDelete(DeleteBehavior.Cascade);
    }
}
```

The ApplicationDbContext acts as the bridge between your C# code and the database, telling EF Core what tables (DbSet<T>) to track. OnModelCreating is where you can configure relationships using Fluent API—here, we define that a Post belongs to one Blog, and a Blog has many Posts, with cascading delete.

5. Configuring the Connection String

Add your SQL Server connection string to appsettings.json:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=BlogDb;Trusted_Connection=True;MultipleActiveResultSets=true"
  },
  "Logging": { ... },
  "AllowedHosts": "*"
}
```

Step 6: Register DbContext in Program.cs

```csharp
// Program.cs
using BlogApi.Data;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddControllers();

// Register DbContext with SQL Server
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// Learn more about configuring Swagger/OpenAPI
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

Registering DbContext with dependency injection ensures each request gets a fresh instance of your database context, allowing EF Core to manage database connections efficiently.

7. Creating and Applying Migrations

Run these commands to create the database from your models (Code-First approach):

```bash
dotnet ef migrations add InitialCreate
dotnet ef database update
```

In Visual Studio Package Manager Console:

```
Add-Migration InitialCreate
Update-Database
```

The Add-Migration command scans your entity classes and creates migration files that represent the database schema changes. Update-Database applies these changes to the actual SQL Server database.

8. Creating the Blog Controller with LINQ Queries

```csharp
// Controllers/BlogsController.cs
using BlogApi.Data;
using BlogApi.Models;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

namespace BlogApi.Controllers;

[Route("api/[controller]")]
[ApiController]
public class BlogsController : ApplicationController
{
    private readonly ApplicationDbContext _context;

    public BlogsController(ApplicationDbContext context)
    {
        _context = context;
    }
```

GET: Get all blogs

```csharp
[HttpGet]
public async Task<ActionResult<IEnumerable<Blog>>> GetBlogs()
{
    // LINQ query to load all blogs
    var blogs = await _context.Blogs.ToListAsync();
    return Ok(blogs);
}
```

ToListAsync() executes the LINQ query against the database and returns all blog records.

GET with filter and sorting

```csharp
[HttpGet("published")]
public async Task<ActionResult<IEnumerable<Blog>>> GetPublishedBlogs()
{
    // LINQ query with WHERE filter and ORDER BY sorting
    var blogs = await _context.Blogs
        .Where(b => b.IsPublished == true)
        .OrderByDescending(b => b.CreatedAt)
        .ToListAsync();
    
    return Ok(blogs);
}
```

Where() applies a filter (only published blogs), and OrderByDescending() sorts the results by creation date—both are translated to SQL WHERE and ORDER BY clauses.

GET single blog with posts (Eager Loading)

```csharp
[HttpGet("{id}")]
public async Task<ActionResult<Blog>> GetBlog(int id)
{
    // LINQ query with Include() for eager loading related Posts
    var blog = await _context.Blogs
        .Include(b => b.Posts)
        .FirstOrDefaultAsync(b => b.Id == id);

    if (blog == null)
        return NotFound();

    return Ok(blog);
}
```

Include(b => b.Posts) performs an eager loading join, loading both the blog and its related posts in a single database query, preventing N+1 query issues.

GET with projection (Select specific fields)

```csharp
[HttpGet("summary")]
public async Task<ActionResult<IEnumerable<object>>> GetBlogSummaries()
{
    // LINQ projection to return only specific fields
    var summaries = await _context.Blogs
        .Where(b => b.IsPublished)
        .Select(b => new {
            b.Id,
            b.Title,
            b.CreatedAt,
            PostCount = b.Posts.Count
        })
        .ToListAsync();

    return Ok(summaries);
}
```

Select() projects only the needed fields, improving performance by reducing data transferred from the database.

POST: Create a new blog

```csharp
[HttpPost]
public async Task<ActionResult<Blog>> CreateBlog(Blog blog)
{
    _context.Blogs.Add(blog);
    await _context.SaveChangesAsync();

    return CreatedAtAction(nameof(GetBlog), new { id = blog.Id }, blog);
}
```

Add() marks the entity as added, and SaveChangesAsync() generates and executes the INSERT SQL command.

PUT: Update a blog

```csharp
[HttpPut("{id}")]
public async Task<IActionResult> UpdateBlog(int id, Blog blog)
{
    if (id != blog.Id)
        return BadRequest();

    _context.Entry(blog).State = EntityState.Modified;

    try
    {
        await _context.SaveChangesAsync();
    }
    catch (DbUpdateConcurrencyException)
    {
        if (!_context.Blogs.Any(b => b.Id == id))
            return NotFound();
        throw;
    }

    return NoContent();
}
```

Setting the entity state to Modified tells EF Core to generate an UPDATE statement for all modified properties.

DELETE: Remove a blog

```csharp
[HttpDelete("{id}")]
public async Task<IActionResult> DeleteBlog(int id)
{
    var blog = await _context.Blogs.FindAsync(id);
    if (blog == null)
        return NotFound();

    _context.Blogs.Remove(blog);
    await _context.SaveChangesAsync();

    return NoContent();
}
```

FindAsync() retrieves the blog by its primary key, Remove() marks it for deletion, and SaveChangesAsync() generates the DELETE SQL.

9. Working with Related Data (Posts Controller)

```csharp
// Controllers/PostsController.cs
[Route("api/[controller]")]
[ApiController]
public class PostsController : ApplicationController
{
    private readonly ApplicationDbContext _context;

    public PostsController(ApplicationDbContext context)
    {
        _context = context;
    }

    // GET: api/posts/blog/5
    [HttpGet("blog/{blogId}")]
    public async Task<ActionResult<IEnumerable<Post>>> GetPostsByBlog(int blogId)
    {
        var posts = await _context.Posts
            .Where(p => p.BlogId == blogId)
            .Include(p => p.Blog)
            .OrderByDescending(p => p.PublishedAt)
            .ToListAsync();

        return Ok(posts);
    }

    // POST: api/posts
    [HttpPost]
    public async Task<ActionResult<Post>> CreatePost(Post post)
    {
        // Verify the blog exists
        var blog = await _context.Blogs.FindAsync(post.BlogId);
        if (blog == null)
            return BadRequest("Blog not found");

        _context.Posts.Add(post);
        await _context.SaveChangesAsync();

        return CreatedAtAction(nameof(GetPost), new { id = post.Id }, post);
    }
}
```

When working with related entities, LINQ's Where() filters by the foreign key, and Include() loads the parent Blog information simultaneously.

10. Advanced LINQ Queries

Aggregation operations

```csharp
// Count total blogs
var totalBlogs = await _context.Blogs.CountAsync();

// Sum, Average, Min, Max
var avgPosts = await _context.Blogs.AverageAsync(b => b.Posts.Count);
```

Joining tables

```csharp
var query = from blog in _context.Blogs
            join post in _context.Posts on blog.Id equals post.BlogId
            where blog.IsPublished && post.PublishedAt > DateTime.Now.AddDays(-7)
            select new { blog.Title, post.Title, post.PublishedAt };
```

Complex filtering with Contains (LIKE in SQL)

```csharp
var searchTerm = "tutorial";
var blogs = await _context.Blogs
    .Where(b => b.Title.Contains(searchTerm) || b.Content.Contains(searchTerm))
    .ToListAsync();
```

Contains() translates to SQL's LIKE '%tutorial%', performing a substring search.

11. Summary

You've built a complete Blog API that:

· Uses EF Core Code-First with SQL Server to automatically generate your database schema from C# models
· Leverages LINQ queries (Where, Include, Select, OrderBy, Join) to perform all database operations
· Implements CRUD endpoints for Blog and Post entities
· Models one-to-many relationships between Blogs and Posts using foreign keys and navigation properties

To test your API, run the project and navigate to https://localhost:5001/swagger to interact with all endpoints using the Swagger UI.

Further Learning

· Microsoft's official EF Core documentation for in-depth concepts and patterns
· 101 LINQ Samples for extensive query examples
· EF Core Relationships Guide for mastering one-to-many, one-to-one, and many-to-many configurations