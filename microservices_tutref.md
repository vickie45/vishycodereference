# Microservices using ASP.NET Core - Complete Course Syllabus with Code Examples

## Module 1: Foundations & Fundamentals

### Chapter 1: Introduction to Microservices Architecture

#### What are Applications and Their Structure

Applications are software systems designed to perform specific functions. They're structured into layers:

```
┌─────────────────────────────────────┐
│      Presentation Layer (UI)        │ ← User Interface
├─────────────────────────────────────┤
│      Business Logic Layer           │ ← Core functionality
├─────────────────────────────────────┤
│      Data Access Layer              │ ← Database operations
├─────────────────────────────────────┤
│      Database Layer                 │ ← Data storage
└─────────────────────────────────────┘
```

#### Monolithic Architecture Example

```csharp
// Monolithic E-commerce Application Structure
public class MonolithicECommerceApp
{
    // Everything in one application
}

// Controllers handle all concerns
[ApiController]
[Route("api/[controller]")]
public class MonolithicController : ControllerBase
{
    private readonly MonolithicDbContext _context;
    
    public MonolithicController(MonolithicDbContext context)
    {
        _context = context;
    }
    
    // User management
    [HttpPost("users")]
    public async Task<IActionResult> CreateUser(User user)
    {
        _context.Users.Add(user);
        await _context.SaveChangesAsync();
        return Ok(user);
    }
    
    // Product management
    [HttpPost("products")]
    public async Task<IActionResult> CreateProduct(Product product)
    {
        _context.Products.Add(product);
        await _context.SaveChangesAsync();
        return Ok(product);
    }
    
    // Order management
    [HttpPost("orders")]
    public async Task<IActionResult> CreateOrder(Order order)
    {
        _context.Orders.Add(order);
        await _context.SaveChangesAsync();
        return Ok(order);
    }
}

// Single DbContext handling all entities
public class MonolithicDbContext : DbContext
{
    public DbSet<User> Users { get; set; }
    public DbSet<Product> Products { get; set; }
    public DbSet<Order> Orders { get; set; }
    public DbSet<Payment> Payments { get; set; }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // All relationships defined here
        modelBuilder.Entity<Order>()
            .HasOne(o => o.User)
            .WithMany(u => u.Orders)
            .HasForeignKey(o => o.UserId);
            
        modelBuilder.Entity<Order>()
            .HasMany(o => o.Items)
            .WithOne(i => i.Order)
            .HasForeignKey(i => i.OrderId);
    }
}
```

**Challenges of Monolithic Architecture:**
- Single point of failure
- Difficult to scale specific components
- Long deployment cycles
- Technology lock-in
- Team coordination overhead

#### Microservices Architecture Example

```csharp
// User Microservice (runs independently)
// Project: UserService.API
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly UserDbContext _context;
    
    public UsersController(UserDbContext context)
    {
        _context = context;
    }
    
    [HttpPost]
    public async Task<IActionResult> CreateUser(User user)
    {
        _context.Users.Add(user);
        await _context.SaveChangesAsync();
        return Ok(user);
    }
}

// Product Microservice (runs independently)
// Project: ProductService.API
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly ProductDbContext _context;
    
    public ProductsController(ProductDbContext context)
    {
        _context = context;
    }
    
    [HttpPost]
    public async Task<IActionResult> CreateProduct(Product product)
    {
        _context.Products.Add(product);
        await _context.SaveChangesAsync();
        return Ok(product);
    }
}

// Order Microservice (runs independently)
// Project: OrderService.API
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    private readonly OrderDbContext _context;
    private readonly IUserServiceClient _userClient;
    private readonly IProductServiceClient _productClient;
    
    public OrdersController(
        OrderDbContext context,
        IUserServiceClient userClient,
        IProductServiceClient productClient)
    {
        _context = context;
        _userClient = userClient;
        _productClient = productClient;
    }
    
    [HttpPost]
    public async Task<IActionResult> CreateOrder(OrderRequest request)
    {
        // Validate user exists via API call
        var user = await _userClient.GetUserAsync(request.UserId);
        if (user == null)
            return BadRequest("Invalid user");
            
        // Validate products via API call
        var product = await _productClient.GetProductAsync(request.ProductId);
        if (product == null || product.Stock < request.Quantity)
            return BadRequest("Product unavailable");
            
        var order = new Order
        {
            UserId = request.UserId,
            Items = new List<OrderItem>
            {
                new OrderItem
                {
                    ProductId = request.ProductId,
                    Quantity = request.Quantity,
                    Price = product.Price
                }
            },
            TotalAmount = product.Price * request.Quantity,
            Status = OrderStatus.Placed
        };
        
        _context.Orders.Add(order);
        await _context.SaveChangesAsync();
        
        return Ok(order);
    }
}
```

---

### Chapter 2: Core Microservices Design Principles

#### Single Responsibility Principle (SRP)

```csharp
// ❌ Bad: One service doing too much
public class UserService
{
    public void RegisterUser(User user)
    {
        // Validate user
        // Save to database
        // Send welcome email
        // Create audit log
        // Update CRM
    }
}

// ✅ Good: Each service has single responsibility
public class UserRegistrationService
{
    private readonly IUserRepository _userRepository;
    private readonly IMessageBus _messageBus;
    
    public async Task RegisterUser(UserRegistrationRequest request)
    {
        var user = new User
        {
            Email = request.Email,
            Name = request.Name,
            PasswordHash = HashPassword(request.Password)
        };
        
        await _userRepository.CreateAsync(user);
        
        // Publish event instead of doing everything
        await _messageBus.PublishAsync(new UserRegisteredEvent
        {
            UserId = user.Id,
            Email = user.Email,
            Name = user.Name
        });
    }
}

// Separate service handles email
public class EmailService : IEventHandler<UserRegisteredEvent>
{
    public async Task Handle(UserRegisteredEvent @event)
    {
        await SendWelcomeEmail(@event.Email, @event.Name);
    }
}

// Separate service handles CRM
public class CRMService : IEventHandler<UserRegisteredEvent>
{
    public async Task Handle(UserRegisteredEvent @event)
    {
        await UpdateCRM(@event.UserId, @event.Email);
    }
}
```

#### Decentralized Data Management

```csharp
// Each microservice owns its database

// User Service Database
public class UserDbContext : DbContext
{
    public DbSet<User> Users { get; set; }
    
    protected override void OnConfiguring(DbContextOptionsBuilder options)
    {
        options.UseSqlServer("Server=localhost;Database=UserServiceDb;...");
    }
}

// Product Service Database
public class ProductDbContext : DbContext
{
    public DbSet<Product> Products { get; set; }
    public DbSet<Category> Categories { get; set; }
    
    protected override void OnConfiguring(DbContextOptionsBuilder options)
    {
        options.UseSqlServer("Server=localhost;Database=ProductServiceDb;...");
    }
}

// Order Service Database
public class OrderDbContext : DbContext
{
    public DbSet<Order> Orders { get; set; }
    public DbSet<OrderItem> OrderItems { get; set; }
    
    protected override void OnConfiguring(DbContextOptionsBuilder options)
    {
        options.UseSqlServer("Server=localhost;Database=OrderServiceDb;...");
    }
    
    // Stores only necessary user/product data
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Order>()
            .Property(o => o.UserId)
            .IsRequired();
            
        modelBuilder.Entity<OrderItem>()
            .OwnsOne(i => i.ProductSnapshot, snapshot =>
            {
                snapshot.Property(p => p.ProductId);
                snapshot.Property(p => p.Name);
                snapshot.Property(p => p.Price);
            });
    }
}
```

#### Event-Driven Communication

```csharp
// Event definition
public record OrderPlacedEvent
{
    public Guid OrderId { get; init; }
    public Guid UserId { get; init; }
    public decimal TotalAmount { get; init; }
    public List<OrderItemDetail> Items { get; init; }
    public DateTime OccurredOn { get; init; }
}

public record OrderItemDetail
{
    public Guid ProductId { get; init; }
    public int Quantity { get; init; }
    public decimal Price { get; init; }
}

// Publisher (Order Service)
public class OrderService
{
    private readonly IMessageBus _messageBus;
    private readonly IOrderRepository _repository;
    
    public async Task<Order> PlaceOrder(PlaceOrderRequest request)
    {
        var order = new Order { /* ... */ };
        await _repository.CreateAsync(order);
        
        // Publish event
        await _messageBus.PublishAsync(new OrderPlacedEvent
        {
            OrderId = order.Id,
            UserId = order.UserId,
            TotalAmount = order.TotalAmount,
            Items = order.Items.Select(i => new OrderItemDetail
            {
                ProductId = i.ProductId,
                Quantity = i.Quantity,
                Price = i.Price
            }).ToList(),
            OccurredOn = DateTime.UtcNow
        });
        
        return order;
    }
}

// Subscriber (Inventory Service)
public class InventoryService : IEventHandler<OrderPlacedEvent>
{
    private readonly IProductRepository _repository;
    
    public async Task Handle(OrderPlacedEvent @event)
    {
        foreach (var item in @event.Items)
        {
            await _repository.DecrementStockAsync(item.ProductId, item.Quantity);
        }
    }
}
```

---

### Chapter 3: Clean Architecture

```csharp
// Solution Structure
/*
MicroservicesSolution/
├── src/
│   ├── UserService/
│   │   ├── UserService.Domain/          # Entities, Value Objects
│   │   ├── UserService.Application/     # Use Cases, DTOs, Interfaces
│   │   ├── UserService.Infrastructure/  # Repositories, External Services
│   │   └── UserService.API/            # Controllers, Middleware
│   ├── ProductService/
│   └── OrderService/
*/

// ========== DOMAIN LAYER ==========
// UserService.Domain/Entities/User.cs
public class User
{
    public Guid Id { get; private set; }
    public string Email { get; private set; }
    public string Name { get; private set; }
    public string PasswordHash { get; private set; }
    public UserRole Role { get; private set; }
    public DateTime CreatedAt { get; private set; }
    
    private User() { } // EF Core
    
    public User(string email, string name, string passwordHash, UserRole role)
    {
        Id = Guid.NewGuid();
        Email = email ?? throw new ArgumentNullException(nameof(email));
        Name = name ?? throw new ArgumentNullException(nameof(name));
        PasswordHash = passwordHash ?? throw new ArgumentNullException(nameof(passwordHash));
        Role = role;
        CreatedAt = DateTime.UtcNow;
    }
    
    public void UpdateProfile(string name)
    {
        Name = name ?? throw new ArgumentNullException(nameof(name));
    }
    
    public void ChangePassword(string newPasswordHash)
    {
        PasswordHash = newPasswordHash ?? throw new ArgumentNullException(nameof(newPasswordHash));
    }
}

// UserService.Domain/ValueObjects/Address.cs
public class Address : ValueObject
{
    public string Street { get; }
    public string City { get; }
    public string State { get; }
    public string ZipCode { get; }
    public string Country { get; }
    
    public Address(string street, string city, string state, string zipCode, string country)
    {
        Street = street;
        City = city;
        State = state;
        ZipCode = zipCode;
        Country = country;
    }
    
    protected override IEnumerable<object> GetEqualityComponents()
    {
        yield return Street;
        yield return City;
        yield return State;
        yield return ZipCode;
        yield return Country;
    }
}

// ValueObject base class
public abstract class ValueObject
{
    protected abstract IEnumerable<object> GetEqualityComponents();
    
    public override bool Equals(object obj)
    {
        if (obj == null || obj.GetType() != GetType())
            return false;
            
        var other = (ValueObject)obj;
        return GetEqualityComponents().SequenceEqual(other.GetEqualityComponents());
    }
    
    public override int GetHashCode()
    {
        return GetEqualityComponents()
            .Select(x => x?.GetHashCode() ?? 0)
            .Aggregate((x, y) => x ^ y);
    }
}

// ========== APPLICATION LAYER ==========
// UserService.Application/Interfaces/IUserRepository.cs
public interface IUserRepository
{
    Task<User> GetByIdAsync(Guid id);
    Task<User> GetByEmailAsync(string email);
    Task<IEnumerable<User>> GetAllAsync();
    Task AddAsync(User user);
    Task UpdateAsync(User user);
    Task DeleteAsync(Guid id);
}

// UserService.Application/DTOs/UserDto.cs
public class UserDto
{
    public Guid Id { get; set; }
    public string Email { get; set; }
    public string Name { get; set; }
    public string Role { get; set; }
    public DateTime CreatedAt { get; set; }
}

// UserService.Application/DTOs/CreateUserRequest.cs
public class CreateUserRequest
{
    public string Email { get; set; }
    public string Name { get; set; }
    public string Password { get; set; }
    public string Role { get; set; }
}

// UserService.Application/Services/UserService.cs
public class UserService : IUserService
{
    private readonly IUserRepository _repository;
    private readonly IPasswordHasher _passwordHasher;
    private readonly IMapper _mapper;
    
    public UserService(
        IUserRepository repository,
        IPasswordHasher passwordHasher,
        IMapper mapper)
    {
        _repository = repository;
        _passwordHasher = passwordHasher;
        _mapper = mapper;
    }
    
    public async Task<UserDto> CreateUserAsync(CreateUserRequest request)
    {
        // Domain logic
        var existingUser = await _repository.GetByEmailAsync(request.Email);
        if (existingUser != null)
            throw new DomainException("User with this email already exists");
            
        var role = Enum.Parse<UserRole>(request.Role);
        var passwordHash = _passwordHasher.HashPassword(request.Password);
        
        var user = new User(request.Email, request.Name, passwordHash, role);
        
        await _repository.AddAsync(user);
        
        return _mapper.Map<UserDto>(user);
    }
    
    public async Task<UserDto> GetUserByIdAsync(Guid id)
    {
        var user = await _repository.GetByIdAsync(id);
        if (user == null)
            throw new NotFoundException($"User with id {id} not found");
            
        return _mapper.Map<UserDto>(user);
    }
}

// ========== INFRASTRUCTURE LAYER ==========
// UserService.Infrastructure/Persistence/UserRepository.cs
public class UserRepository : IUserRepository
{
    private readonly UserDbContext _context;
    
    public UserRepository(UserDbContext context)
    {
        _context = context;
    }
    
    public async Task<User> GetByIdAsync(Guid id)
    {
        return await _context.Users.FindAsync(id);
    }
    
    public async Task<User> GetByEmailAsync(string email)
    {
        return await _context.Users
            .FirstOrDefaultAsync(u => u.Email == email);
    }
    
    public async Task<IEnumerable<User>> GetAllAsync()
    {
        return await _context.Users.ToListAsync();
    }
    
    public async Task AddAsync(User user)
    {
        await _context.Users.AddAsync(user);
        await _context.SaveChangesAsync();
    }
    
    public async Task UpdateAsync(User user)
    {
        _context.Users.Update(user);
        await _context.SaveChangesAsync();
    }
    
    public async Task DeleteAsync(Guid id)
    {
        var user = await GetByIdAsync(id);
        if (user != null)
        {
            _context.Users.Remove(user);
            await _context.SaveChangesAsync();
        }
    }
}

// UserService.Infrastructure/Persistence/UserDbContext.cs
public class UserDbContext : DbContext
{
    public DbSet<User> Users { get; set; }
    
    public UserDbContext(DbContextOptions<UserDbContext> options) : base(options) { }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<User>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Email).IsRequired().HasMaxLength(256);
            entity.HasIndex(e => e.Email).IsUnique();
            entity.Property(e => e.Name).IsRequired().HasMaxLength(200);
            entity.Property(e => e.PasswordHash).IsRequired();
            entity.Property(e => e.Role).HasConversion<string>();
        });
    }
}

// ========== API LAYER ==========
// UserService.API/Controllers/UsersController.cs
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly IUserService _userService;
    
    public UsersController(IUserService userService)
    {
        _userService = userService;
    }
    
    [HttpPost]
    [ProducesResponseType(typeof(UserDto), StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<IActionResult> CreateUser([FromBody] CreateUserRequest request)
    {
        try
        {
            var user = await _userService.CreateUserAsync(request);
            return CreatedAtAction(nameof(GetUser), new { id = user.Id }, user);
        }
        catch (DomainException ex)
        {
            return BadRequest(new { error = ex.Message });
        }
    }
    
    [HttpGet("{id:guid}")]
    [ProducesResponseType(typeof(UserDto), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> GetUser(Guid id)
    {
        try
        {
            var user = await _userService.GetUserByIdAsync(id);
            return Ok(user);
        }
        catch (NotFoundException)
        {
            return NotFound();
        }
    }
}

// UserService.API/Program.cs
var builder = WebApplication.CreateBuilder(args);

// Add services
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Application layer
builder.Services.AddScoped<IUserService, UserService.Application.Services.UserService>();
builder.Services.AddScoped<IPasswordHasher, PasswordHasher>();

// Infrastructure layer
builder.Services.AddScoped<IUserRepository, UserRepository>();
builder.Services.AddDbContext<UserDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("UserDb")));

// AutoMapper
builder.Services.AddAutoMapper(typeof(UserMappingProfile));

var app = builder.Build();

// Configure pipeline
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

---

### Chapter 4: Domain-Driven Design (DDD)

```csharp
// ========== DOMAIN LAYER WITH DDD ==========

// Aggregates
// OrderService.Domain/Aggregates/Order.cs
public class Order : IAggregateRoot
{
    public Guid Id { get; private set; }
    public Guid UserId { get; private set; }
    public OrderStatus Status { get; private set; }
    public Address ShippingAddress { get; private set; }
    public Money TotalAmount { get; private set; }
    
    private readonly List<OrderItem> _items = new();
    public IReadOnlyCollection<OrderItem> Items => _items.AsReadOnly();
    
    public DateTime CreatedAt { get; private set; }
    public DateTime? PaidAt { get; private set; }
    public DateTime? ShippedAt { get; private set; }
    public DateTime? DeliveredAt { get; private set; }
    
    private Order() { }
    
    public Order(Guid userId, Address shippingAddress)
    {
        Id = Guid.NewGuid();
        UserId = userId;
        ShippingAddress = shippingAddress;
        Status = OrderStatus.Draft;
        CreatedAt = DateTime.UtcNow;
    }
    
    public void AddItem(Guid productId, string productName, Money unitPrice, int quantity)
    {
        if (Status != OrderStatus.Draft)
            throw new DomainException("Cannot add items to a non-draft order");
            
        var existingItem = _items.FirstOrDefault(i => i.ProductId == productId);
        if (existingItem != null)
        {
            existingItem.UpdateQuantity(existingItem.Quantity + quantity);
        }
        else
        {
            _items.Add(new OrderItem(productId, productName, unitPrice, quantity));
        }
        
        RecalculateTotal();
    }
    
    public void Place()
    {
        if (Status != OrderStatus.Draft)
            throw new DomainException("Order cannot be placed");
            
        if (!_items.Any())
            throw new DomainException("Order must have at least one item");
            
        Status = OrderStatus.Placed;
        
        AddDomainEvent(new OrderPlacedEvent(Id, UserId, TotalAmount, _items));
    }
    
    public void MarkAsPaid()
    {
        if (Status != OrderStatus.Placed)
            throw new DomainException("Only placed orders can be marked as paid");
            
        Status = OrderStatus.Paid;
        PaidAt = DateTime.UtcNow;
        
        AddDomainEvent(new OrderPaidEvent(Id));
    }
    
    public void MarkAsShipped()
    {
        if (Status != OrderStatus.Paid)
            throw new DomainException("Only paid orders can be shipped");
            
        Status = OrderStatus.Shipped;
        ShippedAt = DateTime.UtcNow;
        
        AddDomainEvent(new OrderShippedEvent(Id));
    }
    
    public void Cancel(string reason)
    {
        if (Status == OrderStatus.Shipped || Status == OrderStatus.Delivered)
            throw new DomainException("Cannot cancel shipped or delivered orders");
            
        Status = OrderStatus.Cancelled;
        AddDomainEvent(new OrderCancelledEvent(Id, reason));
    }
    
    private void RecalculateTotal()
    {
        TotalAmount = new Money(_items.Sum(i => i.TotalPrice.Amount), "USD");
    }
    
    private readonly List<IDomainEvent> _domainEvents = new();
    public IReadOnlyCollection<IDomainEvent> DomainEvents => _domainEvents.AsReadOnly();
    
    protected void AddDomainEvent(IDomainEvent domainEvent)
    {
        _domainEvents.Add(domainEvent);
    }
    
    public void ClearDomainEvents()
    {
        _domainEvents.Clear();
    }
}

// Entities
// OrderService.Domain/Entities/OrderItem.cs
public class OrderItem : Entity
{
    public Guid ProductId { get; private set; }
    public string ProductName { get; private set; }
    public Money UnitPrice { get; private set; }
    public int Quantity { get; private set; }
    public Money TotalPrice => new Money(UnitPrice.Amount * Quantity, UnitPrice.Currency);
    
    private OrderItem() { }
    
    public OrderItem(Guid productId, string productName, Money unitPrice, int quantity)
    {
        if (quantity <= 0)
            throw new DomainException("Quantity must be positive");
            
        Id = Guid.NewGuid();
        ProductId = productId;
        ProductName = productName ?? throw new ArgumentNullException(nameof(productName));
        UnitPrice = unitPrice;
        Quantity = quantity;
    }
    
    public void UpdateQuantity(int newQuantity)
    {
        if (newQuantity <= 0)
            throw new DomainException("Quantity must be positive");
            
        Quantity = newQuantity;
    }
}

// Value Objects
// OrderService.Domain/ValueObjects/Money.cs
public class Money : ValueObject
{
    public decimal Amount { get; }
    public string Currency { get; }
    
    public Money(decimal amount, string currency)
    {
        if (amount < 0)
            throw new DomainException("Amount cannot be negative");
            
        Amount = amount;
        Currency = currency ?? throw new ArgumentNullException(nameof(currency));
    }
    
    public static Money Zero(string currency = "USD") => new(0, currency);
    
    public Money Add(Money other)
    {
        if (Currency != other.Currency)
            throw new DomainException("Cannot add money with different currencies");
            
        return new Money(Amount + other.Amount, Currency);
    }
    
    public Money Multiply(int multiplier)
    {
        return new Money(Amount * multiplier, Currency);
    }
    
    protected override IEnumerable<object> GetEqualityComponents()
    {
        yield return Amount;
        yield return Currency;
    }
}

// Domain Events
public interface IDomainEvent
{
    DateTime OccurredOn { get; }
}

public record OrderPlacedEvent(
    Guid OrderId,
    Guid UserId,
    Money TotalAmount,
    IReadOnlyCollection<OrderItem> Items
) : IDomainEvent
{
    public DateTime OccurredOn { get; } = DateTime.UtcNow;
}

// Domain Services
// OrderService.Domain/Services/PricingService.cs
public class PricingService : IPricingService
{
    private readonly IDiscountRepository _discountRepository;
    
    public PricingService(IDiscountRepository discountRepository)
    {
        _discountRepository = discountRepository;
    }
    
    public async Task<Money> CalculatePrice(Guid productId, int quantity, string couponCode = null)
    {
        var basePrice = await GetBasePrice(productId);
        var totalPrice = basePrice.Multiply(quantity);
        
        if (!string.IsNullOrEmpty(couponCode))
        {
            var discount = await _discountRepository.GetByCodeAsync(couponCode);
            if (discount?.IsValid() == true)
            {
                totalPrice = discount.ApplyTo(totalPrice);
            }
        }
        
        return totalPrice;
    }
}

// Repository Interface (in Domain)
public interface IOrderRepository
{
    Task<Order> GetByIdAsync(Guid id);
    Task<IEnumerable<Order>> GetByUserIdAsync(Guid userId);
    Task AddAsync(Order order);
    Task UpdateAsync(Order order);
}

// Specification Pattern
public class ActiveOrderSpecification : ISpecification<Order>
{
    public Expression<Func<Order, bool>> Criteria => 
        o => o.Status != OrderStatus.Delivered && 
             o.Status != OrderStatus.Cancelled;
}

public class OrderByStatusSpecification : ISpecification<Order>
{
    private readonly OrderStatus _status;
    
    public OrderByStatusSpecification(OrderStatus status)
    {
        _status = status;
    }
    
    public Expression<Func<Order, bool>> Criteria => 
        o => o.Status == _status;
}

// ========== APPLICATION LAYER ==========
// OrderService.Application/Commands/PlaceOrderCommand.cs
public record PlaceOrderCommand(
    Guid UserId,
    Address ShippingAddress,
    List<OrderItemRequest> Items
) : IRequest<OrderDto>;

public record OrderItemRequest(Guid ProductId, int Quantity);

// OrderService.Application/Handlers/PlaceOrderHandler.cs
public class PlaceOrderHandler : IRequestHandler<PlaceOrderCommand, OrderDto>
{
    private readonly IOrderRepository _orderRepository;
    private readonly IUserServiceClient _userService;
    private readonly IProductServiceClient _productService;
    private readonly IPricingService _pricingService;
    private readonly IMapper _mapper;
    
    public PlaceOrderHandler(
        IOrderRepository orderRepository,
        IUserServiceClient userService,
        IProductServiceClient productService,
        IPricingService pricingService,
        IMapper mapper)
    {
        _orderRepository = orderRepository;
        _userService = userService;
        _productService = productService;
        _pricingService = pricingService;
        _mapper = mapper;
    }
    
    public async Task<OrderDto> Handle(PlaceOrderCommand request, CancellationToken cancellationToken)
    {
        // Validate user
        var user = await _userService.GetUserAsync(request.UserId);
        if (user == null)
            throw new ValidationException("User not found");
            
        // Create order aggregate
        var order = new Order(request.UserId, request.ShippingAddress);
        
        foreach (var item in request.Items)
        {
            var product = await _productService.GetProductAsync(item.ProductId);
            if (product == null)
                throw new ValidationException($"Product {item.ProductId} not found");
                
            if (product.Stock < item.Quantity)
                throw new ValidationException($"Insufficient stock for product {product.Name}");
                
            var price = await _pricingService.CalculatePrice(product.Id, item.Quantity);
            
            order.AddItem(
                product.Id,
                product.Name,
                new Money(product.Price, "USD"),
                item.Quantity
            );
        }
        
        // Place the order
        order.Place();
        
        // Persist
        await _orderRepository.AddAsync(order);
        
        return _mapper.Map<OrderDto>(order);
    }
}

// ========== INFRASTRUCTURE LAYER ==========
// OrderService.Infrastructure/Persistence/OrderRepository.cs
public class OrderRepository : IOrderRepository
{
    private readonly OrderDbContext _context;
    private readonly IMediator _mediator;
    
    public OrderRepository(OrderDbContext context, IMediator mediator)
    {
        _context = context;
        _mediator = mediator;
    }
    
    public async Task<Order> GetByIdAsync(Guid id)
    {
        return await _context.Orders
            .Include(o => o.Items)
            .FirstOrDefaultAsync(o => o.Id == id);
    }
    
    public async Task<IEnumerable<Order>> GetByUserIdAsync(Guid userId)
    {
        return await _context.Orders
            .Include(o => o.Items)
            .Where(o => o.UserId == userId)
            .ToListAsync();
    }
    
    public async Task AddAsync(Order order)
    {
        await _context.Orders.AddAsync(order);
        await _context.SaveChangesAsync();
        
        // Dispatch domain events
        await DispatchDomainEvents(order);
    }
    
    public async Task UpdateAsync(Order order)
    {
        _context.Orders.Update(order);
        await _context.SaveChangesAsync();
        
        await DispatchDomainEvents(order);
    }
    
    private async Task DispatchDomainEvents(Order order)
    {
        var events = order.DomainEvents.ToList();
        order.ClearDomainEvents();
        
        foreach (var domainEvent in events)
        {
            await _mediator.Publish(domainEvent);
        }
    }
}

// OrderService.Infrastructure/Persistence/OrderDbContext.cs
public class OrderDbContext : DbContext
{
    public DbSet<Order> Orders { get; set; }
    
    public OrderDbContext(DbContextOptions<OrderDbContext> options) : base(options) { }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Order>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Status).HasConversion<string>();
            entity.OwnsOne(e => e.ShippingAddress, address =>
            {
                address.Property(a => a.Street).HasColumnName("ShippingStreet");
                address.Property(a => a.City).HasColumnName("ShippingCity");
                address.Property(a => a.State).HasColumnName("ShippingState");
                address.Property(a => a.ZipCode).HasColumnName("ShippingZipCode");
                address.Property(a => a.Country).HasColumnName("ShippingCountry");
            });
            entity.OwnsOne(e => e.TotalAmount, money =>
            {
                money.Property(m => m.Amount).HasColumnName("TotalAmount");
                money.Property(m => m.Currency).HasColumnName("Currency");
            });
            entity.HasMany(e => e.Items)
                  .WithOne()
                  .HasForeignKey("OrderId");
        });
        
        modelBuilder.Entity<OrderItem>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.ProductName).IsRequired();
            entity.OwnsOne(e => e.UnitPrice, money =>
            {
                money.Property(m => m.Amount).HasColumnName("UnitPrice");
                money.Property(m => m.Currency).HasColumnName("Currency");
            });
        });
        
        modelBuilder.Ignore<IDomainEvent>();
    }
}
```

---

## Module 2: Building Core Microservices

### Chapter 5: Microservices Project Setup in Visual Studio

```bash
# Solution Structure
ECommerce.Microservices/
├── src/
│   ├── Services/
│   │   ├── UserService/
│   │   │   ├── UserService.Domain/
│   │   │   ├── UserService.Application/
│   │   │   ├── UserService.Infrastructure/
│   │   │   ├── UserService.API/
│   │   │   └── UserService.Tests/
│   │   ├── ProductService/
│   │   │   ├── ProductService.Domain/
│   │   │   ├── ProductService.Application/
│   │   │   ├── ProductService.Infrastructure/
│   │   │   ├── ProductService.API/
│   │   │   └── ProductService.Tests/
│   │   ├── OrderService/
│   │   │   ├── OrderService.Domain/
│   │   │   ├── OrderService.Application/
│   │   │   ├── OrderService.Infrastructure/
│   │   │   ├── OrderService.API/
│   │   │   └── OrderService.Tests/
│   │   ├── PaymentService/
│   │   └── NotificationService/
│   ├── BuildingBlocks/
│   │   ├── Common/
│   │   │   ├── Common.Domain/
│   │   │   ├── Common.Application/
│   │   │   └── Common.Infrastructure/
│   │   └── EventBus/
│   └── ApiGateways/
│       └── OcelotApiGateway/
├── docker-compose.yml
└── ECommerce.sln
```

```csharp
// Common.Domain/BaseEntity.cs (Building Blocks)
namespace Common.Domain;

public abstract class Entity
{
    public Guid Id { get; protected set; }
    
    private readonly List<IDomainEvent> _domainEvents = new();
    public IReadOnlyCollection<IDomainEvent> DomainEvents => _domainEvents.AsReadOnly();
    
    protected void AddDomainEvent(IDomainEvent domainEvent)
    {
        _domainEvents.Add(domainEvent);
    }
    
    public void ClearDomainEvents()
    {
        _domainEvents.Clear();
    }
    
    public override bool Equals(object obj)
    {
        if (obj is not Entity other)
            return false;
            
        if (ReferenceEquals(this, other))
            return true;
            
        if (GetType() != other.GetType())
            return false;
            
        return Id == other.Id;
    }
    
    public override int GetHashCode() => Id.GetHashCode();
}

public interface IAggregateRoot { }

public interface IDomainEvent
{
    DateTime OccurredOn { get; }
}
```

```xml
<!-- UserService.API.csproj -->
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
  
  <ItemGroup>
    <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="8.0.0" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="8.0.0" />
    <PackageReference Include="Swashbuckle.AspNetCore" Version="6.5.0" />
    <PackageReference Include="AutoMapper.Extensions.Microsoft.DependencyInjection" Version="12.0.1" />
    <PackageReference Include="MediatR" Version="12.2.0" />
    <PackageReference Include="FluentValidation.AspNetCore" Version="11.3.0" />
  </ItemGroup>
  
  <ItemGroup>
    <ProjectReference Include="..\UserService.Application\UserService.Application.csproj" />
    <ProjectReference Include="..\UserService.Infrastructure\UserService.Infrastructure.csproj" />
    <ProjectReference Include="..\..\..\BuildingBlocks\Common.Infrastructure\Common.Infrastructure.csproj" />
  </ItemGroup>
</Project>
```

---

### Chapter 6: Creating the User Microservice

```csharp
// UserService.Domain/Entities/User.cs
namespace UserService.Domain.Entities;

public class User : Entity, IAggregateRoot
{
    public string Email { get; private set; }
    public string Name { get; private set; }
    public string PasswordHash { get; private set; }
    public string PhoneNumber { get; private set; }
    public bool IsEmailVerified { get; private set; }
    public bool IsPhoneVerified { get; private set; }
    public UserRole Role { get; private set; }
    public DateTime CreatedAt { get; private set; }
    public DateTime? LastLoginAt { get; private set; }
    
    private readonly List<Address> _addresses = new();
    public IReadOnlyCollection<Address> Addresses => _addresses.AsReadOnly();
    
    private readonly List<RefreshToken> _refreshTokens = new();
    public IReadOnlyCollection<RefreshToken> RefreshTokens => _refreshTokens.AsReadOnly();
    
    private User() { }
    
    public User(string email, string name, string passwordHash, string phoneNumber, UserRole role)
    {
        Id = Guid.NewGuid();
        Email = email?.ToLowerInvariant() ?? throw new ArgumentNullException(nameof(email));
        Name = name ?? throw new ArgumentNullException(nameof(name));
        PasswordHash = passwordHash ?? throw new ArgumentNullException(nameof(passwordHash));
        PhoneNumber = phoneNumber;
        Role = role;
        CreatedAt = DateTime.UtcNow;
        IsEmailVerified = false;
        IsPhoneVerified = false;
        
        AddDomainEvent(new UserRegisteredEvent(Id, Email, Name));
    }
    
    public void VerifyEmail()
    {
        IsEmailVerified = true;
        AddDomainEvent(new EmailVerifiedEvent(Id, Email));
    }
    
    public void VerifyPhone()
    {
        IsPhoneVerified = true;
    }
    
    public void UpdateProfile(string name, string phoneNumber)
    {
        Name = name ?? throw new ArgumentNullException(nameof(name));
        PhoneNumber = phoneNumber;
    }
    
    public void AddAddress(Address address)
    {
        if (_addresses.Count >= 10)
            throw new DomainException("Maximum 10 addresses allowed");
            
        _addresses.Add(address);
    }
    
    public void RemoveAddress(Guid addressId)
    {
        var address = _addresses.FirstOrDefault(a => a.Id == addressId);
        if (address != null)
            _addresses.Remove(address);
    }
    
    public RefreshToken AddRefreshToken(string token, DateTime expiresAt)
    {
        var refreshToken = new RefreshToken(token, expiresAt);
        _refreshTokens.Add(refreshToken);
        return refreshToken;
    }
    
    public void RevokeRefreshToken(string token)
    {
        var refreshToken = _refreshTokens.FirstOrDefault(rt => rt.Token == token);
        if (refreshToken != null)
            refreshToken.Revoke();
    }
    
    public void RecordLogin()
    {
        LastLoginAt = DateTime.UtcNow;
    }
}

public enum UserRole
{
    Customer,
    Admin,
    Vendor
}

// UserService.Domain/Entities/RefreshToken.cs
public class RefreshToken : Entity
{
    public string Token { get; private set; }
    public DateTime ExpiresAt { get; private set; }
    public bool IsRevoked { get; private set; }
    public DateTime CreatedAt { get; private set; }
    
    private RefreshToken() { }
    
    public RefreshToken(string token, DateTime expiresAt)
    {
        Id = Guid.NewGuid();
        Token = token;
        ExpiresAt = expiresAt;
        CreatedAt = DateTime.UtcNow;
        IsRevoked = false;
    }
    
    public bool IsExpired => DateTime.UtcNow >= ExpiresAt;
    public bool IsActive => !IsRevoked && !IsExpired;
    
    public void Revoke()
    {
        IsRevoked = true;
    }
}

// UserService.Application/Services/AuthService.cs
namespace UserService.Application.Services;

public class AuthService : IAuthService
{
    private readonly IUserRepository _userRepository;
    private readonly IPasswordHasher _passwordHasher;
    private readonly ITokenService _tokenService;
    
    public AuthService(
        IUserRepository userRepository,
        IPasswordHasher passwordHasher,
        ITokenService tokenService)
    {
        _userRepository = userRepository;
        _passwordHasher = passwordHasher;
        _tokenService = tokenService;
    }
    
    public async Task<AuthenticationResult> RegisterAsync(RegisterRequest request)
    {
        var existingUser = await _userRepository.GetByEmailAsync(request.Email);
        if (existingUser != null)
            throw new DomainException("Email already registered");
            
        var passwordHash = _passwordHasher.HashPassword(request.Password);
        var user = new User(
            request.Email,
            request.Name,
            passwordHash,
            request.PhoneNumber,
            UserRole.Customer
        );
        
        await _userRepository.AddAsync(user);
        
        return await GenerateAuthenticationResult(user);
    }
    
    public async Task<AuthenticationResult> LoginAsync(LoginRequest request)
    {
        var user = await _userRepository.GetByEmailAsync(request.Email);
        if (user == null)
            throw new DomainException("Invalid email or password");
            
        if (!_passwordHasher.VerifyPassword(request.Password, user.PasswordHash))
            throw new DomainException("Invalid email or password");
            
        user.RecordLogin();
        await _userRepository.UpdateAsync(user);
        
        return await GenerateAuthenticationResult(user);
    }
    
    public async Task<AuthenticationResult> RefreshTokenAsync(string refreshToken)
    {
        var user = await _userRepository.GetByRefreshTokenAsync(refreshToken);
        if (user == null)
            throw new DomainException("Invalid refresh token");
            
        var token = user.RefreshTokens.FirstOrDefault(rt => rt.Token == refreshToken);
        if (token == null || !token.IsActive)
        {
            if (token != null)
                user.RevokeRefreshToken(refreshToken);
                
            throw new DomainException("Invalid or expired refresh token");
        }
        
        // Revoke old token
        user.RevokeRefreshToken(refreshToken);
        
        // Generate new tokens
        var result = await GenerateAuthenticationResult(user);
        await _userRepository.UpdateAsync(user);
        
        return result;
    }
    
    private async Task<AuthenticationResult> GenerateAuthenticationResult(User user)
    {
        var accessToken = _tokenService.GenerateAccessToken(user);
        var refreshToken = _tokenService.GenerateRefreshToken();
        
        user.AddRefreshToken(refreshToken.Token, refreshToken.ExpiresAt);
        
        return new AuthenticationResult
        {
            AccessToken = accessToken,
            RefreshToken = refreshToken.Token,
            ExpiresAt = refreshToken.ExpiresAt,
            User = new UserDto
            {
                Id = user.Id,
                Email = user.Email,
                Name = user.Name,
                Role = user.Role.ToString()
            }
        };
    }
}

// UserService.Application/Services/TokenService.cs
public class TokenService : ITokenService
{
    private readonly JwtSettings _jwtSettings;
    
    public TokenService(IOptions<JwtSettings> jwtSettings)
    {
        _jwtSettings = jwtSettings.Value;
    }
    
    public string GenerateAccessToken(User user)
    {
        var claims = new List<Claim>
        {
            new(ClaimTypes.NameIdentifier, user.Id.ToString()),
            new(ClaimTypes.Email, user.Email),
            new(ClaimTypes.Name, user.Name),
            new(ClaimTypes.Role, user.Role.ToString())
        };
        
        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_jwtSettings.Secret));
        var credentials = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);
        
        var token = new JwtSecurityToken(
            issuer: _jwtSettings.Issuer,
            audience: _jwtSettings.Audience,
            claims: claims,
            expires: DateTime.UtcNow.AddMinutes(_jwtSettings.AccessTokenExpirationMinutes),
            signingCredentials: credentials
        );
        
        return new JwtSecurityTokenHandler().WriteToken(token);
    }
    
    public RefreshTokenDto GenerateRefreshToken()
    {
        var randomBytes = new byte[32];
        using var rng = RandomNumberGenerator.Create();
        rng.GetBytes(randomBytes);
        
        return new RefreshTokenDto
        {
            Token = Convert.ToBase64String(randomBytes),
            ExpiresAt = DateTime.UtcNow.AddDays(7)
        };
    }
}

// UserService.API/Controllers/AuthController.cs
[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly IAuthService _authService;
    
    public AuthController(IAuthService authService)
    {
        _authService = authService;
    }
    
    [HttpPost("register")]
    [ProducesResponseType(typeof(AuthenticationResult), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<IActionResult> Register([FromBody] RegisterRequest request)
    {
        var result = await _authService.RegisterAsync(request);
        return Ok(result);
    }
    
    [HttpPost("login")]
    [ProducesResponseType(typeof(AuthenticationResult), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status401Unauthorized)]
    public async Task<IActionResult> Login([FromBody] LoginRequest request)
    {
        var result = await _authService.LoginAsync(request);
        return Ok(result);
    }
    
    [HttpPost("refresh")]
    [ProducesResponseType(typeof(AuthenticationResult), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status401Unauthorized)]
    public async Task<IActionResult> RefreshToken([FromBody] RefreshTokenRequest request)
    {
        var result = await _authService.RefreshTokenAsync(request.RefreshToken);
        return Ok(result);
    }
}

// UserService.API/Controllers/UserProfileController.cs
[ApiController]
[Route("api/[controller]")]
[Authorize]
public class UserProfileController : ControllerBase
{
    private readonly IUserService _userService;
    
    public UserProfileController(IUserService userService)
    {
        _userService = userService;
    }
    
    [HttpGet]
    public async Task<IActionResult> GetProfile()
    {
        var userId = Guid.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier));
        var user = await _userService.GetUserByIdAsync(userId);
        return Ok(user);
    }
    
    [HttpPut]
    public async Task<IActionResult> UpdateProfile([FromBody] UpdateProfileRequest request)
    {
        var userId = Guid.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier));
        var user = await _userService.UpdateProfileAsync(userId, request);
        return Ok(user);
    }
    
    [HttpGet("addresses")]
    public async Task<IActionResult> GetAddresses()
    {
        var userId = Guid.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier));
        var addresses = await _userService.GetUserAddressesAsync(userId);
        return Ok(addresses);
    }
    
    [HttpPost("addresses")]
    public async Task<IActionResult> AddAddress([FromBody] AddAddressRequest request)
    {
        var userId = Guid.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier));
        var address = await _userService.AddAddressAsync(userId, request);
        return CreatedAtAction(nameof(GetAddresses), null, address);
    }
}
```

---

### Chapter 7: Building the Product Microservice

```csharp
// ProductService.Domain/Entities/Product.cs
namespace ProductService.Domain.Entities;

public class Product : Entity, IAggregateRoot
{
    public string Name { get; private set; }
    public string Description { get; private set; }
    public string SKU { get; private set; }
    public Money Price { get; private set; }
    public int StockQuantity { get; private set; }
    public int LowStockThreshold { get; private set; }
    public ProductStatus Status { get; private set; }
    public Guid CategoryId { get; private set; }
    public Category Category { get; private set; }
    
    private readonly List<ProductImage> _images = new();
    public IReadOnlyCollection<ProductImage> Images => _images.AsReadOnly();
    
    private readonly List<ProductReview> _reviews = new();
    public IReadOnlyCollection<ProductReview> Reviews => _reviews.AsReadOnly();
    
    public decimal AverageRating => _reviews.Any() 
        ? Math.Round(_reviews.Average(r => r.Rating), 1) 
        : 0;
    
    public int ReviewCount => _reviews.Count;
    
    public DateTime CreatedAt { get; private set; }
    public DateTime UpdatedAt { get; private set; }
    
    private Product() { }
    
    public Product(string name, string description, string sku, Money price, 
                   int stockQuantity, Guid categoryId)
    {
        Id = Guid.NewGuid();
        Name = name ?? throw new ArgumentNullException(nameof(name));
        Description = description;
        SKU = sku ?? throw new ArgumentNullException(nameof(sku));
        Price = price;
        StockQuantity = stockQuantity;
        LowStockThreshold = 10;
        CategoryId = categoryId;
        Status = stockQuantity > 0 ? ProductStatus.Active : ProductStatus.OutOfStock;
        CreatedAt = DateTime.UtcNow;
        UpdatedAt = DateTime.UtcNow;
        
        AddDomainEvent(new ProductCreatedEvent(Id, Name, SKU));
    }
    
    public void UpdateDetails(string name, string description, Money price)
    {
        Name = name ?? throw new ArgumentNullException(nameof(name));
        Description = description;
        Price = price;
        UpdatedAt = DateTime.UtcNow;
    }
    
    public void UpdateStock(int quantity, string reason)
    {
        var oldQuantity = StockQuantity;
        StockQuantity = quantity;
        
        if (StockQuantity <= 0 && oldQuantity > 0)
        {
            Status = ProductStatus.OutOfStock;
            AddDomainEvent(new ProductOutOfStockEvent(Id, Name));
        }
        else if (StockQuantity > 0 && oldQuantity <= 0)
        {
            Status = ProductStatus.Active;
            AddDomainEvent(new ProductBackInStockEvent(Id, Name));
        }
        else if (StockQuantity <= LowStockThreshold && oldQuantity > LowStockThreshold)
        {
            AddDomainEvent(new ProductLowStockEvent(Id, Name, StockQuantity));
        }
        
        UpdatedAt = DateTime.UtcNow;
    }
    
    public void AddImage(string url, string altText, bool isPrimary)
    {
        if (isPrimary)
        {
            foreach (var image in _images)
                image.SetAsNonPrimary();
        }
        
        _images.Add(new ProductImage(url, altText, isPrimary));
    }
    
    public void AddReview(Guid userId, int rating, string comment)
    {
        if (rating < 1 || rating > 5)
            throw new DomainException("Rating must be between 1 and 5");
            
        var existingReview = _reviews.FirstOrDefault(r => r.UserId == userId);
        if (existingReview != null)
            throw new DomainException("User has already reviewed this product");
            
        _reviews.Add(new ProductReview(userId, rating, comment));
    }
    
    public void Deactivate()
    {
        Status = ProductStatus.Inactive;
        AddDomainEvent(new ProductDeactivatedEvent(Id, Name));
    }
    
    public void Activate()
    {
        Status = StockQuantity > 0 ? ProductStatus.Active : ProductStatus.OutOfStock;
        AddDomainEvent(new ProductActivatedEvent(Id, Name));
    }
}

public enum ProductStatus
{
    Active,
    Inactive,
    OutOfStock,
    Discontinued
}

// ProductService.Domain/Entities/Category.cs
public class Category : Entity
{
    public string Name { get; private set; }
    public string Description { get; private set; }
    public string Slug { get; private set; }
    public Guid? ParentCategoryId { get; private set; }
    public Category ParentCategory { get; private set; }
    
    private readonly List<Category> _subCategories = new();
    public IReadOnlyCollection<Category> SubCategories => _subCategories.AsReadOnly();
    
    private Category() { }
    
    public Category(string name, string description, Guid? parentCategoryId = null)
    {
        Id = Guid.NewGuid();
        Name = name ?? throw new ArgumentNullException(nameof(name));
        Description = description;
        Slug = GenerateSlug(name);
        ParentCategoryId = parentCategoryId;
    }
    
    private static string GenerateSlug(string name)
    {
        return name.ToLowerInvariant()
                   .Replace(" ", "-")
                   .Replace("--", "-");
    }
}

// ProductService.Application/Commands/CreateProductCommand.cs
public record CreateProductCommand(
    string Name,
    string Description,
    string SKU,
    decimal Price,
    string Currency,
    int StockQuantity,
    Guid CategoryId
) : IRequest<ProductDto>;

public class CreateProductCommandHandler : IRequestHandler<CreateProductCommand, ProductDto>
{
    private readonly IProductRepository _repository;
    private readonly IMapper _mapper;
    
    public CreateProductCommandHandler(IProductRepository repository, IMapper mapper)
    {
        _repository = repository;
        _mapper = mapper;
    }
    
    public async Task<ProductDto> Handle(CreateProductCommand request, CancellationToken cancellationToken)
    {
        // Validate uniqueness
        var existingProduct = await _repository.GetBySKUAsync(request.SKU);
        if (existingProduct != null)
            throw new ValidationException($"Product with SKU '{request.SKU}' already exists");
            
        var price = new Money(request.Price, request.Currency);
        var product = new Product(
            request.Name,
            request.Description,
            request.SKU,
            price,
            request.StockQuantity,
            request.CategoryId
        );
        
        await _repository.AddAsync(product);
        
        return _mapper.Map<ProductDto>(product);
    }
}

// ProductService.Application/Queries/SearchProductsQuery.cs
public record SearchProductsQuery(
    string SearchTerm = null,
    Guid? CategoryId = null,
    decimal? MinPrice = null,
    decimal? MaxPrice = null,
    int? MinRating = null,
    string SortBy = "name",
    bool SortDescending = false,
    int Page = 1,
    int PageSize = 20
) : IRequest<PaginatedResult<ProductDto>>;

public class SearchProductsQueryHandler : IRequestHandler<SearchProductsQuery, PaginatedResult<ProductDto>>
{
    private readonly IProductRepository _repository;
    private readonly IMapper _mapper;
    
    public SearchProductsQueryHandler(IProductRepository repository, IMapper mapper)
    {
        _repository = repository;
        _mapper = mapper;
    }
    
    public async Task<PaginatedResult<ProductDto>> Handle(
        SearchProductsQuery request, CancellationToken cancellationToken)
    {
        var specification = new ProductSearchSpecification(request);
        var products = await _repository.SearchAsync(specification, request.Page, request.PageSize);
        var totalCount = await _repository.CountAsync(specification);
        
        return new PaginatedResult<ProductDto>
        {
            Items = _mapper.Map<List<ProductDto>>(products),
            TotalCount = totalCount,
            Page = request.Page,
            PageSize = request.PageSize,
            TotalPages = (int)Math.Ceiling(totalCount / (double)request.PageSize)
        };
    }
}

// ProductService.API/Controllers/ProductsController.cs
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IMediator _mediator;
    
    public ProductsController(IMediator mediator)
    {
        _mediator = mediator;
    }
    
    [HttpGet]
    public async Task<IActionResult> SearchProducts([FromQuery] SearchProductsQuery query)
    {
        var result = await _mediator.Send(query);
        return Ok(result);
    }
    
    [HttpGet("{id:guid}")]
    public async Task<IActionResult> GetProduct(Guid id)
    {
        var product = await _mediator.Send(new GetProductByIdQuery(id));
        return Ok(product);
    }
    
    [HttpPost]
    [Authorize(Roles = "Admin,Vendor")]
    public async Task<IActionResult> CreateProduct([FromBody] CreateProductCommand command)
    {
        var product = await _mediator.Send(command);
        return CreatedAtAction(nameof(GetProduct), new { id = product.Id }, product);
    }
    
    [HttpPut("{id:guid}")]
    [Authorize(Roles = "Admin,Vendor")]
    public async Task<IActionResult> UpdateProduct(Guid id, [FromBody] UpdateProductCommand command)
    {
        if (id != command.Id)
            return BadRequest("Id mismatch");
            
        var product = await _mediator.Send(command);
        return Ok(product);
    }
    
    [HttpPatch("{id:guid}/stock")]
    [Authorize(Roles = "Admin,Vendor")]
    public async Task<IActionResult> UpdateStock(Guid id, [FromBody] UpdateStockCommand command)
    {
        if (id != command.ProductId)
            return BadRequest("Id mismatch");
            
        await _mediator.Send(command);
        return NoContent();
    }
    
    [HttpPost("{id:guid}/reviews")]
    [Authorize]
    public async Task<IActionResult> AddReview(Guid id, [FromBody] AddReviewCommand command)
    {
        if (id != command.ProductId)
            return BadRequest("Id mismatch");
            
        var review = await _mediator.Send(command);
        return CreatedAtAction(nameof(GetProduct), new { id }, review);
    }
}

// FluentValidation Example
public class CreateProductCommandValidator : AbstractValidator<CreateProductCommand>
{
    public CreateProductCommandValidator()
    {
        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("Product name is required")
            .MaximumLength(200).WithMessage("Product name must not exceed 200 characters");
            
        RuleFor(x => x.SKU)
            .NotEmpty().WithMessage("SKU is required")
            .MaximumLength(50).WithMessage("SKU must not exceed 50 characters")
            .Matches(@"^[A-Z0-9\-]+$").WithMessage("SKU must contain only uppercase letters, numbers, and hyphens");
            
        RuleFor(x => x.Price)
            .GreaterThan(0).WithMessage("Price must be greater than 0");
            
        RuleFor(x => x.StockQuantity)
            .GreaterThanOrEqualTo(0).WithMessage("Stock quantity cannot be negative");
            
        RuleFor(x => x.CategoryId)
            .NotEmpty().WithMessage("Category is required");
    }
}

// Register validation in Program.cs
builder.Services.AddValidatorsFromAssemblyContaining<CreateProductCommandValidator>();
```

---

### Chapter 8: Building the Order Microservice

```csharp
// OrderService.Domain/Entities/Order.cs
namespace OrderService.Domain.Entities;

public class Order : Entity, IAggregateRoot
{
    public string OrderNumber { get; private set; }
    public Guid UserId { get; private set; }
    public OrderStatus Status { get; private set; }
    public Address ShippingAddress { get; private set; }
    public Address BillingAddress { get; private set; }
    public Money SubTotal { get; private set; }
    public Money TaxAmount { get; private set; }
    public Money ShippingCost { get; private set; }
    public Money DiscountAmount { get; private set; }
    public Money TotalAmount { get; private set; }
    public string CouponCode { get; private set; }
    public string Notes { get; private set; }
    
    private readonly List<OrderItem> _items = new();
    public IReadOnlyCollection<OrderItem> Items => _items.AsReadOnly();
    
    private readonly List<OrderStatusHistory> _statusHistory = new();
    public IReadOnlyCollection<OrderStatusHistory> StatusHistory => _statusHistory.AsReadOnly();
    
    public DateTime CreatedAt { get; private set; }
    public DateTime UpdatedAt { get; private set; }
    public DateTime? PaidAt { get; private set; }
    public DateTime? ShippedAt { get; private set; }
    public DateTime? DeliveredAt { get; private set; }
    public DateTime? CancelledAt { get; private set; }
    
    private Order() { }
    
    public Order(Guid userId, Address shippingAddress, Address billingAddress, string notes = null)
    {
        Id = Guid.NewGuid();
        OrderNumber = GenerateOrderNumber();
        UserId = userId;
        Status = OrderStatus.Draft;
        ShippingAddress = shippingAddress;
        BillingAddress = billingAddress ?? shippingAddress;
        Notes = notes;
        SubTotal = Money.Zero();
        TaxAmount = Money.Zero();
        ShippingCost = Money.Zero();
        DiscountAmount = Money.Zero();
        TotalAmount = Money.Zero();
        CreatedAt = DateTime.UtcNow;
        UpdatedAt = DateTime.UtcNow;
        
        AddStatusHistory(OrderStatus.Draft, "Order created");
        AddDomainEvent(new OrderCreatedEvent(Id, OrderNumber, UserId));
    }
    
    public void AddItem(Guid productId, string productName, Money unitPrice, int quantity)
    {
        if (Status != OrderStatus.Draft)
            throw new DomainException("Cannot modify order in current status");
            
        if (quantity <= 0)
            throw new DomainException("Quantity must be positive");
            
        var existingItem = _items.FirstOrDefault(i => i.ProductId == productId);
        if (existingItem != null)
        {
            existingItem.UpdateQuantity(existingItem.Quantity + quantity);
        }
        else
        {
            _items.Add(new OrderItem(productId, productName, unitPrice, quantity));
        }
        
        RecalculateTotals();
    }
    
    public void RemoveItem(Guid productId)
    {
        if (Status != OrderStatus.Draft)
            throw new DomainException("Cannot modify order in current status");
            
        var item = _items.FirstOrDefault(i => i.ProductId == productId);
        if (item != null)
        {
            _items.Remove(item);
            RecalculateTotals();
        }
    }
    
    public void ApplyCoupon(string couponCode, Money discountAmount)
    {
        if (Status != OrderStatus.Draft)
            throw new DomainException("Cannot apply coupon in current status");
            
        CouponCode = couponCode;
        DiscountAmount = discountAmount;
        RecalculateTotals();
    }
    
    public void Place()
    {
        if (Status != OrderStatus.Draft)
            throw new DomainException("Order can only be placed from draft status");
            
        if (!_items.Any())
            throw new DomainException("Order must have at least one item");
            
        Status = OrderStatus.Placed;
        UpdatedAt = DateTime.UtcNow;
        AddStatusHistory(OrderStatus.Placed, "Order placed");
        AddDomainEvent(new OrderPlacedEvent(Id, OrderNumber, UserId, TotalAmount, Items.ToList()));
    }
    
    public void Confirm()
    {
        if (Status != OrderStatus.Placed)
            throw new DomainException("Only placed orders can be confirmed");
            
        Status = OrderStatus.Confirmed;
        UpdatedAt = DateTime.UtcNow;
        AddStatusHistory(OrderStatus.Confirmed, "Order confirmed");
    }
    
    public void MarkAsPaid(Guid paymentId)
    {
        if (Status != OrderStatus.Confirmed && Status != OrderStatus.Placed)
            throw new DomainException("Order must be placed or confirmed to mark as paid");
            
        Status = OrderStatus.Paid;
        PaidAt = DateTime.UtcNow;
        UpdatedAt = DateTime.UtcNow;
        AddStatusHistory(OrderStatus.Paid, $"Payment received: {paymentId}");
        AddDomainEvent(new OrderPaidEvent(Id, OrderNumber, paymentId));
    }
    
    public void Ship(string trackingNumber, string carrier)
    {
        if (Status != OrderStatus.Paid)
            throw new DomainException("Only paid orders can be shipped");
            
        Status = OrderStatus.Shipped;
        ShippedAt = DateTime.UtcNow;
        UpdatedAt = DateTime.UtcNow;
        AddStatusHistory(OrderStatus.Shipped, $"Shipped via {carrier}, tracking: {trackingNumber}");
        AddDomainEvent(new OrderShippedEvent(Id, OrderNumber, trackingNumber, carrier));
    }
    
    public void Deliver()
    {
        if (Status != OrderStatus.Shipped)
            throw new DomainException("Only shipped orders can be delivered");
            
        Status = OrderStatus.Delivered;
        DeliveredAt = DateTime.UtcNow;
        UpdatedAt = DateTime.UtcNow;
        AddStatusHistory(OrderStatus.Delivered, "Order delivered");
        AddDomainEvent(new OrderDeliveredEvent(Id, OrderNumber));
    }
    
    public void Cancel(string reason)
    {
        if (Status == OrderStatus.Delivered || Status == OrderStatus.Returned)
            throw new DomainException("Cannot cancel delivered or returned orders");
            
        if (Status == OrderStatus.Shipped)
            throw new DomainException("Shipped orders cannot be cancelled, request return instead");
            
        Status = OrderStatus.Cancelled;
        CancelledAt = DateTime.UtcNow;
        UpdatedAt = DateTime.UtcNow;
        AddStatusHistory(OrderStatus.Cancelled, reason);
        AddDomainEvent(new OrderCancelledEvent(Id, OrderNumber, reason));
    }
    
    public void RequestReturn(string reason)
    {
        if (Status != OrderStatus.Delivered)
            throw new DomainException("Only delivered orders can be returned");
            
        Status = OrderStatus.ReturnRequested;
        UpdatedAt = DateTime.UtcNow;
        AddStatusHistory(OrderStatus.ReturnRequested, reason);
        AddDomainEvent(new OrderReturnRequestedEvent(Id, OrderNumber, reason));
    }
    
    private void RecalculateTotals()
    {
        SubTotal = new Money(_items.Sum(i => i.TotalPrice.Amount), "USD");
        TaxAmount = new Money(SubTotal.Amount * 0.1m, "USD"); // 10% tax
        ShippingCost = new Money(SubTotal.Amount > 100 ? 0 : 9.99m, "USD"); // Free shipping over $100
        TotalAmount = SubTotal.Add(TaxAmount).Add(ShippingCost);
        
        if (DiscountAmount.Amount > 0)
        {
            TotalAmount = new Money(Math.Max(0, TotalAmount.Amount - DiscountAmount.Amount), "USD");
        }
    }
    
    private void AddStatusHistory(OrderStatus status, string comment)
    {
        _statusHistory.Add(new OrderStatusHistory(status, comment));
    }
    
    private static string GenerateOrderNumber()
    {
        return $"ORD-{DateTime.UtcNow:yyyyMMdd}-{Guid.NewGuid().ToString("N")[..8].ToUpper()}";
    }
}

public enum OrderStatus
{
    Draft,
    Placed,
    Confirmed,
    Paid,
    Packed,
    Shipped,
    Delivered,
    Cancelled,
    ReturnRequested,
    Returned
}

// OrderService.Application/Commands/PlaceOrderCommand.cs
public record PlaceOrderCommand(
    Guid UserId,
    AddressDto ShippingAddress,
    AddressDto BillingAddress,
    List<OrderItemRequest> Items,
    string CouponCode = null,
    string Notes = null
) : IRequest<OrderDto>;

public class PlaceOrderCommandHandler : IRequestHandler<PlaceOrderCommand, OrderDto>
{
    private readonly IOrderRepository _orderRepository;
    private readonly IUserServiceClient _userClient;
    private readonly IProductServiceClient _productClient;
    private readonly ICouponServiceClient _couponClient;
    private readonly IMapper _mapper;
    private readonly ILogger<PlaceOrderCommandHandler> _logger;
    
    public PlaceOrderCommandHandler(
        IOrderRepository orderRepository,
        IUserServiceClient userClient,
        IProductServiceClient productClient,
        ICouponServiceClient couponClient,
        IMapper mapper,
        ILogger<PlaceOrderCommandHandler> logger)
    {
        _orderRepository = orderRepository;
        _userClient = userClient;
        _productClient = productClient;
        _couponClient = couponClient;
        _mapper = mapper;
        _logger = logger;
    }
    
    public async Task<OrderDto> Handle(PlaceOrderCommand request, CancellationToken cancellationToken)
    {
        _logger.LogInformation("Placing order for user {UserId}", request.UserId);
        
        // Validate user
        var user = await _userClient.GetUserAsync(request.UserId);
        if (user == null)
            throw new ValidationException("User not found");
            
        // Create order
        var shippingAddress = _mapper.Map<Address>(request.ShippingAddress);
        var billingAddress = _mapper.Map<Address>(request.BillingAddress);
        var order = new Order(request.UserId, shippingAddress, billingAddress, request.Notes);
        
        // Add items with validation
        foreach (var item in request.Items)
        {
            var product = await _productClient.GetProductAsync(item.ProductId);
            if (product == null)
                throw new ValidationException($"Product {item.ProductId} not found");
                
            if (!product.IsAvailable)
                throw new ValidationException($"Product {product.Name} is not available");
                
            if (product.StockQuantity < item.Quantity)
                throw new ValidationException($"Insufficient stock for {product.Name}. Available: {product.StockQuantity}");
                
            var unitPrice = new Money(product.Price, "USD");
            order.AddItem(product.Id, product.Name, unitPrice, item.Quantity);
        }
        
        // Apply coupon if provided
        if (!string.IsNullOrEmpty(request.CouponCode))
        {
            var coupon = await _couponClient.ValidateCouponAsync(request.CouponCode);
            if (coupon != null && coupon.IsValid)
            {
                var discountAmount = new Money(coupon.DiscountAmount, "USD");
                order.ApplyCoupon(request.CouponCode, discountAmount);
            }
        }
        
        // Place order
        order.Place();
        
        // Persist
        await _orderRepository.AddAsync(order);
        
        _logger.LogInformation("Order {OrderNumber} placed successfully", order.OrderNumber);
        
        return _mapper.Map<OrderDto>(order);
    }
}

// OrderService.API/Controllers/OrdersController.cs
[ApiController]
[Route("api/[controller]")]
[Authorize]
public class OrdersController : ControllerBase
{
    private readonly IMediator _mediator;
    
    public OrdersController(IMediator mediator)
    {
        _mediator = mediator;
    }
    
    [HttpPost]
    public async Task<IActionResult> PlaceOrder([FromBody] PlaceOrderCommand command)
    {
        var userId = Guid.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier));
        var orderCommand = command with { UserId = userId };
        var order = await _mediator.Send(orderCommand);
        return CreatedAtAction(nameof(GetOrder), new { id = order.Id }, order);
    }
    
    [HttpGet("{id:guid}")]
    public async Task<IActionResult> GetOrder(Guid id)
    {
        var order = await _mediator.Send(new GetOrderByIdQuery(id));
        
        // Ensure user can only access their own orders unless admin
        var userId = Guid.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier));
        if (order.UserId != userId && !User.IsInRole("Admin"))
            return Forbid();
            
        return Ok(order);
    }
    
    [HttpGet]
    public async Task<IActionResult> GetMyOrders([FromQuery] GetUserOrdersQuery query)
    {
        var userId = Guid.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier));
        var orders = await _mediator.Send(query with { UserId = userId });
        return Ok(orders);
    }
    
    [HttpPost("{id:guid}/cancel")]
    public async Task<IActionResult> CancelOrder(Guid id, [FromBody] CancelOrderCommand command)
    {
        var userId = Guid.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier));
        var cancelCommand = command with { OrderId = id, UserId = userId };
        await _mediator.Send(cancelCommand);
        return NoContent();
    }
    
    [HttpPost("{id:guid}/return")]
    public async Task<IActionResult> RequestReturn(Guid id, [FromBody] RequestReturnCommand command)
    {
        var userId = Guid.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier));
        var returnCommand = command with { OrderId = id, UserId = userId };
        await _mediator.Send(returnCommand);
        return NoContent();
    }
}
```

---

### Chapter 9: Creating the Payment Microservice

```csharp
// PaymentService.Domain/Entities/Payment.cs
namespace PaymentService.Domain.Entities;

public class Payment : Entity, IAggregateRoot
{
    public Guid OrderId { get; private set; }
    public string OrderNumber { get; private set; }
    public Guid UserId { get; private set; }
    public Money Amount { get; private set; }
    public PaymentMethod Method { get; private set; }
    public PaymentStatus Status { get; private set; }
    public string TransactionId { get; private set; }
    public string GatewayReference { get; private set; }
    public string FailureReason { get; private set; }
    public int AttemptCount { get; private set; }
    public DateTime CreatedAt { get; private set; }
    public DateTime UpdatedAt { get; private set; }
    public DateTime? CompletedAt { get; private set; }
    
    private Payment() { }
    
    public Payment(Guid orderId, string orderNumber, Guid userId, Money amount, PaymentMethod method)
    {
        Id = Guid.NewGuid();
        OrderId = orderId;
        OrderNumber = orderNumber;
        UserId = userId;
        Amount = amount;
        Method = method;
        Status = PaymentStatus.Pending;
        CreatedAt = DateTime.UtcNow;
        UpdatedAt = DateTime.UtcNow;
        AttemptCount = 1;
        
        AddDomainEvent(new PaymentInitiatedEvent(Id, OrderId, amount));
    }
    
    public void ProcessPayment(string transactionId, string gatewayReference)
    {
        TransactionId = transactionId;
        GatewayReference = gatewayReference;
        Status = PaymentStatus.Processing;
        UpdatedAt = DateTime.UtcNow;
    }
    
    public void CompletePayment()
    {
        Status = PaymentStatus.Completed;
        CompletedAt = DateTime.UtcNow;
        UpdatedAt = DateTime.UtcNow;
        AddDomainEvent(new PaymentCompletedEvent(Id, OrderId, TransactionId));
    }
    
    public void FailPayment(string reason)
    {
        Status = PaymentStatus.Failed;
        FailureReason = reason;
        UpdatedAt = DateTime.UtcNow;
        AddDomainEvent(new PaymentFailedEvent(Id, OrderId, reason));
    }
    
    public void Retry()
    {
        if (Status != PaymentStatus.Failed)
            throw new DomainException("Can only retry failed payments");
            
        if (AttemptCount >= 3)
            throw new DomainException("Maximum retry attempts reached");
            
        AttemptCount++;
        Status = PaymentStatus.Pending;
        UpdatedAt = DateTime.UtcNow;
    }
    
    public void Refund(Money refundAmount, string reason)
    {
        if (Status != PaymentStatus.Completed)
            throw new DomainException("Can only refund completed payments");
            
        if (refundAmount.Amount > Amount.Amount)
            throw new DomainException("Refund amount cannot exceed payment amount");
            
        Status = refundAmount.Amount == Amount.Amount 
            ? PaymentStatus.FullyRefunded 
            : PaymentStatus.PartiallyRefunded;
            
        UpdatedAt = DateTime.UtcNow;
        AddDomainEvent(new PaymentRefundedEvent(Id, OrderId, refundAmount, reason));
    }
}

public enum PaymentMethod
{
    CreditCard,
    DebitCard,
    NetBanking,
    UPI,
    Wallet,
    CashOnDelivery
}

public enum PaymentStatus
{
    Pending,
    Processing,
    Completed,
    Failed,
    Cancelled,
    PartiallyRefunded,
    FullyRefunded
}

// PaymentService.Application/Services/PaymentGatewayService.cs
public class PaymentGatewayService : IPaymentGatewayService
{
    private readonly IHttpClientFactory _httpClientFactory;
    private readonly IPaymentRepository _repository;
    private readonly ILogger<PaymentGatewayService> _logger;
    
    public PaymentGatewayService(
        IHttpClientFactory httpClientFactory,
        IPaymentRepository repository,
        ILogger<PaymentGatewayService> logger)
    {
        _httpClientFactory = httpClientFactory;
        _repository = repository;
        _logger = logger;
    }
    
    public async Task<PaymentResult> ProcessPaymentAsync(Payment payment, PaymentCardDetails cardDetails)
    {
        try
        {
            _logger.LogInformation("Processing payment {PaymentId} for order {OrderNumber}", 
                payment.Id, payment.OrderNumber);
            
            payment.ProcessPayment(Guid.NewGuid().ToString(), null);
            await _repository.UpdateAsync(payment);
            
            // Simulate payment gateway call
            var result = await SimulatePaymentGatewayCall(payment, cardDetails);
            
            if (result.Success)
            {
                payment.CompletePayment();
                await _repository.UpdateAsync(payment);
            }
            else
            {
                payment.FailPayment(result.ErrorMessage);
                await _repository.UpdateAsync(payment);
            }
            
            return result;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Payment processing failed for payment {PaymentId}", payment.Id);
            payment.FailPayment("Internal processing error");
            await _repository.UpdateAsync(payment);
            
            return new PaymentResult 
            { 
                Success = false, 
                ErrorMessage = "Payment processing failed. Please try again." 
            };
        }
    }
    
    private async Task<PaymentResult> SimulatePaymentGatewayCall(
        Payment payment, PaymentCardDetails cardDetails)
    {
        // Simulate network delay
        await Task.Delay(500);
        
        // Simulate success rate of 90%
        if (Random.Shared.Next(100) < 90)
        {
            return new PaymentResult
            {
                Success = true,
                TransactionId = $"TXN{DateTime.UtcNow:yyyyMMddHHmmss}{Random.Shared.Next(10000):D4}",
                GatewayReference = $"GW-{Guid.NewGuid().ToString("N")[..12]}"
            };
        }
        
        return new PaymentResult
        {
            Success = false,
            ErrorMessage = "Payment declined by bank. Please try again or use a different card."
        };
    }
}

// PaymentService.Application/EventHandlers/OrderPlacedEventHandler.cs
public class OrderPlacedEventHandler : IEventHandler<OrderPlacedIntegrationEvent>
{
    private readonly IMediator _mediator;
    private readonly ILogger<OrderPlacedEventHandler> _logger;
    
    public OrderPlacedEventHandler(IMediator mediator, ILogger<OrderPlacedEventHandler> logger)
    {
        _mediator = mediator;
        _logger = logger;
    }
    
    public async Task Handle(OrderPlacedIntegrationEvent @event)
    {
        _logger.LogInformation("Received OrderPlaced event for order {OrderNumber}", @event.OrderNumber);
        
        // Create payment record
        var command = new CreatePaymentCommand
        {
            OrderId = @event.OrderId,
            OrderNumber = @event.OrderNumber,
            UserId = @event.UserId,
            Amount = @event.TotalAmount,
            Currency = @event.Currency
        };
        
        await _mediator.Send(command);
    }
}

// PaymentService.API/Controllers/PaymentsController.cs
[ApiController]
[Route("api/[controller]")]
[Authorize]
public class PaymentsController : ControllerBase
{
    private readonly IMediator _mediator;
    
    public PaymentsController(IMediator mediator)
    {
        _mediator = mediator;
    }
    
    [HttpPost]
    public async Task<IActionResult> ProcessPayment([FromBody] ProcessPaymentCommand command)
    {
        var userId = Guid.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier));
        var paymentCommand = command with { UserId = userId };
        var result = await _mediator.Send(paymentCommand);
        
        if (result.Success)
            return Ok(result);
            
        return BadRequest(result);
    }
    
    [HttpGet("{id:guid}")]
    public async Task<IActionResult> GetPayment(Guid id)
    {
        var payment = await _mediator.Send(new GetPaymentByIdQuery(id));
        var userId = Guid.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier));
        
        if (payment.UserId != userId && !User.IsInRole("Admin"))
            return Forbid();
            
        return Ok(payment);
    }
    
    [HttpPost("{id:guid}/retry")]
    public async Task<IActionResult> RetryPayment(Guid id)
    {
        var result = await _mediator.Send(new RetryPaymentCommand(id));
        return Ok(result);
    }
    
    [HttpPost("{id:guid}/refund")]
    [Authorize(Roles = "Admin")]
    public async Task<IActionResult> RefundPayment(Guid id, [FromBody] RefundPaymentCommand command)
    {
        var refundCommand = command with { PaymentId = id };
        var result = await _mediator.Send(refundCommand);
        return Ok(result);
    }
}
```

---

### Chapter 10: Creating the Notification Microservice

```csharp
// NotificationService.Domain/Entities/Notification.cs
namespace NotificationService.Domain.Entities;

public class Notification : Entity
{
    public Guid UserId { get; private set; }
    public NotificationType Type { get; private set; }
    public NotificationChannel Channel { get; private set; }
    public string TemplateCode { get; private set; }
    public string Subject { get; private set; }
    public string Body { get; private set; }
    public Dictionary<string, string> Data { get; private set; }
    public NotificationStatus Status { get; private set; }
    public string ErrorMessage { get; private set; }
    public int RetryCount { get; private set; }
    public DateTime CreatedAt { get; private set; }
    public DateTime? SentAt { get; private set; }
    public DateTime? DeliveredAt { get; private set; }
    public DateTime? ReadAt { get; private set; }
    
    private Notification() { }
    
    public Notification(
        Guid userId, 
        NotificationType type, 
        NotificationChannel channel,
        string templateCode,
        string subject,
        string body,
        Dictionary<string, string> data)
    {
        Id = Guid.NewGuid();
        UserId = userId;
        Type = type;
        Channel = channel;
        TemplateCode = templateCode;
        Subject = subject;
        Body = body;
        Data = data ?? new Dictionary<string, string>();
        Status = NotificationStatus.Pending;
        CreatedAt = DateTime.UtcNow;
        RetryCount = 0;
    }
    
    public void MarkAsSent()
    {
        Status = NotificationStatus.Sent;
        SentAt = DateTime.UtcNow;
    }
    
    public void MarkAsDelivered()
    {
        Status = NotificationStatus.Delivered;
        DeliveredAt = DateTime.UtcNow;
    }
    
    public void MarkAsRead()
    {
        if (Status == NotificationStatus.Delivered)
        {
            Status = NotificationStatus.Read;
            ReadAt = DateTime.UtcNow;
        }
    }
    
    public void MarkAsFailed(string errorMessage)
    {
        Status = NotificationStatus.Failed;
        ErrorMessage = errorMessage;
        
        if (RetryCount < 3)
        {
            RetryCount++;
            Status = NotificationStatus.Pending; // Will be retried
        }
    }
}

// NotificationService.Domain/Entities/NotificationTemplate.cs
public class NotificationTemplate : Entity
{
    public string Code { get; private set; }
    public string Name { get; private set; }
    public string SubjectTemplate { get; private set; }
    public string BodyTemplate { get; private set; }
    public NotificationType Type { get; private set; }
    public List<NotificationChannel> SupportedChannels { get; private set; }
    public bool IsActive { get; private set; }
    
    public NotificationTemplate(
        string code, 
        string name, 
        string subjectTemplate, 
        string bodyTemplate,
        NotificationType type,
        List<NotificationChannel> supportedChannels)
    {
        Id = Guid.NewGuid();
        Code = code;
        Name = name;
        SubjectTemplate = subjectTemplate;
        BodyTemplate = bodyTemplate;
        Type = type;
        SupportedChannels = supportedChannels;
        IsActive = true;
    }
    
    public string RenderSubject(Dictionary<string, string> data)
    {
        return RenderTemplate(SubjectTemplate, data);
    }
    
    public string RenderBody(Dictionary<string, string> data)
    {
        return RenderTemplate(BodyTemplate, data);
    }
    
    private string RenderTemplate(string template, Dictionary<string, string> data)
    {
        var result = template;
        foreach (var kvp in data)
        {
            result = result.Replace($"{{{{{kvp.Key}}}}}", kvp.Value);
        }
        return result;
    }
}

// NotificationService.Application/Services/EmailService.cs
public class EmailService : IEmailService
{
    private readonly IEmailSender _emailSender;
    private readonly ILogger<EmailService> _logger;
    
    public EmailService(IEmailSender emailSender, ILogger<EmailService> logger)
    {
        _emailSender = emailSender;
        _logger = logger;
    }
    
    public async Task SendEmailAsync(string to, string subject, string body)
    {
        try
        {
            _logger.LogInformation("Sending email to {Email}, subject: {Subject}", to, subject);
            
            await _emailSender.SendAsync(new EmailMessage
            {
                To = to,
                Subject = subject,
                Body = body,
                IsHtml = true
            });
            
            _logger.LogInformation("Email sent successfully to {Email}", to);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to send email to {Email}", to);
            throw;
        }
    }
}

// NotificationService.Application/EventHandlers/OrderEventsHandler.cs
public class OrderEventsHandler : 
    IEventHandler<OrderPlacedIntegrationEvent>,
    IEventHandler<OrderShippedIntegrationEvent>,
    IEventHandler<OrderDeliveredIntegrationEvent>,
    IEventHandler<OrderCancelledIntegrationEvent>
{
    private readonly INotificationService _notificationService;
    private readonly IUserServiceClient _userClient;
    private readonly ILogger<OrderEventsHandler> _logger;
    
    public OrderEventsHandler(
        INotificationService notificationService,
        IUserServiceClient userClient,
        ILogger<OrderEventsHandler> logger)
    {
        _notificationService = notificationService;
        _userClient = userClient;
        _logger = logger;
    }
    
    public async Task Handle(OrderPlacedIntegrationEvent @event)
    {
        _logger.LogInformation("Handling OrderPlaced event for order {OrderNumber}", @event.OrderNumber);
        
        var user = await _userClient.GetUserAsync(@event.UserId);
        if (user == null) return;
        
        var templateData = new Dictionary<string, string>
        {
            ["UserName"] = user.Name,
            ["OrderNumber"] = @event.OrderNumber,
            ["OrderTotal"] = @event.TotalAmount.ToString("C"),
            ["OrderDate"] = @event.OccurredOn.ToString("f"),
            ["ItemCount"] = @event.ItemCount.ToString()
        };
        
        await _notificationService.SendNotificationAsync(
            @event.UserId,
            user.Email,
            user.PhoneNumber,
            NotificationType.OrderConfirmation,
            "order-confirmation",
            templateData
        );
    }
    
    public async Task Handle(OrderShippedIntegrationEvent @event)
    {
        var user = await _userClient.GetUserAsync(@event.UserId);
        if (user == null) return;
        
        var templateData = new Dictionary<string, string>
        {
            ["UserName"] = user.Name,
            ["OrderNumber"] = @event.OrderNumber,
            ["TrackingNumber"] = @event.TrackingNumber,
            ["Carrier"] = @event.Carrier,
            ["EstimatedDelivery"] = @event.EstimatedDelivery?.ToString("d")
        };
        
        await _notificationService.SendNotificationAsync(
            @event.UserId,
            user.Email,
            user.PhoneNumber,
            NotificationType.ShipmentUpdate,
            "order-shipped",
            templateData
        );
    }
    
    public async Task Handle(OrderDeliveredIntegrationEvent @event)
    {
        var user = await _userClient.GetUserAsync(@event.UserId);
        if (user == null) return;
        
        var templateData = new Dictionary<string, string>
        {
            ["UserName"] = user.Name,
            ["OrderNumber"] = @event.OrderNumber,
            ["DeliveryDate"] = @event.DeliveredAt.ToString("f")
        };
        
        await _notificationService.SendNotificationAsync(
            @event.UserId,
            user.Email,
            user.PhoneNumber,
            NotificationType.DeliveryConfirmation,
            "order-delivered",
            templateData
        );
    }
    
    public async Task Handle(OrderCancelledIntegrationEvent @event)
    {
        var user = await _userClient.GetUserAsync(@event.UserId);
        if (user == null) return;
        
        var templateData = new Dictionary<string, string>
        {
            ["UserName"] = user.Name,
            ["OrderNumber"] = @event.OrderNumber,
            ["CancellationReason"] = @event.Reason
        };
        
        await _notificationService.SendNotificationAsync(
            @event.UserId,
            user.Email,
            user.PhoneNumber,
            NotificationType.OrderCancelled,
            "order-cancelled",
            templateData
        );
    }
}

// NotificationService.Infrastructure/Services/SmsService.cs
public class SmsService : ISmsService
{
    private readonly IHttpClientFactory _httpClientFactory;
    private readonly SmsSettings _settings;
    private readonly ILogger<SmsService> _logger;
    
    public SmsService(
        IHttpClientFactory httpClientFactory,
        IOptions<SmsSettings> settings,
        ILogger<SmsService> logger)
    {
        _httpClientFactory = httpClientFactory;
        _settings = settings.Value;
        _logger = logger;
    }
    
    public async Task SendSmsAsync(string phoneNumber, string message)
    {
        try
        {
            _logger.LogInformation("Sending SMS to {PhoneNumber}", phoneNumber[..4] + "****");
            
            var client = _httpClientFactory.CreateClient("SmsProvider");
            
            var payload = new
            {
                apiKey = _settings.ApiKey,
                to = phoneNumber,
                message = message,
                senderId = _settings.SenderId
            };
            
            var response = await client.PostAsJsonAsync("send", payload);
            response.EnsureSuccessStatusCode();
            
            _logger.LogInformation("SMS sent successfully to {PhoneNumber}", phoneNumber[..4] + "****");
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to send SMS to {PhoneNumber}", phoneNumber[..4] + "****");
            throw;
        }
    }
}

// NotificationService.API/Controllers/NotificationsController.cs
[ApiController]
[Route("api/[controller]")]
[Authorize]
public class NotificationsController : ControllerBase
{
    private readonly IMediator _mediator;
    
    public NotificationsController(IMediator mediator)
    {
        _mediator = mediator;
    }
    
    [HttpGet]
    public async Task<IActionResult> GetNotifications([FromQuery] GetNotificationsQuery query)
    {
        var userId = Guid.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier));
        var notifications = await _mediator.Send(query with { UserId = userId });
        return Ok(notifications);
    }
    
    [HttpPut("{id:guid}/read")]
    public async Task<IActionResult> MarkAsRead(Guid id)
    {
        await _mediator.Send(new MarkNotificationAsReadCommand(id));
        return NoContent();
    }
    
    [HttpPut("read-all")]
    public async Task<IActionResult> MarkAllAsRead()
    {
        var userId = Guid.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier));
        await _mediator.Send(new MarkAllNotificationsAsReadCommand(userId));
        return NoContent();
    }
    
    [HttpGet("preferences")]
    public async Task<IActionResult> GetPreferences()
    {
        var userId = Guid.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier));
        var preferences = await _mediator.Send(new GetNotificationPreferencesQuery(userId));
        return Ok(preferences);
    }
    
    [HttpPut("preferences")]
    public async Task<IActionResult> UpdatePreferences([FromBody] UpdateNotificationPreferencesCommand command)
    {
        var userId = Guid.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier));
        await _mediator.Send(command with { UserId = userId });
        return NoContent();
    }
}
```

---

## Module 3: Inter-Service Communication

### Chapter 11: Inter-Service Communication in Microservices

```csharp
// Synchronous Communication - REST Client

// BuildingBlocks/Common.Infrastructure/Http/HttpClientService.cs
public class HttpClientService : IHttpClientService
{
    private readonly IHttpClientFactory _httpClientFactory;
    private readonly ILogger<HttpClientService> _logger;
    
    public HttpClientService(
        IHttpClientFactory httpClientFactory,
        ILogger<HttpClientService> logger)
    {
        _httpClientFactory = httpClientFactory;
        _logger = logger;
    }
    
    public async Task<T> GetAsync<T>(string clientName, string endpoint)
    {
        var client = _httpClientFactory.CreateClient(clientName);
        
        try
        {
            var response = await client.GetAsync(endpoint);
            response.EnsureSuccessStatusCode();
            
            var content = await response.Content.ReadAsStringAsync();
            return JsonSerializer.Deserialize<T>(content, new JsonSerializerOptions
            {
                PropertyNameCaseInsensitive = true
            });
        }
        catch (HttpRequestException ex)
        {
            _logger.LogError(ex, "HTTP GET failed for {ClientName}{Endpoint}", clientName, endpoint);
            throw new ServiceCommunicationException($"Failed to communicate with {clientName}", ex);
        }
    }
    
    public async Task<T> PostAsync<T>(string clientName, string endpoint, object data)
    {
        var client = _httpClientFactory.CreateClient(clientName);
        
        try
        {
            var response = await client.PostAsJsonAsync(endpoint, data);
            response.EnsureSuccessStatusCode();
            
            var content = await response.Content.ReadAsStringAsync();
            return JsonSerializer.Deserialize<T>(content, new JsonSerializerOptions
            {
                PropertyNameCaseInsensitive = true
            });
        }
        catch (HttpRequestException ex)
        {
            _logger.LogError(ex, "HTTP POST failed for {ClientName}{Endpoint}", clientName, endpoint);
            throw new ServiceCommunicationException($"Failed to communicate with {clientName}", ex);
        }
    }
}

// Service Client Implementation
// OrderService.Infrastructure/Clients/ProductServiceClient.cs
public class ProductServiceClient : IProductServiceClient
{
    private readonly IHttpClientService _httpClient;
    private readonly ICircuitBreakerPolicy _circuitBreaker;
    
    public ProductServiceClient(
        IHttpClientService httpClient,
        ICircuitBreakerPolicy circuitBreaker)
    {
        _httpClient = httpClient;
        _circuitBreaker = circuitBreaker;
    }
    
    public async Task<ProductDto> GetProductAsync(Guid productId)
    {
        return await _circuitBreaker.ExecuteAsync(async () =>
        {
            return await _httpClient.GetAsync<ProductDto>(
                "ProductService",
                $"api/products/{productId}"
            );
        });
    }
    
    public async Task<bool> CheckStockAsync(Guid productId, int quantity)
    {
        var product = await GetProductAsync(productId);
        return product?.StockQuantity >= quantity;
    }
}

// Register HttpClient in Program.cs
builder.Services.AddHttpClient("ProductService", client =>
{
    client.BaseAddress = new Uri(builder.Configuration["ServiceUrls:ProductService"]);
    client.DefaultRequestHeaders.Add("Accept", "application/json");
    client.Timeout = TimeSpan.FromSeconds(10);
})
.AddPolicyHandler(GetRetryPolicy())
.AddPolicyHandler(GetCircuitBreakerPolicy());

// Retry Policy
static IAsyncPolicy<HttpResponseMessage> GetRetryPolicy()
{
    return HttpPolicyExtensions
        .HandleTransientHttpError()
        .Or<TimeoutRejectedException>()
        .WaitAndRetryAsync(
            3,
            retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)),
            onRetry: (outcome, timespan, retryAttempt, context) =>
            {
                // Log retry
            });
}

// Circuit Breaker Policy
static IAsyncPolicy<HttpResponseMessage> GetCircuitBreakerPolicy()
{
    return HttpPolicyExtensions
        .HandleTransientHttpError()
        .CircuitBreakerAsync(
            5,
            TimeSpan.FromSeconds(30),
            onBreak: (outcome, timespan) =>
            {
                // Log circuit break
            },
            onReset: () =>
            {
                // Log circuit reset
            });
}
```

---

### Chapter 12: Data Management Strategies in Microservices

#### Saga Pattern Implementation

```csharp
// Saga Orchestrator for Order Processing
// OrderService.Application/Sagas/OrderProcessingSaga.cs
public class OrderProcessingSaga : ISaga
{
    private readonly IMediator _mediator;
    private readonly IMessageBus _messageBus;
    private readonly ILogger<OrderProcessingSaga> _logger;
    private readonly Dictionary<Guid, SagaState> _activeSagas = new();
    
    public OrderProcessingSaga(
        IMediator mediator,
        IMessageBus messageBus,
        ILogger<OrderProcessingSaga> logger)
    {
        _mediator = mediator;
        _messageBus = messageBus;
        _logger = logger;
    }
    
    public async Task StartAsync(OrderPlacedEvent @event)
    {
        var sagaId = Guid.NewGuid();
        var sagaState = new SagaState
        {
            SagaId = sagaId,
            OrderId = @event.OrderId,
            Status = SagaStatus.Pending,
            StartedAt = DateTime.UtcNow
        };
        
        _activeSagas[sagaId] = sagaState;
        _logger.LogInformation("Starting saga {SagaId} for order {OrderId}", sagaId, @event.OrderId);
        
        try
        {
            // Step 1: Reserve Inventory
            await ReserveInventoryAsync(sagaState, @event);
            
            // Step 2: Process Payment
            await ProcessPaymentAsync(sagaState, @event);
            
            // Step 3: Confirm Order
            await ConfirmOrderAsync(sagaState);
            
            sagaState.Status = SagaStatus.Completed;
            sagaState.CompletedAt = DateTime.UtcNow;
            _logger.LogInformation("Saga {SagaId} completed successfully", sagaId);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Saga {SagaId} failed, initiating compensation", sagaId);
            await CompensateAsync(sagaState);
        }
        finally
        {
            _activeSagas.Remove(sagaId);
        }
    }
    
    private async Task ReserveInventoryAsync(SagaState state, OrderPlacedEvent @event)
    {
        state.CurrentStep = "ReserveInventory";
        
        var command = new ReserveInventoryCommand
        {
            OrderId = @event.OrderId,
            Items = @event.Items.Select(i => new InventoryItem
            {
                ProductId = i.ProductId,
                Quantity = i.Quantity
            }).ToList()
        };
        
        await _messageBus.PublishAsync(command);
        
        // Wait for response (simplified - in real scenario use correlation)
        await Task.Delay(1000);
        
        state.CompletedSteps.Add("ReserveInventory");
    }
    
    private async Task ProcessPaymentAsync(SagaState state, OrderPlacedEvent @event)
    {
        state.CurrentStep = "ProcessPayment";
        
        var command = new ProcessPaymentCommand
        {
            OrderId = @event.OrderId,
            UserId = @event.UserId,
            Amount = @event.TotalAmount
        };
        
        await _messageBus.PublishAsync(command);
        
        // Wait for response
        await Task.Delay(1000);
        
        state.CompletedSteps.Add("ProcessPayment");
    }
    
    private async Task ConfirmOrderAsync(SagaState state)
    {
        state.CurrentStep = "ConfirmOrder";
        
        var command = new ConfirmOrderCommand { OrderId = state.OrderId };
        await _mediator.Send(command);
        
        state.CompletedSteps.Add("ConfirmOrder");
    }
    
    private async Task CompensateAsync(SagaState state)
    {
        _logger.LogWarning("Compensating saga {SagaId}, completed steps: {Steps}", 
            state.SagaId, string.Join(", ", state.CompletedSteps));
        
        foreach (var step in state.CompletedSteps.AsEnumerable().Reverse())
        {
            try
            {
                switch (step)
                {
                    case "ProcessPayment":
                        await RefundPaymentAsync(state.OrderId);
                        break;
                    case "ReserveInventory":
                        await ReleaseInventoryAsync(state.OrderId);
                        break;
                }
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Compensation step {Step} failed for saga {SagaId}", step, state.SagaId);
            }
        }
        
        state.Status = SagaStatus.Compensated;
    }
    
    private async Task RefundPaymentAsync(Guid orderId)
    {
        await _messageBus.PublishAsync(new RefundPaymentCommand { OrderId = orderId });
    }
    
    private async Task ReleaseInventoryAsync(Guid orderId)
    {
        await _messageBus.PublishAsync(new ReleaseInventoryCommand { OrderId = orderId });
    }
}

// Choreography-based Saga using Events
public class OrderSagaParticipant : 
    IEventHandler<OrderPlacedEvent>,
    IEventHandler<PaymentCompletedEvent>,
    IEventHandler<InventoryReservedEvent>,
    IEventHandler<PaymentFailedEvent>,
    IEventHandler<InventoryReservationFailedEvent>
{
    private readonly IMessageBus _messageBus;
    
    // Happy path
    public async Task Handle(OrderPlacedEvent @event)
    {
        // Publish event to reserve inventory
        await _messageBus.PublishAsync(new ReserveInventoryCommand
        {
            OrderId = @event.OrderId,
            Items = @event.Items
        });
    }
    
    public async Task Handle(InventoryReservedEvent @event)
    {
        // When inventory is reserved, process payment
        await _messageBus.PublishAsync(new ProcessPaymentCommand
        {
            OrderId = @event.OrderId,
            Amount = @event.TotalAmount
        });
    }
    
    public async Task Handle(PaymentCompletedEvent @event)
    {
        // When payment completes, confirm order
        await _messageBus.PublishAsync(new ConfirmOrderCommand
        {
            OrderId = @event.OrderId
        });
    }
    
    // Compensation
    public async Task Handle(PaymentFailedEvent @event)
    {
        // If payment fails, release inventory
        await _messageBus.PublishAsync(new ReleaseInventoryCommand
        {
            OrderId = @event.OrderId,
            Reason = "Payment failed"
        });
    }
    
    public async Task Handle(InventoryReservationFailedEvent @event)
    {
        // If inventory reservation fails, cancel order
        await _messageBus.PublishAsync(new CancelOrderCommand
        {
            OrderId = @event.OrderId,
            Reason = "Insufficient inventory"
        });
    }
}
```

---

### Chapter 13: API Gateway and Service Discovery

```json
// Ocelot API Gateway Configuration
// ApiGateway/ocelot.json
{
  "Routes": [
    {
      "DownstreamPathTemplate": "/api/users/{everything}",
      "DownstreamScheme": "http",
      "DownstreamHostAndPorts": [
        {
          "Host": "localhost",
          "Port": 5001
        }
      ],
      "UpstreamPathTemplate": "/api/users/{everything}",
      "UpstreamHttpMethod": [ "GET", "POST", "PUT", "DELETE" ],
      "AuthenticationOptions": {
        "AuthenticationProviderKey": "Bearer",
        "AllowedScopes": []
      },
      "RateLimitOptions": {
        "ClientWhitelist": [],
        "EnableRateLimiting": true,
        "Period": "1m",
        "PeriodTimespan": 1,
        "Limit": 100
      }
    },
    {
      "DownstreamPathTemplate": "/api/products/{everything}",
      "DownstreamScheme": "http",
      "DownstreamHostAndPorts": [
        {
          "Host": "localhost",
          "Port": 5002
        }
      ],
      "UpstreamPathTemplate": "/api/products/{everything}",
      "UpstreamHttpMethod": [ "GET", "POST", "PUT", "DELETE" ]
    },
    {
      "DownstreamPathTemplate": "/api/orders/{everything}",
      "DownstreamScheme": "http",
      "DownstreamHostAndPorts": [
        {
          "Host": "localhost",
          "Port": 5003
        }
      ],
      "UpstreamPathTemplate": "/api/orders/{everything}",
      "UpstreamHttpMethod": [ "GET", "POST", "PUT", "DELETE" ],
      "AuthenticationOptions": {
        "AuthenticationProviderKey": "Bearer"
      }
    },
    {
      "DownstreamPathTemplate": "/api/payments/{everything}",
      "DownstreamScheme": "http",
      "DownstreamHostAndPorts": [
        {
          "Host": "localhost",
          "Port": 5004
        }
      ],
      "UpstreamPathTemplate": "/api/payments/{everything}",
      "UpstreamHttpMethod": [ "GET", "POST" ],
      "AuthenticationOptions": {
        "AuthenticationProviderKey": "Bearer"
      }
    }
  ],
  "Aggregates": [
    {
      "RouteKeys": [
        "user-details",
        "user-orders"
      ],
      "UpstreamPathTemplate": "/api/user-profile/{userId}"
    }
  ],
  "GlobalConfiguration": {
    "BaseUrl": "https://localhost:5000",
    "ServiceDiscoveryProvider": {
      "Scheme": "http",
      "Host": "localhost",
      "Port": 8500,
      "Type": "Consul"
    },
    "RateLimitOptions": {
      "DisableRateLimitHeaders": false,
      "QuotaExceededMessage": "Too many requests. Please try again later.",
      "HttpStatusCode": 429
    }
  }
}
```

```csharp
// ApiGateway/Program.cs
var builder = WebApplication.CreateBuilder(args);

// Add Ocelot
builder.Configuration.AddJsonFile("ocelot.json", optional: false, reloadOnChange: true);
builder.Services.AddOcelot(builder.Configuration);

// Add Consul service discovery
builder.Services.AddOcelot()
    .AddConsul()
    .AddConfigStoredInConsul();

// Add authentication
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidAudience = builder.Configuration["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Secret"]))
        };
    });

// Add CORS
builder.Services.AddCors(options =>
{
    options.AddPolicy("GatewayCors", policy =>
    {
        policy.AllowAnyOrigin()
              .AllowAnyMethod()
              .AllowAnyHeader();
    });
});

var app = builder.Build();

app.UseCors("GatewayCors");
app.UseAuthentication();
app.UseAuthorization();

await app.UseOcelot();

app.Run();
```

---

### Chapter 14: Rate Limiting / Throttling

```csharp
// Custom Rate Limiting Middleware
public class RateLimitingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IRateLimitService _rateLimitService;
    
    public RateLimitingMiddleware(RequestDelegate next, IRateLimitService rateLimitService)
    {
        _next = next;
        _rateLimitService = rateLimitService;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        var clientId = GetClientIdentifier(context);
        
        if (await _rateLimitService.IsRateLimitedAsync(clientId))
        {
            context.Response.StatusCode = StatusCodes.Status429TooManyRequests;
            context.Response.Headers.Add("Retry-After", "60");
            
            await context.Response.WriteAsJsonAsync(new
            {
                error = "Too many requests",
                message = "Please try again later",
                retryAfter = 60
            });
            
            return;
        }
        
        await _next(context);
    }
    
    private string GetClientIdentifier(HttpContext context)
    {
        // Try API key first
        var apiKey = context.Request.Headers["X-API-Key"].FirstOrDefault();
        if (!string.IsNullOrEmpty(apiKey))
            return $"apikey:{apiKey}";
            
        // Try authenticated user
        if (context.User?.Identity?.IsAuthenticated == true)
            return $"user:{context.User.FindFirstValue(ClaimTypes.NameIdentifier)}";
            
        // Fall back to IP address
        return $"ip:{context.Connection.RemoteIpAddress}";
    }
}

// Token Bucket Algorithm Implementation
public class TokenBucketRateLimiter : IRateLimitService
{
    private readonly ConcurrentDictionary<string, TokenBucket> _buckets = new();
    private readonly IOptions<RateLimitOptions> _options;
    
    public TokenBucketRateLimiter(IOptions<RateLimitOptions> options)
    {
        _options = options;
    }
    
    public async Task<bool> IsRateLimitedAsync(string clientId)
    {
        var bucket = _buckets.GetOrAdd(clientId, CreateNewBucket);
        
        // Refill tokens based on elapsed time
        var now = DateTime.UtcNow;
        var elapsedTime = now - bucket.LastRefillTime;
        var tokensToAdd = elapsedTime.TotalSeconds * _options.Value.TokensPerSecond;
        
        bucket.Tokens = Math.Min(bucket.Capacity, bucket.Tokens + tokensToAdd);
        bucket.LastRefillTime = now;
        
        if (bucket.Tokens >= 1)
        {
            bucket.Tokens--;
            return false; // Not rate limited
        }
        
        return true; // Rate limited
    }
    
    private TokenBucket CreateNewBucket(string clientId)
    {
        return new TokenBucket
        {
            Tokens = _options.Value.BucketCapacity,
            Capacity = _options.Value.BucketCapacity,
            LastRefillTime = DateTime.UtcNow
        };
    }
    
    private class TokenBucket
    {
        public double Tokens { get; set; }
        public double Capacity { get; set; }
        public DateTime LastRefillTime { get; set; }
    }
}

// Rate Limiting with ASP.NET Core Middleware
public static class RateLimiterExtensions
{
    public static IServiceCollection AddRateLimiting(this IServiceCollection services, 
        Action<RateLimitOptions> configureOptions)
    {
        services.Configure(configureOptions);
        services.AddSingleton<IRateLimitService, TokenBucketRateLimiter>();
        services.AddMemoryCache();
        
        return services;
    }
    
    public static IApplicationBuilder UseRateLimiting(this IApplicationBuilder app)
    {
        return app.UseMiddleware<RateLimitingMiddleware>();
    }
}

// Register in Program.cs
builder.Services.AddRateLimiting(options =>
{
    options.BucketCapacity = 100;
    options.TokensPerSecond = 10;
    options.DefaultLimit = 100;
    options.Period = TimeSpan.FromMinutes(1);
});

app.UseRateLimiting();

// Rate Limiting with Polly
public static class PollyRateLimiting
{
    public static IAsyncPolicy GetRateLimitPolicy()
    {
        return Policy.RateLimitAsync(
            numberOfExecutions: 100,
            perTimeSpan: TimeSpan.FromMinutes(1),
            maxBurst: 20
        );
    }
}
```

---

### Chapter 15: Security in Microservices

```csharp
// JWT Authentication Setup
// Program.cs (each microservice)
builder.Services.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
})
.AddJwtBearer(options =>
{
    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuer = true,
        ValidateAudience = true,
        ValidateLifetime = true,
        ValidateIssuerSigningKey = true,
        ValidIssuer = builder.Configuration["Jwt:Issuer"],
        ValidAudience = builder.Configuration["Jwt:Audience"],
        IssuerSigningKey = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Secret"])),
        ClockSkew = TimeSpan.Zero
    };
    
    options.Events = new JwtBearerEvents
    {
        OnTokenValidated = async context =>
        {
            // Additional validation logic
            var userService = context.HttpContext.RequestServices
                .GetRequiredService<IUserService>();
            
            var userId = context.Principal.FindFirstValue(ClaimTypes.NameIdentifier);
            if (userId == null)
            {
                context.Fail("Invalid token");
                return;
            }
            
            var user = await userService.GetUserByIdAsync(Guid.Parse(userId));
            if (user == null || !user.IsActive)
            {
                context.Fail("User not found or inactive");
            }
        }
    };
});

// Role-Based Authorization
[Authorize(Roles = "Admin")]
[ApiController]
[Route("api/admin/[controller]")]
public class AdminController : ControllerBase
{
    // Only accessible by Admin role
}

// Policy-Based Authorization
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("RequireAdminOrVendor", policy =>
        policy.RequireRole("Admin", "Vendor"));
        
    options.AddPolicy("CanManageProducts", policy =>
        policy.RequireAssertion(context =>
            context.User.IsInRole("Admin") ||
            (context.User.IsInRole("Vendor") && 
             context.User.HasClaim("Permission", "ManageProducts"))));
            
    options.AddPolicy("AtLeast21", policy =>
        policy.Requirements.Add(new MinimumAgeRequirement(21)));
});

[Authorize(Policy = "CanManageProducts")]
[HttpPost]
public async Task<IActionResult> CreateProduct([FromBody] CreateProductCommand command)
{
    // Only users with CanManageProducts policy can access
}

// Secure Inter-Service Communication
public class ServiceAuthenticationHandler : DelegatingHandler
{
    private readonly ITokenService _tokenService;
    
    public ServiceAuthenticationHandler(ITokenService tokenService)
    {
        _tokenService = tokenService;
    }
    
    protected override async Task<HttpResponseMessage> SendAsync(
        HttpRequestMessage request, CancellationToken cancellationToken)
    {
        // Generate service-to-service token
        var token = _tokenService.GenerateServiceToken();
        request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", token);
        
        return await base.SendAsync(request, cancellationToken);
    }
}

// Register service client with authentication
builder.Services.AddHttpClient("OrderService", client =>
{
    client.BaseAddress = new Uri(builder.Configuration["ServiceUrls:OrderService"]);
})
.AddHttpMessageHandler<ServiceAuthenticationHandler>();

// API Key Authentication
public class ApiKeyAuthenticationHandler : AuthenticationHandler<ApiKeyAuthenticationOptions>
{
    private readonly IApiKeyService _apiKeyService;
    
    public ApiKeyAuthenticationHandler(
        IOptionsMonitor<ApiKeyAuthenticationOptions> options,
        ILoggerFactory logger,
        UrlEncoder encoder,
        ISystemClock clock,
        IApiKeyService apiKeyService)
        : base(options, logger, encoder, clock)
    {
        _apiKeyService = apiKeyService;
    }
    
    protected override async Task<AuthenticateResult> HandleAuthenticateAsync()
    {
        if (!Request.Headers.TryGetValue("X-API-Key", out var apiKeyValues))
        {
            return AuthenticateResult.NoResult();
        }
        
        var apiKey = apiKeyValues.FirstOrDefault();
        if (string.IsNullOrEmpty(apiKey))
        {
            return AuthenticateResult.NoResult();
        }
        
        var isValid = await _apiKeyService.ValidateApiKeyAsync(apiKey);
        if (!isValid)
        {
            return AuthenticateResult.Fail("Invalid API key");
        }
        
        var claims = new[]
        {
            new Claim(ClaimTypes.AuthenticationMethod, "ApiKey"),
            new Claim("ApiKey", apiKey)
        };
        
        var identity = new ClaimsIdentity(claims, Scheme.Name);
        var principal = new ClaimsPrincipal(identity);
        var ticket = new AuthenticationTicket(principal, Scheme.Name);
        
        return AuthenticateResult.Success(ticket);
    }
}

// Secrets Management with Azure Key Vault
builder.Configuration.AddAzureKeyVault(
    new Uri($"https://{builder.Configuration["KeyVault:VaultName"]}.vault.azure.net/"),
    new DefaultAzureCredential()
);

// Or with environment variables for local development
builder.Configuration.AddEnvironmentVariables();

// Encryption Service
public class EncryptionService : IEncryptionService
{
    private readonly byte[] _key;
    
    public EncryptionService(IConfiguration configuration)
    {
        _key = Convert.FromBase64String(configuration["Encryption:Key"]);
    }
    
    public string Encrypt(string plainText)
    {
        using var aes = Aes.Create();
        aes.Key = _key;
        aes.GenerateIV();
        
        using var encryptor = aes.CreateEncryptor();
        var plainBytes = Encoding.UTF8.GetBytes(plainText);
        var cipherBytes = encryptor.TransformFinalBlock(plainBytes, 0, plainBytes.Length);
        
        // Prepend IV to ciphertext
        var result = new byte[aes.IV.Length + cipherBytes.Length];
        Array.Copy(aes.IV, 0, result, 0, aes.IV.Length);
        Array.Copy(cipherBytes, 0, result, aes.IV.Length, cipherBytes.Length);
        
        return Convert.ToBase64String(result);
    }
    
    public string Decrypt(string cipherText)
    {
        var fullCipher = Convert.FromBase64String(cipherText);
        
        using var aes = Aes.Create();
        aes.Key = _key;
        
        // Extract IV
        var iv = new byte[16];
        Array.Copy(fullCipher, 0, iv, 0, iv.Length);
        aes.IV = iv;
        
        // Extract ciphertext
        var cipherBytes = new byte[fullCipher.Length - iv.Length];
        Array.Copy(fullCipher, iv.Length, cipherBytes, 0, cipherBytes.Length);
        
        using var decryptor = aes.CreateDecryptor();
        var plainBytes = decryptor.TransformFinalBlock(cipherBytes, 0, cipherBytes.Length);
        
        return Encoding.UTF8.GetString(plainBytes);
    }
}
```

---

### Chapter 16: Performance Optimization with Caching

```csharp
// Redis Cache Setup
// Program.cs
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = builder.Configuration.GetConnectionString("Redis");
    options.InstanceName = "ECommerce:";
});

// Or using IDistributedCache directly
builder.Services.AddDistributedRedisCache(options =>
{
    options.Configuration = builder.Configuration.GetConnectionString("Redis");
    options.InstanceName = "ECommerce:";
});

// Cache Service Implementation
public class CacheService : ICacheService
{
    private readonly IDistributedCache _cache;
    private readonly ILogger<CacheService> _logger;
    private readonly JsonSerializerOptions _jsonOptions;
    
    public CacheService(IDistributedCache cache, ILogger<CacheService> logger)
    {
        _cache = cache;
        _logger = logger;
        _jsonOptions = new JsonSerializerOptions
        {
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase
        };
    }
    
    public async Task<T> GetAsync<T>(string key) where T : class
    {
        try
        {
            var cachedData = await _cache.GetStringAsync(key);
            
            if (string.IsNullOrEmpty(cachedData))
                return null;
                
            return JsonSerializer.Deserialize<T>(cachedData, _jsonOptions);
        }
        catch (Exception ex)
        {
            _logger.LogWarning(ex, "Failed to get cached value for key {Key}", key);
            return null;
        }
    }
    
    public async Task SetAsync<T>(string key, T value, TimeSpan? expiry = null) where T : class
    {
        try
        {
            var options = new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = expiry ?? TimeSpan.FromMinutes(5)
            };
            
            var serializedData = JsonSerializer.Serialize(value, _jsonOptions);
            await _cache.SetStringAsync(key, serializedData, options);
        }
        catch (Exception ex)
        {
            _logger.LogWarning(ex, "Failed to set cached value for key {Key}", key);
        }
    }
    
    public async Task RemoveAsync(string key)
    {
        try
        {
            await _cache.RemoveAsync(key);
        }
        catch (Exception ex)
        {
            _logger.LogWarning(ex, "Failed to remove cached value for key {Key}", key);
        }
    }
    
    public async Task<T> GetOrSetAsync<T>(string key, Func<Task<T>> factory, TimeSpan? expiry = null) 
        where T : class
    {
        var cachedValue = await GetAsync<T>(key);
        if (cachedValue != null)
        {
            _logger.LogDebug("Cache hit for key {Key}", key);
            return cachedValue;
        }
        
        _logger.LogDebug("Cache miss for key {Key}", key);
        
        var value = await factory();
        
        if (value != null)
        {
            await SetAsync(key, value, expiry);
        }
        
        return value;
    }
}

// Cache-Aside Pattern in Repository
public class CachedProductRepository : IProductRepository
{
    private readonly IProductRepository _decorated;
    private readonly ICacheService _cache;
    private static readonly TimeSpan CacheExpiry = TimeSpan.FromMinutes(10);
    
    public CachedProductRepository(IProductRepository decorated, ICacheService cache)
    {
        _decorated = decorated;
        _cache = cache;
    }
    
    public async Task<Product> GetByIdAsync(Guid id)
    {
        var cacheKey = $"product:{id}";
        
        return await _cache.GetOrSetAsync(
            cacheKey,
            () => _decorated.GetByIdAsync(id),
            CacheExpiry
        );
    }
    
    public async Task AddAsync(Product product)
    {
        await _decorated.AddAsync(product);
        // No need to cache on add
    }
    
    public async Task UpdateAsync(Product product)
    {
        await _decorated.UpdateAsync(product);
        
        // Invalidate cache
        await _cache.RemoveAsync($"product:{product.Id}");
    }
    
    public async Task DeleteAsync(Guid id)
    {
        await _decorated.DeleteAsync(id);
        
        // Invalidate cache
        await _cache.RemoveAsync($"product:{id}");
    }
}

// Register with decorator pattern
builder.Services.AddScoped<IProductRepository, ProductRepository>();
builder.Services.Decorate<IProductRepository, CachedProductRepository>();

// Output Cache for API Responses
builder.Services.AddOutputCache(options =>
{
    options.DefaultExpirationTimeSpan = TimeSpan.FromMinutes(5);
    
    options.AddPolicy("ProductList", builder =>
        builder.Expire(TimeSpan.FromMinutes(2))
               .SetVaryByQuery("categoryId", "search", "page")
               .Tag("products"));
               
    options.AddPolicy("ProductDetail", builder =>
        builder.Expire(TimeSpan.FromMinutes(10))
               .SetVaryByRouteValue("id")
               .Tag("products"));
});

// In Controller
[HttpGet]
[OutputCache(PolicyName = "ProductList")]
public async Task<IActionResult> GetProducts([FromQuery] SearchProductsQuery query)
{
    var products = await _mediator.Send(query);
    return Ok(products);
}

[HttpGet("{id:guid}")]
[OutputCache(PolicyName = "ProductDetail")]
public async Task<IActionResult> GetProduct(Guid id)
{
    var product = await _mediator.Send(new GetProductByIdQuery(id));
    return Ok(product);
}

// Cache Invalidation
[HttpPut("{id:guid}")]
public async Task<IActionResult> UpdateProduct(Guid id, [FromBody] UpdateProductCommand command)
{
    await _mediator.Send(command);
    
    // Invalidate output cache
    await _outputCacheStore.EvictByTagAsync("products", default);
    
    return NoContent();
}

// Response Caching Middleware
builder.Services.AddResponseCaching();

app.UseResponseCaching();

[HttpGet("categories")]
[ResponseCache(Duration = 300, Location = ResponseCacheLocation.Any, VaryByQueryKeys = new[] { "*" })]
public async Task<IActionResult> GetCategories()
{
    var categories = await _mediator.Send(new GetCategoriesQuery());
    return Ok(categories);
}
```

---

## Module 4: Advanced Communication, Messaging & Integration

### Chapter 17: High-Performance Service Communication with gRPC

```protobuf
// Protos/product.proto
syntax = "proto3";

option csharp_namespace = "ProductService.Grpc";

package product;

service ProductGrpcService {
    rpc GetProduct (GetProductRequest) returns (ProductResponse);
    rpc GetProducts (GetProductsRequest) returns (ProductListResponse);
    rpc CheckStock (CheckStockRequest) returns (CheckStockResponse);
    rpc CreateProduct (CreateProductRequest) returns (ProductResponse);
    rpc UpdateProduct (UpdateProductRequest) returns (ProductResponse);
    rpc DeleteProduct (DeleteProductRequest) returns (DeleteProductResponse);
    
    // Streaming
    rpc GetProductStream (GetProductsRequest) returns (stream ProductResponse);
    rpc BulkUpdateStock (stream UpdateStockRequest) returns (BulkUpdateResponse);
}

message GetProductRequest {
    string product_id = 1;
}

message GetProductsRequest {
    int32 page = 1;
    int32 page_size = 2;
    string category_id = 3;
}

message ProductResponse {
    string product_id = 1;
    string name = 2;
    string description = 3;
    string sku = 4;
    Money price = 5;
    int32 stock_quantity = 6;
    string category_id = 7;
    bool is_active = 8;
    double average_rating = 9;
    int32 review_count = 10;
}

message Money {
    decimal amount = 1;
    string currency = 2;
}

message CheckStockRequest {
    string product_id = 1;
    int32 requested_quantity = 2;
}

message CheckStockResponse {
    bool is_available = 1;
    int32 available_quantity = 2;
    string estimated_restock_date = 3;
}

message CreateProductRequest {
    string name = 1;
    string description = 2;
    string sku = 3;
    Money price = 4;
    int32 stock_quantity = 5;
    string category_id = 6;
}

message UpdateProductRequest {
    string product_id = 1;
    string name = 2;
    string description = 3;
    Money price = 4;
}

message UpdateStockRequest {
    string product_id = 1;
    int32 quantity_change = 2;
    string reason = 3;
}

message BulkUpdateResponse {
    bool success = 1;
    int32 updated_count = 2;
    repeated string errors = 3;
}

message DeleteProductRequest {
    string product_id = 1;
}

message DeleteProductResponse {
    bool success = 1;
    string message = 2;
}

message ProductListResponse {
    repeated ProductResponse products = 1;
    int32 total_count = 2;
    int32 page = 3;
    int32 page_size = 4;
}
```

```csharp
// gRPC Service Implementation
public class ProductGrpcService : ProductGrpcServiceBase
{
    private readonly IMediator _mediator;
    private readonly IMapper _mapper;
    private readonly ILogger<ProductGrpcService> _logger;
    
    public ProductGrpcService(IMediator mediator, IMapper mapper, ILogger<ProductGrpcService> logger)
    {
        _mediator = mediator;
        _mapper = mapper;
        _logger = logger;
    }
    
    public override async Task<ProductResponse> GetProduct(
        GetProductRequest request, ServerCallContext context)
    {
        _logger.LogInformation("gRPC GetProduct called for {ProductId}", request.ProductId);
        
        var product = await _mediator.Send(new GetProductByIdQuery(Guid.Parse(request.ProductId)));
        
        return _mapper.Map<ProductResponse>(product);
    }
    
    public override async Task<CheckStockResponse> CheckStock(
        CheckStockRequest request, ServerCallContext context)
    {
        var product = await _mediator.Send(new GetProductByIdQuery(Guid.Parse(request.ProductId)));
        
        return new CheckStockResponse
        {
            IsAvailable = product.StockQuantity >= request.RequestedQuantity,
            AvailableQuantity = product.StockQuantity
        };
    }
    
    public override async Task GetProductStream(
        GetProductsRequest request, 
        IServerStreamWriter<ProductResponse> responseStream, 
        ServerCallContext context)
    {
        var products = await _mediator.Send(new SearchProductsQuery
        {
            Page = request.Page,
            PageSize = request.PageSize
        });
        
        foreach (var product in products.Items)
        {
            if (context.CancellationToken.IsCancellationRequested)
                break;
                
            await responseStream.WriteAsync(_mapper.Map<ProductResponse>(product));
        }
    }
    
    public override async Task<BulkUpdateResponse> BulkUpdateStock(
        IAsyncStreamReader<UpdateStockRequest> requestStream, 
        ServerCallContext context)
    {
        var response = new BulkUpdateResponse { Success = true };
        
        await foreach (var request in requestStream.ReadAllAsync())
        {
            try
            {
                await _mediator.Send(new UpdateStockCommand
                {
                    ProductId = Guid.Parse(request.ProductId),
                    QuantityChange = request.QuantityChange,
                    Reason = request.Reason
                });
                response.UpdatedCount++;
            }
            catch (Exception ex)
            {
                response.Errors.Add($"Failed to update {request.ProductId}: {ex.Message}");
            }
        }
        
        response.Success = !response.Errors.Any();
        return response;
    }
}

// gRPC Client
public class ProductGrpcClient : IProductServiceClient
{
    private readonly ProductGrpcService.ProductGrpcServiceClient _client;
    private readonly ILogger<ProductGrpcClient> _logger;
    
    public ProductGrpcClient(
        ProductGrpcService.ProductGrpcServiceClient client,
        ILogger<ProductGrpcClient> logger)
    {
        _client = client;
        _logger = logger;
    }
    
    public async Task<ProductDto> GetProductAsync(Guid productId)
    {
        var request = new GetProductRequest { ProductId = productId.ToString() };
        var response = await _client.GetProductAsync(request);
        
        return MapToDto(response);
    }
    
    public async Task<bool> CheckStockAsync(Guid productId, int quantity)
    {
        var request = new CheckStockRequest
        {
            ProductId = productId.ToString(),
            RequestedQuantity = quantity
        };
        
        var response = await _client.CheckStockAsync(request);
        return response.IsAvailable;
    }
    
    private ProductDto MapToDto(ProductResponse response)
    {
        return new ProductDto
        {
            Id = Guid.Parse(response.ProductId),
            Name = response.Name,
            Description = response.Description,
            SKU = response.Sku,
            Price = response.Price.Amount,
            StockQuantity = response.StockQuantity,
            CategoryId = Guid.Parse(response.CategoryId)
        };
    }
}

// Register gRPC in Program.cs
builder.Services.AddGrpc(options =>
{
    options.EnableDetailedErrors = true;
    options.MaxReceiveMessageSize = 2 * 1024 * 1024; // 2 MB
    options.MaxSendMessageSize = 5 * 1024 * 1024; // 5 MB
});

// Register gRPC client
builder.Services.AddGrpcClient<ProductGrpcService.ProductGrpcServiceClient>(options =>
{
    options.Address = new Uri(builder.Configuration["GrpcServices:ProductService"]);
})
.ConfigurePrimaryHttpMessageHandler(() =>
{
    var handler = new HttpClientHandler();
    handler.ServerCertificateCustomValidationCallback = 
        HttpClientHandler.DangerousAcceptAnyServerCertificateValidator;
    return handler;
});

// Map gRPC endpoints
app.MapGrpcService<ProductGrpcService>();
```

---

### Chapter 18: Building Event-Driven Microservices Architecture

```csharp
// Event Bus Abstraction
public interface IEventBus
{
    Task PublishAsync<T>(T @event, CancellationToken cancellationToken = default) where T : IntegrationEvent;
    Task SubscribeAsync<T, TH>(CancellationToken cancellationToken = default)
        where T : IntegrationEvent
        where TH : IIntegrationEventHandler<T>;
    Task UnsubscribeAsync<T, TH>(CancellationToken cancellationToken = default)
        where T : IntegrationEvent
        where TH : IIntegrationEventHandler<T>;
}

// Integration Event Base
public abstract record IntegrationEvent
{
    public Guid EventId { get; init; } = Guid.NewGuid();
    public DateTime OccurredOn { get; init; } = DateTime.UtcNow;
    public string EventType => GetType().Name;
}

// Integration Event Handler
public interface IIntegrationEventHandler<in T> where T : IntegrationEvent
{
    Task Handle(T @event);
}

// Event Bus with RabbitMQ
public class RabbitMQEventBus : IEventBus, IDisposable
{
    private readonly IRabbitMQPersistentConnection _persistentConnection;
    private readonly ILogger<RabbitMQEventBus> _logger;
    private readonly IServiceProvider _serviceProvider;
    private readonly IEventBusSubscriptionsManager _subscriptionsManager;
    private readonly string _exchangeName;
    private readonly string _queueName;
    
    private IModel _consumerChannel;
    
    public RabbitMQEventBus(
        IRabbitMQPersistentConnection persistentConnection,
        ILogger<RabbitMQEventBus> logger,
        IServiceProvider serviceProvider,
        IEventBusSubscriptionsManager subscriptionsManager,
        string exchangeName,
        string queueName)
    {
        _persistentConnection = persistentConnection;
        _logger = logger;
        _serviceProvider = serviceProvider;
        _subscriptionsManager = subscriptionsManager;
        _exchangeName = exchangeName;
        _queueName = queueName;
        _consumerChannel = CreateConsumerChannel();
    }
    
    public async Task PublishAsync<T>(T @event, CancellationToken cancellationToken = default) 
        where T : IntegrationEvent
    {
        if (!_persistentConnection.IsConnected)
        {
            _persistentConnection.TryConnect();
        }
        
        var policy = Policy
            .Handle<BrokerUnreachableException>()
            .Or<SocketException>()
            .WaitAndRetry(3, retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)));
            
        var eventName = @event.GetType().Name;
        
        using var channel = _persistentConnection.CreateModel();
        
        channel.ExchangeDeclare(_exchangeName, ExchangeType.Direct, durable: true);
        
        var message = JsonSerializer.Serialize(@event);
        var body = Encoding.UTF8.GetBytes(message);
        
        policy.Execute(() =>
        {
            var properties = channel.CreateBasicProperties();
            properties.DeliveryMode = 2; // persistent
            properties.MessageId = @event.EventId.ToString();
            properties.Type = eventName;
            properties.Timestamp = new AmqpTimestamp(DateTimeOffset.UtcNow.ToUnixTimeSeconds());
            
            channel.BasicPublish(
                exchange: _exchangeName,
                routingKey: eventName,
                mandatory: true,
                basicProperties: properties,
                body: body);
        });
        
        _logger.LogInformation("Published event {EventName} with ID {EventId}", eventName, @event.EventId);
    }
    
    public async Task SubscribeAsync<T, TH>(CancellationToken cancellationToken = default)
        where T : IntegrationEvent
        where TH : IIntegrationEventHandler<T>
    {
        var eventName = _subscriptionsManager.GetEventKey<T>();
        
        DoInternalSubscription(eventName);
        
        _subscriptionsManager.AddSubscription<T, TH>();
        
        StartBasicConsume();
    }
    
    private void DoInternalSubscription(string eventName)
    {
        var containsKey = _subscriptionsManager.HasSubscriptionsForEvent(eventName);
        if (!containsKey)
        {
            if (!_persistentConnection.IsConnected)
            {
                _persistentConnection.TryConnect();
            }
            
            _consumerChannel.QueueBind(
                queue: _queueName,
                exchange: _exchangeName,
                routingKey: eventName);
        }
    }
    
    private IModel CreateConsumerChannel()
    {
        if (!_persistentConnection.IsConnected)
        {
            _persistentConnection.TryConnect();
        }
        
        var channel = _persistentConnection.CreateModel();
        
        channel.ExchangeDeclare(_exchangeName, ExchangeType.Direct, durable: true);
        channel.QueueDeclare(_queueName, durable: true, exclusive: false, autoDelete: false);
        
        channel.CallbackException += (sender, ea) =>
        {
            _consumerChannel.Dispose();
            _consumerChannel = CreateConsumerChannel();
            StartBasicConsume();
        };
        
        return channel;
    }
    
    private void StartBasicConsume()
    {
        if (_consumerChannel == null)
        {
            _logger.LogError("StartBasicConsume can't call on _consumerChannel == null");
            return;
        }
        
        var consumer = new AsyncEventingBasicConsumer(_consumerChannel);
        
        consumer.Received += async (model, ea) =>
        {
            var eventName = ea.RoutingKey;
            var message = Encoding.UTF8.GetString(ea.Body.Span);
            
            try
            {
                await ProcessEvent(eventName, message);
                _consumerChannel.BasicAck(ea.DeliveryTag, multiple: false);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error processing event {EventName}", eventName);
                _consumerChannel.BasicNack(ea.DeliveryTag, multiple: false, requeue: true);
            }
        };
        
        _consumerChannel.BasicConsume(
            queue: _queueName,
            autoAck: false,
            consumer: consumer);
    }
    
    private async Task ProcessEvent(string eventName, string message)
    {
        if (_subscriptionsManager.HasSubscriptionsForEvent(eventName))
        {
            using var scope = _serviceProvider.CreateScope();
            
            var subscriptions = _subscriptionsManager.GetHandlersForEvent(eventName);
            foreach (var subscription in subscriptions)
            {
                var handler = scope.ServiceProvider.GetService(subscription.HandlerType);
                if (handler == null) continue;
                
                var eventType = _subscriptionsManager.GetEventTypeByName(eventName);
                var integrationEvent = JsonSerializer.Deserialize(message, eventType);
                var concreteType = typeof(IIntegrationEventHandler<>).MakeGenericType(eventType);
                
                await (Task)concreteType.GetMethod("Handle")
                    .Invoke(handler, new[] { integrationEvent });
            }
        }
    }
    
    public void Dispose()
    {
        _consumerChannel?.Dispose();
    }
}

// Register Event Bus
builder.Services.AddSingleton<IEventBusSubscriptionsManager, InMemoryEventBusSubscriptionsManager>();
builder.Services.AddSingleton<IEventBus, RabbitMQEventBus>(sp =>
{
    var connection = sp.GetRequiredService<IRabbitMQPersistentConnection>();
    var logger = sp.GetRequiredService<ILogger<RabbitMQEventBus>>();
    var serviceProvider = sp;
    var subsManager = sp.GetRequiredService<IEventBusSubscriptionsManager>();
    
    return new RabbitMQEventBus(
        connection,
        logger,
        serviceProvider,
        subsManager,
        "ecommerce_exchange",
        "order_service_queue"
    );
});

// Subscribe to events
var eventBus = app.Services.GetRequiredService<IEventBus>();

eventBus.SubscribeAsync<OrderPlacedIntegrationEvent, OrderPlacedEventHandler>();
eventBus.SubscribeAsync<PaymentCompletedIntegrationEvent, PaymentCompletedEventHandler>();
```

---

### Chapter 19: Apache Kafka for Microservices

```csharp
// Kafka Producer Service
public class KafkaProducerService : IKafkaProducerService
{
    private readonly IProducer<string, string> _producer;
    private readonly ILogger<KafkaProducerService> _logger;
    
    public KafkaProducerService(IOptions<KafkaSettings> settings, ILogger<KafkaProducerService> logger)
    {
        _logger = logger;
        
        var config = new ProducerConfig
        {
            BootstrapServers = settings.Value.BootstrapServers,
            Acks = Acks.All,
            EnableIdempotence = true,
            MessageSendMaxRetries = 3,
            RetryBackoffMs = 1000,
            CompressionType = CompressionType.Snappy,
            LingerMs = 5,
            BatchSize = 32768
        };
        
        _producer = new ProducerBuilder<string, string>(config).Build();
    }
    
    public async Task ProduceAsync<T>(string topic, string key, T message, 
        CancellationToken cancellationToken = default)
    {
        var json = JsonSerializer.Serialize(message);
        
        var kafkaMessage = new Message<string, string>
        {
            Key = key,
            Value = json,
            Headers = new Headers
            {
                { "event-type", Encoding.UTF8.GetBytes(typeof(T).Name) },
                { "timestamp", Encoding.UTF8.GetBytes(DateTime.UtcNow.ToString("O")) }
            }
        };
        
        try
        {
            var result = await _producer.ProduceAsync(topic, kafkaMessage, cancellationToken);
            
            _logger.LogInformation(
                "Message delivered to {Topic} [{Partition}] @ offset {Offset}",
                result.Topic, result.Partition, result.Offset);
        }
        catch (ProduceException<string, string> ex)
        {
            _logger.LogError(ex, "Failed to deliver message to {Topic}: {Error}", 
                topic, ex.Error.Reason);
            throw;
        }
    }
}

// Kafka Consumer Service
public class KafkaConsumerService : BackgroundService
{
    private readonly IConsumer<string, string> _consumer;
    private readonly IServiceProvider _serviceProvider;
    private readonly ILogger<KafkaConsumerService> _logger;
    private readonly Dictionary<string, Type> _eventTypes;
    
    public KafkaConsumerService(
        IOptions<KafkaSettings> settings,
        IServiceProvider serviceProvider,
        ILogger<KafkaConsumerService> logger)
    {
        _serviceProvider = serviceProvider;
        _logger = logger;
        
        var config = new ConsumerConfig
        {
            BootstrapServers = settings.Value.BootstrapServers,
            GroupId = settings.Value.ConsumerGroup,
            AutoOffsetReset = AutoOffsetReset.Earliest,
            EnableAutoCommit = false,
            MaxPollIntervalMs = 300000,
            SessionTimeoutMs = 30000
        };
        
        _consumer = new ConsumerBuilder<string, string>(config).Build();
        
        _eventTypes = Assembly.GetExecutingAssembly()
            .GetTypes()
            .Where(t => t.IsSubclassOf(typeof(IntegrationEvent)))
            .ToDictionary(t => t.Name, t => t);
    }
    
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        var topics = new[] { "order-events", "payment-events", "inventory-events" };
        _consumer.Subscribe(topics);
        
        _logger.LogInformation("Kafka consumer started, listening to topics: {Topics}", 
            string.Join(", ", topics));
        
        while (!stoppingToken.IsCancellationRequested)
        {
            try
            {
                var consumeResult = _consumer.Consume(stoppingToken);
                
                await ProcessMessageAsync(consumeResult, stoppingToken);
                
                _consumer.Commit(consumeResult);
            }
            catch (ConsumeException ex)
            {
                _logger.LogError(ex, "Error consuming message: {Error}", ex.Error.Reason);
            }
            catch (OperationCanceledException)
            {
                break;
            }
        }
        
        _consumer.Close();
    }
    
    private async Task ProcessMessageAsync(ConsumeResult<string, string> result, 
        CancellationToken cancellationToken)
    {
        var eventTypeHeader = result.Message.Headers
            .FirstOrDefault(h => h.Key == "event-type");
            
        if (eventTypeHeader == null)
        {
            _logger.LogWarning("No event-type header found for message at offset {Offset}", 
                result.Offset);
            return;
        }
        
        var eventTypeName = Encoding.UTF8.GetString(eventTypeHeader.GetValueBytes());
        
        if (!_eventTypes.TryGetValue(eventTypeName, out var eventType))
        {
            _logger.LogWarning("Unknown event type: {EventType}", eventTypeName);
            return;
        }
        
        var @event = JsonSerializer.Deserialize(result.Message.Value, eventType);
        
        using var scope = _serviceProvider.CreateScope();
        
        var handlerType = typeof(IIntegrationEventHandler<>).MakeGenericType(eventType);
        var handlers = scope.ServiceProvider.GetServices(handlerType);
        
        foreach (dynamic handler in handlers)
        {
            await handler.Handle((dynamic)@event);
        }
    }
}

// Kafka Settings
public class KafkaSettings
{
    public string BootstrapServers { get; set; }
    public string ConsumerGroup { get; set; }
    public Dictionary<string, TopicConfig> Topics { get; set; }
}

public class TopicConfig
{
    public string Name { get; set; }
    public int Partitions { get; set; } = 3;
    public short ReplicationFactor { get; set; } = 1;
}

// Register Kafka services
builder.Services.Configure<KafkaSettings>(builder.Configuration.GetSection("Kafka"));
builder.Services.AddSingleton<IKafkaProducerService, KafkaProducerService>();
builder.Services.AddHostedService<KafkaConsumerService>();
```

---

### Chapter 23: Failure Handling in Microservices

```csharp
// Comprehensive Resilience Policies with Polly

public static class ResiliencePolicies
{
    // Retry Policy
    public static IAsyncPolicy GetRetryPolicy(ILogger logger)
    {
        return Policy
            .Handle<HttpRequestException>()
            .Or<TimeoutRejectedException>()
            .Or<BrokenCircuitException>()
            .OrResult<HttpResponseMessage>(r => 
                r.StatusCode >= HttpStatusCode.InternalServerError || 
                r.StatusCode == HttpStatusCode.RequestTimeout)
            .WaitAndRetryAsync(
                retryCount: 3,
                sleepDurationProvider: retryAttempt => 
                    TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)) + 
                    TimeSpan.FromMilliseconds(new Random().Next(0, 1000)),
                onRetry: (outcome, timespan, retryAttempt, context) =>
                {
                    logger.LogWarning(
                        "Retry {RetryAttempt} after {Delay}ms due to {StatusCode}",
                        retryAttempt,
                        timespan.TotalMilliseconds,
                        outcome.Result?.StatusCode);
                });
    }
    
    // Circuit Breaker Policy
    public static IAsyncPolicy GetCircuitBreakerPolicy(ILogger logger)
    {
        return Policy
            .Handle<HttpRequestException>()
            .Or<TimeoutRejectedException>()
            .CircuitBreakerAsync(
                exceptionsAllowedBeforeBreaking: 5,
                durationOfBreak: TimeSpan.FromSeconds(30),
                onBreak: (exception, timespan) =>
                {
                    logger.LogWarning(
                        "Circuit broken for {Duration}s due to {Exception}",
                        timespan.TotalSeconds,
                        exception.Message);
                },
                onReset: () =>
                {
                    logger.LogInformation("Circuit reset");
                },
                onHalfOpen: () =>
                {
                    logger.LogInformation("Circuit half-open, testing...");
                });
    }
    
    // Timeout Policy
    public static IAsyncPolicy GetTimeoutPolicy(int seconds = 10)
    {
        return Policy.TimeoutAsync(
            seconds: seconds,
            timeoutStrategy: TimeoutStrategy.Optimistic);
    }
    
    // Bulkhead Policy
    public static IAsyncPolicy GetBulkheadPolicy(int maxParallelization = 10, int maxQueuingActions = 20)
    {
        return Policy.BulkheadAsync(
            maxParallelization: maxParallelization,
            maxQueuingActions: maxQueuingActions);
    }
    
    // Fallback Policy
    public static IAsyncPolicy<T> GetFallbackPolicy<T>(Func<T> fallbackAction, ILogger logger)
    {
        return Policy<T>
            .Handle<Exception>()
            .FallbackAsync(
                fallbackAction: cancellationToken => Task.FromResult(fallbackAction()),
                onFallbackAsync: (result, context) =>
                {
                    logger.LogWarning("Fallback executed due to {Exception}", 
                        result.Exception?.Message);
                    return Task.CompletedTask;
                });
    }
    
    // Combined Policy
    public static IAsyncPolicy<HttpResponseMessage> GetCombinedPolicy(ILogger logger)
    {
        var retry = GetRetryPolicy(logger);
        var circuitBreaker = GetCircuitBreakerPolicy(logger);
        var timeout = GetTimeoutPolicy(5);
        var bulkhead = GetBulkheadPolicy(5, 10);
        
        return Policy.WrapAsync(
            retry,
            circuitBreaker,
            timeout,
            bulkhead);
    }
}

// Resilient HttpClient Factory
public static class ResilientHttpClientExtensions
{
    public static IHttpClientBuilder AddResiliencePolicy(this IHttpClientBuilder builder, ILogger logger)
    {
        return builder
            .AddPolicyHandler(ResiliencePolicies.GetRetryPolicy(logger))
            .AddPolicyHandler(ResiliencePolicies.GetCircuitBreakerPolicy(logger))
            .AddPolicyHandler(ResiliencePolicies.GetTimeoutPolicy(10));
    }
}

// Register with DI
builder.Services.AddHttpClient("ResilientClient")
    .AddResiliencePolicy(sp.GetRequiredService<ILogger<Program>>());

// Resilient Service Call Example
public class ResilientOrderService
{
    private readonly IHttpClientFactory _httpClientFactory;
    private readonly AsyncPolicy _resiliencePolicy;
    
    public async Task<OrderDto> GetOrderAsync(Guid orderId)
    {
        var client = _httpClientFactory.CreateClient("OrderService");
        
        return await _resiliencePolicy.ExecuteAsync(async () =>
        {
            var response = await client.GetAsync($"api/orders/{orderId}");
            response.EnsureSuccessStatusCode();
            return await response.Content.ReadFromJsonAsync<OrderDto>();
        });
    }
}

// Health Checks for Resilience
builder.Services.AddHealthChecks()
    .AddCheck<OrderServiceHealthCheck>("order_service")
    .AddCheck<PaymentServiceHealthCheck>("payment_service")
    .AddSqlServer(
        connectionString: builder.Configuration.GetConnectionString("DefaultConnection"),
        name: "sql_server")
    .AddRedis(
        redisConnectionString: builder.Configuration.GetConnectionString("Redis"),
        name: "redis")
    .AddRabbitMQ(
        rabbitConnectionString: builder.Configuration.GetConnectionString("RabbitMQ"),
        name: "rabbitmq");

public class OrderServiceHealthCheck : IHealthCheck
{
    private readonly IOrderRepository _repository;
    
    public async Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context, 
        CancellationToken cancellationToken = default)
    {
        try
        {
            // Try to query a single order to verify database connectivity
            await _repository.GetByIdAsync(Guid.Empty);
            return HealthCheckResult.Healthy("Order service is healthy");
        }
        catch (Exception ex)
        {
            return HealthCheckResult.Unhealthy("Order service is unhealthy", ex);
        }
    }
}

app.MapHealthChecks("/health", new HealthCheckOptions
{
    ResponseWriter = UIResponseWriter.WriteHealthCheckUIResponse
});
```

---

## Module 6: Containerization & Load Balancing

### Chapter 26: Docker for Microservices

```dockerfile
# Dockerfile for OrderService.API
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

# Copy csproj and restore dependencies
COPY ["src/Services/OrderService/OrderService.API/OrderService.API.csproj", "Services/OrderService/OrderService.API/"]
COPY ["src/Services/OrderService/OrderService.Application/OrderService.Application.csproj", "Services/OrderService/OrderService.Application/"]
COPY ["src/Services/OrderService/OrderService.Domain/OrderService.Domain.csproj", "Services/OrderService/OrderService.Domain/"]
COPY ["src/Services/OrderService/OrderService.Infrastructure/OrderService.Infrastructure.csproj", "Services/OrderService/OrderService.Infrastructure/"]
COPY ["src/BuildingBlocks/Common.Domain/Common.Domain.csproj", "BuildingBlocks/Common.Domain/"]
COPY ["src/BuildingBlocks/Common.Application/Common.Application.csproj", "BuildingBlocks/Common.Application/"]
COPY ["src/BuildingBlocks/Common.Infrastructure/Common.Infrastructure.csproj", "BuildingBlocks/Common.Infrastructure/"]

RUN dotnet restore "Services/OrderService/OrderService.API/OrderService.API.csproj"

# Copy everything else and build
COPY . .
WORKDIR "/src/src/Services/OrderService/OrderService.API"
RUN dotnet build "OrderService.API.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "OrderService.API.csproj" -c Release -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "OrderService.API.dll"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  # Databases
  sqlserver:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=YourStrong!Passw0rd
      - MSSQL_PID=Express
    ports:
      - "1433:1433"
    volumes:
      - sqlserver_data:/var/opt/mssql
    networks:
      - ecommerce-network

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - ecommerce-network

  rabbitmq:
    image: rabbitmq:3-management-alpine
    environment:
      - RABBITMQ_DEFAULT_USER=admin
      - RABBITMQ_DEFAULT_PASS=admin123
    ports:
      - "5672:5672"   # AMQP
      - "15672:15672" # Management UI
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
    networks:
      - ecommerce-network
  
  # Microservices
  user-service:
    build:
      context: .
      dockerfile: src/Services/UserService/UserService.API/Dockerfile
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ConnectionStrings__DefaultConnection=Server=sqlserver;Database=UserServiceDb;User=sa;Password=YourStrong!Passw0rd;TrustServerCertificate=true
      - Jwt__Secret=YourSuperSecretKeyThatIsAtLeast32BytesLong!
      - Jwt__Issuer=ecommerce
      - Jwt__Audience=ecommerce-clients
      - Redis__ConnectionString=redis:6379
    ports:
      - "5001:80"
    depends_on:
      - sqlserver
      - redis
    networks:
      - ecommerce-network

  product-service:
    build:
      context: .
      dockerfile: src/Services/ProductService/ProductService.API/Dockerfile
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ConnectionStrings__DefaultConnection=Server=sqlserver;Database=ProductServiceDb;User=sa;Password=YourStrong!Passw0rd;TrustServerCertificate=true
      - Redis__ConnectionString=redis:6379
    ports:
      - "5002:80"
    depends_on:
      - sqlserver
      - redis
    networks:
      - ecommerce-network

  order-service:
    build:
      context: .
      dockerfile: src/Services/OrderService/OrderService.API/Dockerfile
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ConnectionStrings__DefaultConnection=Server=sqlserver;Database=OrderServiceDb;User=sa;Password=YourStrong!Passw0rd;TrustServerCertificate=true
      - ServiceUrls__UserService=http://user-service
      - ServiceUrls__ProductService=http://product-service
      - RabbitMQ__ConnectionString=amqp://admin:admin123@rabbitmq:5672
      - Redis__ConnectionString=redis:6379
    ports:
      - "5003:80"
    depends_on:
      - sqlserver
      - rabbitmq
      - redis
      - user-service
      - product-service
    networks:
      - ecommerce-network

  payment-service:
    build:
      context: .
      dockerfile: src/Services/PaymentService/PaymentService.API/Dockerfile
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ConnectionStrings__DefaultConnection=Server=sqlserver;Database=PaymentServiceDb;User=sa;Password=YourStrong!Passw0rd;TrustServerCertificate=true
      - RabbitMQ__ConnectionString=amqp://admin:admin123@rabbitmq:5672
    ports:
      - "5004:80"
    depends_on:
      - sqlserver
      - rabbitmq
    networks:
      - ecommerce-network

  notification-service:
    build:
      context: .
      dockerfile: src/Services/NotificationService/NotificationService.API/Dockerfile
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ConnectionStrings__DefaultConnection=Server=sqlserver;Database=NotificationServiceDb;User=sa;Password=YourStrong!Passw0rd;TrustServerCertificate=true
      - RabbitMQ__ConnectionString=amqp://admin:admin123@rabbitmq:5672
    ports:
      - "5005:80"
    depends_on:
      - sqlserver
      - rabbitmq
    networks:
      - ecommerce-network

  api-gateway:
    build:
      context: .
      dockerfile: src/ApiGateways/OcelotApiGateway/Dockerfile
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
    ports:
      - "5000:80"
    depends_on:
      - user-service
      - product-service
      - order-service
      - payment-service
    networks:
      - ecommerce-network

volumes:
  sqlserver_data:
  redis_data:
  rabbitmq_data:

networks:
  ecommerce-network:
    driver: bridge
```

---

### Chapter 27: Kubernetes (K8s) for Microservices

```yaml
# kubernetes/order-service-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
  namespace: ecommerce
  labels:
    app: order-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: order-service
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: order-service
    spec:
      containers:
      - name: order-service
        image: ecommerce/order-service:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 80
          name: http
        env:
        - name: ASPNETCORE_ENVIRONMENT
          value: "Production"
        - name: ConnectionStrings__DefaultConnection
          valueFrom:
            secretKeyRef:
              name: order-service-secrets
              key: connection-string
        - name: ServiceUrls__UserService
          value: "http://user-service"
        - name: ServiceUrls__ProductService
          value: "http://product-service"
        - name: RabbitMQ__ConnectionString
          valueFrom:
            secretKeyRef:
              name: rabbitmq-secrets
              key: connection-string
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 2
---
apiVersion: v1
kind: Service
metadata:
  name: order-service
  namespace: ecommerce
  labels:
    app: order-service
spec:
  type: ClusterIP
  selector:
    app: order-service
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: order-service-hpa
  namespace: ecommerce
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: order-service
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

```yaml
# kubernetes/secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: order-service-secrets
  namespace: ecommerce
type: Opaque
data:
  connection-string: U2VydmVyPXNxbHNlcnZlcjtEYXRhYmFzZT1PcmRlclNlcnZpY2VEYjtVc2VyPXNhO1Bhc3N3b3JkPVlvdXJTdHJvbmchUGFzc3cwcmQ7VHJ1c3RTZXJ2ZXJDZXJ0aWZpY2F0ZT10cnVl
  jwt-secret: WW91clN1cGVyU2VjcmV0S2V5VGhhdElzQXRMZWFzdDMyQnl0ZXNMb25nIQ==
---
apiVersion: v1
kind: Secret
metadata:
  name: rabbitmq-secrets
  namespace: ecommerce
type: Opaque
data:
  connection-string: YW1xcDovL2FkbWluOmFkbWluMTIzQHJhYmJpdG1xOjU2NzI=
```

```yaml
# kubernetes/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ecommerce-ingress
  namespace: ecommerce
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/cors-allow-origin: "*"
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/rate-limit-period: "1m"
spec:
  tls:
  - hosts:
    - api.ecommerce.com
    secretName: ecommerce-tls
  rules:
  - host: api.ecommerce.com
    http:
      paths:
      - path: /api/users
        pathType: Prefix
        backend:
          service:
            name: user-service
            port:
              number: 80
      - path: /api/products
        pathType: Prefix
        backend:
          service:
            name: product-service
            port:
              number: 80
      - path: /api/orders
        pathType: Prefix
        backend:
          service:
            name: order-service
            port:
              number: 80
      - path: /api/payments
        pathType: Prefix
        backend:
          service:
            name: payment-service
            port:
              number: 80
```

---

### Chapter 30: CI/CD Pipelines for Microservices

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: '8.0.x'
    
    - name: Restore dependencies
      run: dotnet restore
    
    - name: Build
      run: dotnet build --configuration Release --no-restore
    
    - name: Run unit tests
      run: dotnet test --configuration Release --no-build --filter "Category=Unit" --collect:"XPlat Code Coverage"
    
    - name: Run integration tests
      run: dotnet test --configuration Release --no-build --filter "Category=Integration"
    
    - name: Run code analysis
      run: |
        dotnet tool install -g dotnet-sonarscanner
        dotnet sonarscanner begin /k:"ecommerce-microservices" /o:"your-org" /d:sonar.login=${{ secrets.SONAR_TOKEN }}
        dotnet build
        dotnet sonarscanner end /d:sonar.login=${{ secrets.SONAR_TOKEN }}
    
    - name: Upload coverage
      uses: actions/upload-artifact@v4
      with:
        name: coverage
        path: '**/coverage.cobertura.xml'

  docker-build-and-push:
    needs: build-and-test
    runs-on: ubuntu-latest
    strategy:
      matrix:
        service: 
          - UserService
          - ProductService
          - OrderService
          - PaymentService
          - NotificationService
          - ApiGateway
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3
    
    - name: Login to Container Registry
      uses: docker/login-action@v3
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    
    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/${{ matrix.service }}
        tags: |
          type=sha,format=short
          type=ref,event=branch
          type=raw,value=latest,enable=${{ github.ref == 'refs/heads/main' }}
    
    - name: Build and push
      uses: docker/build-push-action@v5
      with:
        context: .
        file: src/Services/${{ matrix.service }}/${{ matrix.service }}.API/Dockerfile
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
        build-args: |
          BUILD_CONFIGURATION=Release

  deploy-to-staging:
    needs: docker-build-and-push
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    environment: staging
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Set up kubectl
      uses: azure/setup-kubectl@v3
      with:
        version: 'latest'
    
    - name: Configure kubectl
      run: |
        echo "${{ secrets.KUBE_CONFIG_STAGING }}" > kubeconfig.yaml
        export KUBECONFIG=kubeconfig.yaml
    
    - name: Deploy to staging
      run: |
        kubectl apply -f kubernetes/namespace.yaml
        kubectl apply -f kubernetes/secrets.yaml
        kubectl apply -f kubernetes/ -n ecommerce-staging
        
    - name: Verify deployment
      run: |
        kubectl rollout status deployment/user-service -n ecommerce-staging --timeout=5m
        kubectl rollout status deployment/product-service -n ecommerce-staging --timeout=5m
        kubectl rollout status deployment/order-service -n ecommerce-staging --timeout=5m

  deploy-to-production:
    needs: docker-build-and-push
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Set up kubectl
      uses: azure/setup-kubectl@v3
    
    - name: Configure kubectl
      run: |
        echo "${{ secrets.KUBE_CONFIG_PRODUCTION }}" > kubeconfig.yaml
        export KUBECONFIG=kubeconfig.yaml
    
    - name: Deploy to production (Canary)
      run: |
        # Deploy canary version
        kubectl set image deployment/order-service \
          order-service=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/OrderService:sha-${GITHUB_SHA::7} \
          -n ecommerce-production
        
        # Wait for canary to be ready
        kubectl rollout status deployment/order-service -n ecommerce-production --timeout=5m
        
        # Verify canary health
        ./scripts/health-check.sh
        
    - name: Rollback on failure
      if: failure()
      run: |
        kubectl rollout undo deployment/order-service -n ecommerce-production
```

---

### Chapter 31: Testing Strategies for Microservices

```csharp
// ========== Unit Tests ==========

// Unit test for domain entity
public class OrderTests
{
    [Fact]
    public void Place_OrderWithItems_ShouldSucceed()
    {
        // Arrange
        var order = new Order(
            Guid.NewGuid(),
            new Address("123 Main St", "NYC", "NY", "10001", "US"),
            new Address("123 Main St", "NYC", "NY", "10001", "US")
        );
        
        order.AddItem(
            Guid.NewGuid(), 
            "Test Product", 
            new Money(10.99m, "USD"), 
            2
        );
        
        // Act
        order.Place();
        
        // Assert
        Assert.Equal(OrderStatus.Placed, order.Status);
        Assert.Single(order.DomainEvents.OfType<OrderPlacedEvent>());
    }
    
    [Fact]
    public void Place_EmptyOrder_ShouldThrowException()
    {
        // Arrange
        var order = new Order(
            Guid.NewGuid(),
            new Address("123 Main St", "NYC", "NY", "10001", "US"),
            new Address("123 Main St", "NYC", "NY", "10001", "US")
        );
        
        // Act & Assert
        Assert.Throws<DomainException>(() => order.Place());
    }
    
    [Fact]
    public void Cancel_ShippedOrder_ShouldThrowException()
    {
        // Arrange
        var order = CreateShippedOrder();
        
        // Act & Assert
        Assert.Throws<DomainException>(() => order.Cancel("Test reason"));
    }
    
    private Order CreateShippedOrder()
    {
        var order = new Order(
            Guid.NewGuid(),
            new Address("123 Main St", "NYC", "NY", "10001", "US"),
            new Address("123 Main St", "NYC", "NY", "10001", "US")
        );
        
        order.AddItem(Guid.NewGuid(), "Product", new Money(10m, "USD"), 1);
        order.Place();
        order.Confirm();
        order.MarkAsPaid(Guid.NewGuid());
        order.Ship("TRACK123", "FedEx");
        
        return order;
    }
}

// Unit test for command handler (with mocks)
public class PlaceOrderCommandHandlerTests
{
    private readonly Mock<IOrderRepository> _orderRepositoryMock;
    private readonly Mock<IUserServiceClient> _userClientMock;
    private readonly Mock<IProductServiceClient> _productClientMock;
    private readonly Mock<IMapper> _mapperMock;
    private readonly PlaceOrderCommandHandler _handler;
    
    public PlaceOrderCommandHandlerTests()
    {
        _orderRepositoryMock = new Mock<IOrderRepository>();
        _userClientMock = new Mock<IUserServiceClient>();
        _productClientMock = new Mock<IProductServiceClient>();
        _mapperMock = new Mock<IMapper>();
        
        _handler = new PlaceOrderCommandHandler(
            _orderRepositoryMock.Object,
            _userClientMock.Object,
            _productClientMock.Object,
            Mock.Of<ICouponServiceClient>(),
            _mapperMock.Object,
            Mock.Of<ILogger<PlaceOrderCommandHandler>>()
        );
    }
    
    [Fact]
    public async Task Handle_ValidRequest_ShouldCreateOrder()
    {
        // Arrange
        var command = new PlaceOrderCommand(
            Guid.NewGuid(),
            new AddressDto { Street = "123 Main", City = "NYC", State = "NY", ZipCode = "10001", Country = "US" },
            new AddressDto { Street = "123 Main", City = "NYC", State = "NY", ZipCode = "10001", Country = "US" },
            new List<OrderItemRequest>
            {
                new(Guid.NewGuid(), 2)
            }
        );
        
        var user = new UserDto { Id = command.UserId, Name = "Test User", Email = "test@test.com" };
        var product = new ProductDto 
        { 
            Id = command.Items[0].ProductId, 
            Name = "Test Product", 
            Price = 10m, 
            StockQuantity = 10,
            IsAvailable = true 
        };
        
        _userClientMock.Setup(x => x.GetUserAsync(command.UserId))
            .ReturnsAsync(user);
        _productClientMock.Setup(x => x.GetProductAsync(command.Items[0].ProductId))
            .ReturnsAsync(product);
        
        // Act
        var result = await _handler.Handle(command, CancellationToken.None);
        
        // Assert
        Assert.NotNull(result);
        _orderRepositoryMock.Verify(x => x.AddAsync(It.IsAny<Order>()), Times.Once);
    }
}

// ========== Integration Tests ==========

// Integration test with test database
public class OrderRepositoryIntegrationTests : IClassFixture<DatabaseFixture>
{
    private readonly DatabaseFixture _fixture;
    
    public OrderRepositoryIntegrationTests(DatabaseFixture fixture)
    {
        _fixture = fixture;
    }
    
    [Fact]
    public async Task AddAsync_ValidOrder_ShouldPersistToDatabase()
    {
        // Arrange
        using var context = _fixture.CreateContext();
        var repository = new OrderRepository(context, Mock.Of<IMediator>());
        
        var order = new Order(
            Guid.NewGuid(),
            new Address("123 Main", "NYC", "NY", "10001", "US"),
            new Address("123 Main", "NYC", "NY", "10001", "US")
        );
        order.AddItem(Guid.NewGuid(), "Product", new Money(10m, "USD"), 1);
        
        // Act
        await repository.AddAsync(order);
        
        // Assert
        var savedOrder = await repository.GetByIdAsync(order.Id);
        Assert.NotNull(savedOrder);
        Assert.Equal(order.OrderNumber, savedOrder.OrderNumber);
    }
}

// Database fixture for integration tests
public class DatabaseFixture : IDisposable
{
    private const string ConnectionString = "Server=localhost;Database=OrderServiceTest;User=sa;Password=YourStrong!Passw0rd;TrustServerCertificate=true";
    
    public DatabaseFixture()
    {
        using var context = CreateContext();
        context.Database.EnsureDeleted();
        context.Database.EnsureCreated();
    }
    
    public OrderDbContext CreateContext()
    {
        var options = new DbContextOptionsBuilder<OrderDbContext>()
            .UseSqlServer(ConnectionString)
            .Options;
            
        return new OrderDbContext(options);
    }
    
    public void Dispose()
    {
        using var context = CreateContext();
        context.Database.EnsureDeleted();
    }
}

// ========== End-to-End Tests ==========

// E2E test with WebApplicationFactory
public class OrderApiE2ETests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly WebApplicationFactory<Program> _factory;
    private readonly HttpClient _client;
    
    public OrderApiE2ETests(WebApplicationFactory<Program> factory)
    {
        _factory = factory.WithWebHostBuilder(builder =>
        {
            builder.UseEnvironment("Testing");
            builder.ConfigureServices(services =>
            {
                // Replace database with in-memory for testing
                var descriptor = services.SingleOrDefault(
                    d => d.ServiceType == typeof(DbContextOptions<OrderDbContext>));
                    
                if (descriptor != null)
                    services.Remove(descriptor);
                    
                services.AddDbContext<OrderDbContext>(options =>
                    options.UseInMemoryDatabase("TestDb"));
                    
                // Replace external services with mocks
                services.AddScoped<IUserServiceClient>(_ => 
                    Mock.Of<IUserServiceClient>(x => 
                        x.GetUserAsync(It.IsAny<Guid>()) == 
                        Task.FromResult(new UserDto 
                        { 
                            Id = Guid.NewGuid(), 
                            Name = "Test User", 
                            Email = "test@test.com" 
                        })));
            });
        });
        
        _client = _factory.CreateClient();
    }
    
    [Fact]
    public async Task PlaceOrder_ValidRequest_ShouldReturnCreated()
    {
        // Arrange
        var request = new
        {
            shippingAddress = new
            {
                street = "123 Main St",
                city = "New York",
                state = "NY",
                zipCode = "10001",
                country = "US"
            },
            billingAddress = new
            {
                street = "123 Main St",
                city = "New York",
                state = "NY",
                zipCode = "10001",
                country = "US"
            },
            items = new[]
            {
                new { productId = Guid.NewGuid(), quantity = 2 }
            }
        };
        
        var content = new StringContent(
            JsonSerializer.Serialize(request),
            Encoding.UTF8,
            "application/json");
            
        // Add auth token
        _client.DefaultRequestHeaders.Authorization = 
            new AuthenticationHeaderValue("Bearer", GetTestToken());
        
        // Act
        var response = await _client.PostAsync("/api/orders", content);
        
        // Assert
        Assert.Equal(HttpStatusCode.Created, response.StatusCode);
        
        var order = await response.Content.ReadFromJsonAsync<OrderDto>();
        Assert.NotNull(order);
        Assert.Equal("Placed", order.Status);
    }
    
    private string GetTestToken()
    {
        // Generate test JWT token
        var claims = new[]
        {
            new Claim(ClaimTypes.NameIdentifier, Guid.NewGuid().ToString()),
            new Claim(ClaimTypes.Email, "test@test.com"),
            new Claim(ClaimTypes.Role, "Customer")
        };
        
        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("TestSecretKeyForTestingPurposes123!"));
        var credentials = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);
        
        var token = new JwtSecurityToken(
            issuer: "test",
            audience: "test",
            claims: claims,
            expires: DateTime.UtcNow.AddHours(1),
            signingCredentials: credentials
        );
        
        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}

// ========== Performance Tests ==========

// Load test with NBomber
public class OrderServiceLoadTests
{
    [Fact]
    public void PlaceOrder_ShouldHandle1000RequestsPerSecond()
    {
        // Arrange
        using var httpClient = new HttpClient();
        httpClient.BaseAddress = new Uri("http://localhost:5003");
        
        var scenario = Scenario.Create("place_order", async context =>
        {
            var request = new HttpRequestMessage(HttpMethod.Post, "/api/orders")
            {
                Content = new StringContent(
                    JsonSerializer.Serialize(new
                    {
                        shippingAddress = new { street = "123 Main", city = "NYC", state = "NY", zipCode = "10001", country = "US" },
                        billingAddress = new { street = "123 Main", city = "NYC", state = "NY", zipCode = "10001", country = "US" },
                        items = new[] { new { productId = Guid.NewGuid(), quantity = 1 } }
                    }),
                    Encoding.UTF8,
                    "application/json")
            };
            
            request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", "test-token");
            
            var response = await httpClient.SendAsync(request);
            
            return response.IsSuccessStatusCode 
                ? Response.Ok() 
                : Response.Fail();
        })
        .WithLoadSimulations(
            Simulation.Inject(rate: 1000, interval: TimeSpan.FromSeconds(1), during: TimeSpan.FromSeconds(30))
        );
        
        // Act
        var result = NBomberRunner
            .RegisterScenarios(scenario)
            .Run();
            
        // Assert
        var stats = result.ScenarioStats[0];
        Assert.True(stats.Ok.Requests.RPS >= 800); // Accept 80% success rate
        Assert.True(stats.Ok.Latency.Percent95 < 500); // 95th percentile under 500ms
    }
}
```

---

This comprehensive syllabus covers all 31 chapters with detailed code examples, explanations, and best practices for building microservices with ASP.NET Core. Each chapter builds upon the previous ones, creating a complete understanding of microservices architecture, development, testing, and deployment.
