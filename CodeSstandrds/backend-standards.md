# Backend Development Standards & Guidelines

## Table of Contents
1. [Overview](#overview)
2. [Technology Stack](#technology-stack)
3. [Project Structure](#project-structure)
4. [Architecture Patterns](#architecture-patterns)
5. [Code Standards](#code-standards)
6. [Database Guidelines](#database-guidelines)
7. [API Development](#api-development)
8. [Security Standards](#security-standards)
9. [Testing Standards](#testing-standards)
10. [Best Practices](#best-practices)

---

## Overview

This document outlines the development standards and best practices for backend development teams working with .NET Core 8 and MS SQL Server.

**Key Principles:**
- Follow Clean Architecture principles
- Write maintainable, testable, and scalable code
- Implement proper error handling and logging
- Follow SOLID principles
- Ensure security best practices

---

## Technology Stack

### Core Technologies
- **Framework:** .NET Core 8 (LTS)
- **Language:** C# 12
- **Database:** Microsoft SQL Server 2019/2022
- **ORM:** Entity Framework Core 8
- **API Framework:** ASP.NET Core Web API
- **Authentication:** JWT Bearer Tokens / Identity Server
- **Documentation:** Swagger/OpenAPI

### Recommended Packages
```xml
<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="8.0.*" />
<PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="8.0.*" />
<PackageReference Include="Microsoft.AspNetCore.Authentication.JwtBearer" Version="8.0.*" />
<PackageReference Include="AutoMapper.Extensions.Microsoft.DependencyInjection" Version="12.*" />
<PackageReference Include="FluentValidation.AspNetCore" Version="11.*" />
<PackageReference Include="Serilog.AspNetCore" Version="8.*" />
<PackageReference Include="Swashbuckle.AspNetCore" Version="6.*" />
<PackageReference Include="MediatR" Version="12.*" />
```

---

## Project Structure

### Clean Architecture Structure

```
Solution/
├── src/
│   ├── ProjectName.Domain/               # Core business logic
│   │   ├── Entities/
│   │   │   ├── User.cs
│   │   │   ├── Order.cs
│   │   │   └── BaseEntity.cs
│   │   ├── Enums/
│   │   │   └── OrderStatus.cs
│   │   ├── ValueObjects/
│   │   │   └── Address.cs
│   │   ├── Interfaces/
│   │   │   ├── IRepository.cs
│   │   │   └── IUnitOfWork.cs
│   │   └── Exceptions/
│   │       └── DomainException.cs
│   │
│   ├── ProjectName.Application/          # Business logic & use cases
│   │   ├── DTOs/
│   │   │   ├── UserDto.cs
│   │   │   └── CreateUserDto.cs
│   │   ├── Interfaces/
│   │   │   └── IUserService.cs
│   │   ├── Services/
│   │   │   └── UserService.cs
│   │   ├── Mappings/
│   │   │   └── MappingProfile.cs
│   │   ├── Validators/
│   │   │   └── CreateUserValidator.cs
│   │   ├── Commands/
│   │   │   └── CreateUserCommand.cs
│   │   ├── Queries/
│   │   │   └── GetUserQuery.cs
│   │   └── Handlers/
│   │       ├── CreateUserCommandHandler.cs
│   │       └── GetUserQueryHandler.cs
│   │
│   ├── ProjectName.Infrastructure/       # Data access & external services
│   │   ├── Data/
│   │   │   ├── ApplicationDbContext.cs
│   │   │   ├── Configurations/
│   │   │   │   └── UserConfiguration.cs
│   │   │   ├── Migrations/
│   │   │   └── Seeds/
│   │   │       └── DefaultData.cs
│   │   ├── Repositories/
│   │   │   ├── GenericRepository.cs
│   │   │   ├── UserRepository.cs
│   │   │   └── UnitOfWork.cs
│   │   └── Services/
│   │       ├── EmailService.cs
│   │       └── FileStorageService.cs
│   │
│   ├── ProjectName.API/                  # Web API Layer
│   │   ├── Controllers/
│   │   │   ├── AuthController.cs
│   │   │   └── UsersController.cs
│   │   ├── Middlewares/
│   │   │   ├── ExceptionHandlingMiddleware.cs
│   │   │   └── RequestLoggingMiddleware.cs
│   │   ├── Filters/
│   │   │   └── ValidationFilter.cs
│   │   ├── Extensions/
│   │   │   └── ServiceCollectionExtensions.cs
│   │   ├── appsettings.json
│   │   ├── appsettings.Development.json
│   │   └── Program.cs
│   │
│   └── ProjectName.Shared/               # Shared utilities
│       ├── Constants/
│       │   └── AppConstants.cs
│       ├── Helpers/
│       │   └── DateTimeHelper.cs
│       └── Extensions/
│           └── StringExtensions.cs
│
└── tests/
    ├── ProjectName.UnitTests/
    │   ├── Domain/
    │   ├── Application/
    │   └── Infrastructure/
    ├── ProjectName.IntegrationTests/
    │   └── Controllers/
    └── ProjectName.FunctionalTests/
```

---

## Architecture Patterns

### 1. Clean Architecture Layers

#### Domain Layer
- Contains business entities and domain logic
- No dependencies on other layers
- Pure C# classes with business rules

```csharp
// Domain/Entities/User.cs
public class User : BaseEntity
{
    public string FirstName { get; private set; }
    public string LastName { get; private set; }
    public string Email { get; private set; }
    public DateTime CreatedAt { get; private set; }
    public bool IsActive { get; private set; }

    private User() { } // For EF Core

    public User(string firstName, string lastName, string email)
    {
        if (string.IsNullOrWhiteSpace(email))
            throw new DomainException("Email cannot be empty");

        FirstName = firstName;
        LastName = lastName;
        Email = email;
        CreatedAt = DateTime.UtcNow;
        IsActive = true;
    }

    public void Deactivate()
    {
        IsActive = false;
    }

    public void UpdateEmail(string newEmail)
    {
        if (string.IsNullOrWhiteSpace(newEmail))
            throw new DomainException("Email cannot be empty");

        Email = newEmail;
    }
}

// Domain/Entities/BaseEntity.cs
public abstract class BaseEntity
{
    public Guid Id { get; protected set; }
    public DateTime CreatedAt { get; protected set; }
    public DateTime? UpdatedAt { get; protected set; }
    public string? CreatedBy { get; protected set; }
    public string? UpdatedBy { get; protected set; }

    protected BaseEntity()
    {
        Id = Guid.NewGuid();
        CreatedAt = DateTime.UtcNow;
    }
}
```

#### Application Layer
- Contains business logic and use cases
- Depends only on Domain layer
- Defines interfaces for infrastructure

```csharp
// Application/Interfaces/IUserService.cs
public interface IUserService
{
    Task<UserDto> GetByIdAsync(Guid id, CancellationToken cancellationToken = default);
    Task<PagedResult<UserDto>> GetAllAsync(int pageNumber, int pageSize, CancellationToken cancellationToken = default);
    Task<UserDto> CreateAsync(CreateUserDto dto, CancellationToken cancellationToken = default);
    Task<UserDto> UpdateAsync(Guid id, UpdateUserDto dto, CancellationToken cancellationToken = default);
    Task<bool> DeleteAsync(Guid id, CancellationToken cancellationToken = default);
}

// Application/Services/UserService.cs
public class UserService : IUserService
{
    private readonly IUnitOfWork _unitOfWork;
    private readonly IMapper _mapper;
    private readonly ILogger<UserService> _logger;

    public UserService(
        IUnitOfWork unitOfWork,
        IMapper mapper,
        ILogger<UserService> logger)
    {
        _unitOfWork = unitOfWork;
        _mapper = mapper;
        _logger = logger;
    }

    public async Task<UserDto> CreateAsync(CreateUserDto dto, CancellationToken cancellationToken = default)
    {
        _logger.LogInformation("Creating new user with email: {Email}", dto.Email);

        var user = new User(dto.FirstName, dto.LastName, dto.Email);
        
        await _unitOfWork.Users.AddAsync(user, cancellationToken);
        await _unitOfWork.SaveChangesAsync(cancellationToken);

        _logger.LogInformation("User created successfully with ID: {UserId}", user.Id);

        return _mapper.Map<UserDto>(user);
    }

    // Other methods...
}
```

#### Infrastructure Layer
- Implements interfaces defined in Application layer
- Contains EF Core, repositories, external services

```csharp
// Infrastructure/Data/ApplicationDbContext.cs
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    public DbSet<User> Users => Set<User>();
    public DbSet<Order> Orders => Set<Order>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        // Apply all configurations from assembly
        modelBuilder.ApplyConfigurationsFromAssembly(Assembly.GetExecutingAssembly());
    }

    public override Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
    {
        // Update audit fields
        foreach (var entry in ChangeTracker.Entries<BaseEntity>())
        {
            if (entry.State == EntityState.Added)
            {
                entry.Entity.CreatedAt = DateTime.UtcNow;
            }
            else if (entry.State == EntityState.Modified)
            {
                entry.Entity.UpdatedAt = DateTime.UtcNow;
            }
        }

        return base.SaveChangesAsync(cancellationToken);
    }
}

// Infrastructure/Data/Configurations/UserConfiguration.cs
public class UserConfiguration : IEntityTypeConfiguration<User>
{
    public void Configure(EntityTypeBuilder<User> builder)
    {
        builder.ToTable("Users");

        builder.HasKey(u => u.Id);

        builder.Property(u => u.FirstName)
            .IsRequired()
            .HasMaxLength(100);

        builder.Property(u => u.LastName)
            .IsRequired()
            .HasMaxLength(100);

        builder.Property(u => u.Email)
            .IsRequired()
            .HasMaxLength(255);

        builder.HasIndex(u => u.Email)
            .IsUnique();

        builder.Property(u => u.CreatedAt)
            .IsRequired();
    }
}
```

### 2. Repository Pattern with Unit of Work

```csharp
// Domain/Interfaces/IRepository.cs
public interface IRepository<T> where T : BaseEntity
{
    Task<T?> GetByIdAsync(Guid id, CancellationToken cancellationToken = default);
    Task<List<T>> GetAllAsync(CancellationToken cancellationToken = default);
    Task<PagedResult<T>> GetPagedAsync(int pageNumber, int pageSize, CancellationToken cancellationToken = default);
    Task AddAsync(T entity, CancellationToken cancellationToken = default);
    Task AddRangeAsync(IEnumerable<T> entities, CancellationToken cancellationToken = default);
    void Update(T entity);
    void Remove(T entity);
    Task<bool> ExistsAsync(Guid id, CancellationToken cancellationToken = default);
}

// Infrastructure/Repositories/GenericRepository.cs
public class GenericRepository<T> : IRepository<T> where T : BaseEntity
{
    protected readonly ApplicationDbContext _context;
    protected readonly DbSet<T> _dbSet;

    public GenericRepository(ApplicationDbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }

    public virtual async Task<T?> GetByIdAsync(Guid id, CancellationToken cancellationToken = default)
    {
        return await _dbSet.FindAsync(new object[] { id }, cancellationToken);
    }

    public virtual async Task<List<T>> GetAllAsync(CancellationToken cancellationToken = default)
    {
        return await _dbSet.ToListAsync(cancellationToken);
    }

    public virtual async Task<PagedResult<T>> GetPagedAsync(
        int pageNumber, 
        int pageSize, 
        CancellationToken cancellationToken = default)
    {
        var totalCount = await _dbSet.CountAsync(cancellationToken);
        var items = await _dbSet
            .Skip((pageNumber - 1) * pageSize)
            .Take(pageSize)
            .ToListAsync(cancellationToken);

        return new PagedResult<T>(items, totalCount, pageNumber, pageSize);
    }

    public virtual async Task AddAsync(T entity, CancellationToken cancellationToken = default)
    {
        await _dbSet.AddAsync(entity, cancellationToken);
    }

    public virtual async Task AddRangeAsync(IEnumerable<T> entities, CancellationToken cancellationToken = default)
    {
        await _dbSet.AddRangeAsync(entities, cancellationToken);
    }

    public virtual void Update(T entity)
    {
        _dbSet.Update(entity);
    }

    public virtual void Remove(T entity)
    {
        _dbSet.Remove(entity);
    }

    public virtual async Task<bool> ExistsAsync(Guid id, CancellationToken cancellationToken = default)
    {
        return await _dbSet.AnyAsync(e => e.Id == id, cancellationToken);
    }
}

// Domain/Interfaces/IUnitOfWork.cs
public interface IUnitOfWork : IDisposable
{
    IRepository<User> Users { get; }
    IRepository<Order> Orders { get; }
    Task<int> SaveChangesAsync(CancellationToken cancellationToken = default);
    Task BeginTransactionAsync(CancellationToken cancellationToken = default);
    Task CommitTransactionAsync(CancellationToken cancellationToken = default);
    Task RollbackTransactionAsync(CancellationToken cancellationToken = default);
}

// Infrastructure/Repositories/UnitOfWork.cs
public class UnitOfWork : IUnitOfWork
{
    private readonly ApplicationDbContext _context;
    private IDbContextTransaction? _transaction;

    public IRepository<User> Users { get; }
    public IRepository<Order> Orders { get; }

    public UnitOfWork(ApplicationDbContext context)
    {
        _context = context;
        Users = new GenericRepository<User>(context);
        Orders = new GenericRepository<Order>(context);
    }

    public async Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
    {
        return await _context.SaveChangesAsync(cancellationToken);
    }

    public async Task BeginTransactionAsync(CancellationToken cancellationToken = default)
    {
        _transaction = await _context.Database.BeginTransactionAsync(cancellationToken);
    }

    public async Task CommitTransactionAsync(CancellationToken cancellationToken = default)
    {
        try
        {
            await SaveChangesAsync(cancellationToken);
            await _transaction?.CommitAsync(cancellationToken)!;
        }
        catch
        {
            await RollbackTransactionAsync(cancellationToken);
            throw;
        }
        finally
        {
            _transaction?.Dispose();
            _transaction = null;
        }
    }

    public async Task RollbackTransactionAsync(CancellationToken cancellationToken = default)
    {
        await _transaction?.RollbackAsync(cancellationToken)!;
        _transaction?.Dispose();
        _transaction = null;
    }

    public void Dispose()
    {
        _transaction?.Dispose();
        _context.Dispose();
    }
}
```

---

## Code Standards

### 1. Naming Conventions

- **Classes/Interfaces:** PascalCase (`UserService`, `IUserRepository`)
- **Methods:** PascalCase (`GetUserById`, `CreateAsync`)
- **Properties:** PascalCase (`FirstName`, `IsActive`)
- **Private fields:** _camelCase (`_unitOfWork`, `_logger`)
- **Parameters:** camelCase (`userId`, `cancellationToken`)
- **Constants:** PascalCase or UPPER_SNAKE_CASE
- **Async methods:** Suffix with `Async` (`GetByIdAsync`, `SaveChangesAsync`)

### 2. File Organization

```
✅ Good:
- One class per file
- File name matches class name
- UserService.cs contains UserService class

❌ Bad:
- Multiple unrelated classes in one file
- File name doesn't match class name
```

### 3. Code Documentation

```csharp
/// <summary>
/// Retrieves a user by their unique identifier.
/// </summary>
/// <param name="id">The unique identifier of the user.</param>
/// <param name="cancellationToken">Token to cancel the operation.</param>
/// <returns>The user if found, otherwise null.</returns>
/// <exception cref="ArgumentException">Thrown when id is empty.</exception>
public async Task<UserDto?> GetByIdAsync(Guid id, CancellationToken cancellationToken = default)
{
    if (id == Guid.Empty)
        throw new ArgumentException("User ID cannot be empty", nameof(id));

    var user = await _unitOfWork.Users.GetByIdAsync(id, cancellationToken);
    return user != null ? _mapper.Map<UserDto>(user) : null;
}
```

### 4. SOLID Principles

#### Single Responsibility Principle
```csharp
// ✅ Good: Each class has one responsibility
public class UserService { /* User business logic */ }
public class EmailService { /* Email sending logic */ }
public class UserValidator { /* User validation logic */ }

// ❌ Bad: UserService doing too much
public class UserService 
{ 
    public void CreateUser() { }
    public void SendEmail() { }
    public void ValidateUser() { }
}
```

#### Dependency Inversion
```csharp
// ✅ Good: Depend on abstractions
public class UserService
{
    private readonly IUserRepository _repository;
    private readonly IEmailService _emailService;
    
    public UserService(IUserRepository repository, IEmailService emailService)
    {
        _repository = repository;
        _emailService = emailService;
    }
}

// ❌ Bad: Depend on concrete implementations
public class UserService
{
    private readonly UserRepository _repository = new UserRepository();
    private readonly EmailService _emailService = new EmailService();
}
```

### 5. Async/Await Best Practices

```csharp
// ✅ Good: Async all the way
public async Task<UserDto> GetUserAsync(Guid id)
{
    var user = await _repository.GetByIdAsync(id);
    return _mapper.Map<UserDto>(user);
}

// ✅ Good: Use ConfigureAwait(false) in libraries
public async Task<User> GetByIdAsync(Guid id)
{
    return await _dbSet.FindAsync(id).ConfigureAwait(false);
}

// ❌ Bad: Blocking async code
public UserDto GetUser(Guid id)
{
    var user = _repository.GetByIdAsync(id).Result; // Don't do this!
    return _mapper.Map<UserDto>(user);
}
```

---

## Database Guidelines

### 1. Entity Framework Core Configuration

```csharp
// Program.cs
builder.Services.AddDbContext<ApplicationDbContext>(options =>
{
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection"),
        sqlOptions =>
        {
            sqlOptions.EnableRetryOnFailure(
                maxRetryCount: 3,
                maxRetryDelay: TimeSpan.FromSeconds(30),
                errorNumbersToAdd: null);
            sqlOptions.CommandTimeout(30);
            sqlOptions.MigrationsAssembly(typeof(ApplicationDbContext).Assembly.FullName);
        });

    if (builder.Environment.IsDevelopment())
    {
        options.EnableSensitiveDataLogging();
        options.EnableDetailedErrors();
    }
});
```

### 2. Connection String (appsettings.json)

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyAppDb;User Id=sa;Password=YourPassword;TrustServerCertificate=True;MultipleActiveResultSets=true"
  }
}
```

### 3. Migrations

```bash
# Add a new migration
dotnet ef migrations add InitialCreate --project src/ProjectName.Infrastructure --startup-project src/ProjectName.API

# Update database
dotnet ef database update --project src/ProjectName.Infrastructure --startup-project src/ProjectName.API

# Remove last migration
dotnet ef migrations remove --project src/ProjectName.Infrastructure --startup-project src/ProjectName.API

# Generate SQL script
dotnet ef migrations script --project src/ProjectName.Infrastructure --startup-project src/ProjectName.API --output migration.sql
```

### 4. Database Best Practices

```csharp
// ✅ Good: Use appropriate data types
builder.Property(u => u.Email)
    .IsRequired()
    .HasMaxLength(255)
    .IsUnicode(false); // Use varchar instead of nvarchar for ASCII

builder.Property(o => o.TotalAmount)
    .HasPrecision(18, 2); // For decimal values

// ✅ Good: Create indexes
builder.HasIndex(u => u.Email).IsUnique();
builder.HasIndex(o => o.OrderDate);
builder.HasIndex(o => new { o.CustomerId, o.OrderDate });

// ✅ Good: Use navigation properties
builder.HasMany(c => c.Orders)
    .WithOne(o => o.Customer)
    .HasForeignKey(o => o.CustomerId)
    .OnDelete(DeleteBehavior.Restrict);
```

### 5. Queries Optimization

```csharp
// ✅ Good: Use AsNoTracking for read-only queries
public async Task<List<UserDto>> GetAllUsersAsync()
{
    return await _context.Users
        .AsNoTracking()
        .Select(u => new UserDto
        {
            Id = u.Id,
            FullName = u.FirstName + " " + u.LastName,
            Email = u.Email
        })
        .ToListAsync();
}

// ✅ Good: Use projection to select only needed fields
public async Task<UserSummaryDto> GetUserSummaryAsync(Guid id)
{
    return await _context.Users
        .Where(u => u.Id == id)
        .Select(u => new UserSummaryDto
        {
            Name = u.FirstName + " " + u.LastName,
            OrderCount = u.Orders.Count()
        })
        .FirstOrDefaultAsync();
}

// ✅ Good: Use Include for eager loading when needed
public async Task<User> GetUserWithOrdersAsync(Guid id)
{
    return await _context.Users
        .Include(u => u.Orders)
        .ThenInclude(o => o.OrderItems)
        .FirstOrDefaultAsync(u => u.Id == id);
}

// ❌ Bad: N+1 query problem
public async Task<List<UserDto>> GetUsersWithOrderCount()
{
    var users = await _context.Users.ToListAsync();
    foreach (var user in users)
    {
        user.OrderCount = await _context.Orders.CountAsync(o => o.UserId == user.Id);
    }
    return _mapper.Map<List<UserDto>>(users);
}
```

---

## API Development

### 1. Controller Structure

```csharp
[ApiController]
[Route("api/[controller]")]
[Produces("application/json")]
public class UsersController : ControllerBase
{
    private readonly IUserService _userService;
    private readonly ILogger<UsersController> _logger;

    public UsersController(
        IUserService userService,
        ILogger<UsersController> logger)
    {
        _userService = userService;
        _logger = logger;
    }

    /// <summary>
    /// Retrieves all users with pagination
    /// </summary>
    /// <param name="pageNumber">Page number (default: 1)</param>
    /// <param name="pageSize">Page size (default: 10, max: 100)</param>
    /// <returns>Paginated list of users</returns>
    [HttpGet]
    [ProducesResponseType(typeof(PagedResult<UserDto>), StatusCodes.Status200OK)]
    public async Task<ActionResult<PagedResult<UserDto>>> GetAll(
        [FromQuery] int pageNumber = 1,
        [FromQuery] int pageSize = 10)
    {
        pageSize = Math.Min(pageSize, 100); // Limit max page size
        var result = await _userService.GetAllAsync(pageNumber, pageSize);
        return Ok(result);
    }

    /// <summary>
    /// Retrieves a specific user by ID
    /// </summary>
    /// <param name="id">User ID</param>
    /// <returns>User details</returns>
    [HttpGet("{id}")]
    [ProducesResponseType(typeof(UserDto), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<UserDto>> GetById(Guid id)
    {
        var user = await _userService.GetByIdAsync(id);
        
        if (user == null)
            return NotFound(new { message = "User not found" });

        return Ok(user);
    }

    /// <summary>
    /// Creates a new user
    /// </summary>
    /// <param name="dto">User creation data</param>
    /// <returns>Created user</returns>
    [HttpPost]
    [ProducesResponseType(typeof(UserDto), StatusCodes.Status201Created)]
    [ProducesResponseType(typeof(ValidationProblemDetails), StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<UserDto>> Create([FromBody] CreateUserDto dto)
    {
        var user = await _userService.CreateAsync(dto);
        return CreatedAtAction(nameof(GetById), new { id = user.Id }, user);
    }

    /// <summary>
    /// Updates an existing user
    /// </summary>
    /// <param name="id">User ID</param>
    /// <param name="dto">User update data</param>
    /// <returns>Updated user</returns>
    [HttpPut("{id}")]
    [ProducesResponseType(typeof(UserDto), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [ProducesResponseType(typeof(ValidationProblemDetails), StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<UserDto>> Update(Guid id, [FromBody] UpdateUserDto dto)
    {
        var user = await _userService.UpdateAsync(id, dto);
        
        if (user == null)
            return NotFound(new { message = "User not found" });

        return Ok(user);
    }

    /// <summary>
    /// Deletes a user
    /// </summary>
    /// <param name="id">User ID</param>
    /// <returns>No content</returns>
    [HttpDelete("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> Delete(Guid id)
    {
        var result = await _userService.DeleteAsync(id);
        
        if (!result)
            return NotFound(new { message = "User not found" });

        return NoContent();
    }
}
```

### 2. DTOs (Data Transfer Objects)

```csharp
// Application/DTOs/UserDto.cs
public record UserDto
{
    public Guid Id { get; init; }
    public string FirstName { get; init; } = string.Empty;
    public string LastName { get; init; } = string.Empty;
    public string Email { get; init; } = string.Empty;
    public string FullName => $"{FirstName} {LastName}";
    public DateTime CreatedAt { get; init; }
    public bool IsActive { get; init; }
}

public record CreateUserDto
{
    public string FirstName { get; init; } = string.Empty;
    public string LastName { get; init; } = string.Empty;
    public string Email { get; init; } = string.Empty;
}

public record UpdateUserDto
{
    public string FirstName { get; init; } = string.Empty;
    public string LastName { get; init; } = string.Empty;
    public string Email { get; init; } = string.Empty;
}

public record PagedResult<T>
{
    public List<T> Items { get; init; } = new();
    public int TotalCount { get; init; }
    public int PageNumber { get; init; }
    public int PageSize { get; init; }
    public int TotalPages => (int)Math.Ceiling(TotalCount / (double)PageSize);
    public bool HasPreviousPage => PageNumber > 1;
    public bool HasNextPage => PageNumber < TotalPages;

    public PagedResult() { }

    public PagedResult(List<T> items, int totalCount, int pageNumber, int pageSize)
    {
        Items = items;
        TotalCount = totalCount;
        PageNumber = pageNumber;
        PageSize = pageSize;
    }
}
```

### 3. Validation with FluentValidation

```csharp
// Application/Validators/CreateUserValidator.cs
public class CreateUserValidator : AbstractValidator<CreateUserDto>
{
    public CreateUserValidator()
    {
        RuleFor(x => x.FirstName)
            .NotEmpty().WithMessage("First name is required")
            .MaximumLength(100).WithMessage("First name must not exceed 100 characters");

        RuleFor(x => x.LastName)
            .NotEmpty().WithMessage("Last name is required")
            .MaximumLength(100).WithMessage("Last name must not exceed 100 characters");

        RuleFor(x => x.Email)
            .NotEmpty().WithMessage("Email is required")
            .EmailAddress().WithMessage("Invalid email format")
            .MaximumLength(255).WithMessage("Email must not exceed 255 characters");
    }
}

// Register in Program.cs
builder.Services.AddFluentValidationAutoValidation();
builder.Services.AddValidatorsFromAssemblyContaining<CreateUserValidator>();
```

### 4. AutoMapper Configuration

```csharp
// Application/Mappings/MappingProfile.cs
public class MappingProfile : Profile
{
    public MappingProfile()
    {
        // Entity to DTO
        CreateMap<User, UserDto>();
        
        // DTO to Entity
        CreateMap<CreateUserDto, User>()
            .ConstructUsing(dto => new User(dto.FirstName, dto.LastName, dto.Email));

        CreateMap<UpdateUserDto, User>()
            .ForAllMembers(opts => opts.Condition((src, dest, srcMember) => srcMember != null));
    }
}

// Register in Program.cs
builder.Services.AddAutoMapper(typeof(MappingProfile));
```

### 5. Global Exception Handling

```csharp
// API/Middlewares/ExceptionHandlingMiddleware.cs
public class ExceptionHandlingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ExceptionHandlingMiddleware> _logger;

    public ExceptionHandlingMiddleware(
        RequestDelegate next,
        ILogger<ExceptionHandlingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "An unhandled exception occurred");
            await HandleExceptionAsync(context, ex);
        }
    }

    private static Task HandleExceptionAsync(HttpContext context, Exception exception)
    {
        context.Response.ContentType = "application/json";

        var (statusCode, message) = exception switch
        {
            DomainException => (StatusCodes.Status400BadRequest, exception.Message),
            NotFoundException => (StatusCodes.Status404NotFound, exception.Message),
            UnauthorizedAccessException => (StatusCodes.Status401Unauthorized, "Unauthorized"),
            _ => (StatusCodes.Status500InternalServerError, "An error occurred processing your request")
        };

        context.Response.StatusCode = statusCode;

        var response = new
        {
            statusCode,
            message,
            timestamp = DateTime.UtcNow
        };

        return context.Response.WriteAsJsonAsync(response);
    }
}

// Register in Program.cs
app.UseMiddleware<ExceptionHandlingMiddleware>();
```

---

## Security Standards

### 1. JWT Authentication Setup

```csharp
// Program.cs
var jwtSettings = builder.Configuration.GetSection("JwtSettings");

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
        ValidIssuer = jwtSettings["Issuer"],
        ValidAudience = jwtSettings["Audience"],
        IssuerSigningKey = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(jwtSettings["SecretKey"]!))
    };
});

builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminOnly", policy => policy.RequireRole("Admin"));
    options.AddPolicy("UserOrAdmin", policy => policy.RequireRole("User", "Admin"));
});
```

```json
// appsettings.json
{
  "JwtSettings": {
    "SecretKey": "YourVerySecureSecretKeyHere_MinimumLength32Characters!",
    "Issuer": "YourAppName",
    "Audience": "YourAppName",
    "ExpiryInMinutes": 60
  }
}
```

### 2. Secure Controller Actions

```csharp
[Authorize] // Requires authentication
[ApiController]
[Route("api/[controller]")]
public class SecureController : ControllerBase
{
    [HttpGet]
    [Authorize(Roles = "Admin")] // Requires Admin role
    public IActionResult AdminOnly()
    {
        return Ok("Admin access granted");
    }

    [HttpGet("user")]
    [Authorize(Policy = "UserOrAdmin")] // Requires User or Admin role
    public IActionResult UserAccess()
    {
        var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        return Ok($"User {userId} access granted");
    }

    [AllowAnonymous] // Public endpoint
    [HttpGet("public")]
    public IActionResult Public()
    {
        return Ok("Public access");
    }
}
```

### 3. Password Hashing

```csharp
// Use ASP.NET Core Identity's PasswordHasher
public class AuthService
{
    private readonly IPasswordHasher<User> _passwordHasher;

    public AuthService(IPasswordHasher<User> passwordHasher)
    {
        _passwordHasher = passwordHasher;
    }

    public string HashPassword(User user, string password)
    {
        return _passwordHasher.HashPassword(user, password);
    }

    public bool VerifyPassword(User user, string hashedPassword, string providedPassword)
    {
        var result = _passwordHasher.VerifyHashedPassword(user, hashedPassword, providedPassword);
        return result == PasswordVerificationResult.Success;
    }
}
```

### 4. CORS Configuration

```csharp
// Program.cs
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowSpecificOrigins", builder =>
    {
        builder.WithOrigins("https://yourdomain.com", "https://www.yourdomain.com")
               .AllowAnyMethod()
               .AllowAnyHeader()
               .AllowCredentials();
    });
});

// In middleware pipeline
app.UseCors("AllowSpecificOrigins");
```

### 5. Input Sanitization

```csharp
// Always validate and sanitize user inputs
public class SecurityHelper
{
    public static string SanitizeInput(string input)
    {
        if (string.IsNullOrWhiteSpace(input))
            return string.Empty;

        // Remove dangerous characters
        return input
            .Replace("<", "&lt;")
            .Replace(">", "&gt;")
            .Replace("&", "&amp;")
            .Replace("\"", "&quot;")
            .Replace("'", "&#x27;")
            .Replace("/", "&#x2F;");
    }
}
```

---

## Testing Standards

### 1. Unit Testing Structure

```csharp
// Tests/UnitTests/Application/Services/UserServiceTests.cs
public class UserServiceTests
{
    private readonly Mock<IUnitOfWork> _unitOfWorkMock;
    private readonly Mock<IMapper> _mapperMock;
    private readonly Mock<ILogger<UserService>> _loggerMock;
    private readonly UserService _sut; // System Under Test

    public UserServiceTests()
    {
        _unitOfWorkMock = new Mock<IUnitOfWork>();
        _mapperMock = new Mock<IMapper>();
        _loggerMock = new Mock<ILogger<UserService>>();
        _sut = new UserService(_unitOfWorkMock.Object, _mapperMock.Object, _loggerMock.Object);
    }

    [Fact]
    public async Task CreateAsync_ValidUser_ReturnsUserDto()
    {
        // Arrange
        var createDto = new CreateUserDto 
        { 
            FirstName = "John", 
            LastName = "Doe", 
            Email = "john@example.com" 
        };
        var user = new User(createDto.FirstName, createDto.LastName, createDto.Email);
        var userDto = new UserDto { Id = user.Id, FirstName = "John", LastName = "Doe" };

        _mapperMock.Setup(m => m.Map<UserDto>(It.IsAny<User>())).Returns(userDto);
        _unitOfWorkMock.Setup(u => u.Users.AddAsync(It.IsAny<User>(), default))
            .Returns(Task.CompletedTask);
        _unitOfWorkMock.Setup(u => u.SaveChangesAsync(default)).ReturnsAsync(1);

        // Act
        var result = await _sut.CreateAsync(createDto);

        // Assert
        Assert.NotNull(result);
        Assert.Equal("John", result.FirstName);
        _unitOfWorkMock.Verify(u => u.SaveChangesAsync(default), Times.Once);
    }

    [Fact]
    public async Task GetByIdAsync_NonExistentUser_ReturnsNull()
    {
        // Arrange
        var userId = Guid.NewGuid();
        _unitOfWorkMock.Setup(u => u.Users.GetByIdAsync(userId, default))
            .ReturnsAsync((User?)null);

        // Act
        var result = await _sut.GetByIdAsync(userId);

        // Assert
        Assert.Null(result);
    }
}
```

### 2. Integration Testing

```csharp
// Tests/IntegrationTests/Controllers/UsersControllerTests.cs
public class UsersControllerIntegrationTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;
    private readonly WebApplicationFactory<Program> _factory;

    public UsersControllerIntegrationTests(WebApplicationFactory<Program> factory)
    {
        _factory = factory.WithWebHostBuilder(builder =>
        {
            builder.ConfigureServices(services =>
            {
                // Use in-memory database for testing
                var descriptor = services.SingleOrDefault(
                    d => d.ServiceType == typeof(DbContextOptions<ApplicationDbContext>));
                
                if (descriptor != null)
                    services.Remove(descriptor);

                services.AddDbContext<ApplicationDbContext>(options =>
                {
                    options.UseInMemoryDatabase("TestDb");
                });
            });
        });

        _client = _factory.CreateClient();
    }

    [Fact]
    public async Task GetAll_ReturnsSuccessAndCorrectContentType()
    {
        // Act
        var response = await _client.GetAsync("/api/users");

        // Assert
        response.EnsureSuccessStatusCode();
        Assert.Equal("application/json; charset=utf-8", 
            response.Content.Headers.ContentType?.ToString());
    }

    [Fact]
    public async Task Create_ValidUser_ReturnsCreated()
    {
        // Arrange
        var createDto = new CreateUserDto
        {
            FirstName = "John",
            LastName = "Doe",
            Email = "john@example.com"
        };

        // Act
        var response = await _client.PostAsJsonAsync("/api/users", createDto);

        // Assert
        Assert.Equal(HttpStatusCode.Created, response.StatusCode);
        var user = await response.Content.ReadFromJsonAsync<UserDto>();
        Assert.NotNull(user);
        Assert.Equal("John", user.FirstName);
    }
}
```

---

## Best Practices

### 1. Logging with Serilog

```csharp
// Program.cs
using Serilog;

// Configure Serilog
Log.Logger = new LoggerConfiguration()
    .ReadFrom.Configuration(builder.Configuration)
    .Enrich.FromLogContext()
    .WriteTo.Console()
    .WriteTo.File("logs/log-.txt", rollingInterval: RollingInterval.Day)
    .CreateLogger();

builder.Host.UseSerilog();

// In services
public class UserService
{
    private readonly ILogger<UserService> _logger;

    public async Task<UserDto> CreateAsync(CreateUserDto dto)
    {
        _logger.LogInformation("Creating user with email: {Email}", dto.Email);
        
        try
        {
            // Business logic
            _logger.LogInformation("User created successfully: {UserId}", user.Id);
            return userDto;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error creating user with email: {Email}", dto.Email);
            throw;
        }
    }
}
```

### 2. Dependency Injection Registration

```csharp
// API/Extensions/ServiceCollectionExtensions.cs
public static class ServiceCollectionExtensions
{
    public static IServiceCollection AddApplicationServices(this IServiceCollection services)
    {
        services.AddScoped<IUserService, UserService>();
        services.AddScoped<IOrderService, OrderService>();
        return services;
    }

    public static IServiceCollection AddInfrastructureServices(
        this IServiceCollection services, 
        IConfiguration configuration)
    {
        services.AddDbContext<ApplicationDbContext>(options =>
            options.UseSqlServer(configuration.GetConnectionString("DefaultConnection")));

        services.AddScoped<IUnitOfWork, UnitOfWork>();
        services.AddScoped(typeof(IRepository<>), typeof(GenericRepository<>));
        
        return services;
    }
}

// Usage in Program.cs
builder.Services.AddApplicationServices();
builder.Services.AddInfrastructureServices(builder.Configuration);
```

### 3. Configuration Management

```json
// appsettings.json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning",
      "Microsoft.EntityFrameworkCore": "Warning"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyAppDb;..."
  },
  "JwtSettings": {
    "SecretKey": "Your-Secret-Key",
    "Issuer": "YourApp",
    "Audience": "YourApp",
    "ExpiryInMinutes": 60
  },
  "AllowedHosts": "*"
}
```

```csharp
// Strongly-typed configuration
public class JwtSettings
{
    public string SecretKey { get; set; } = string.Empty;
    public string Issuer { get; set; } = string.Empty;
    public string Audience { get; set; } = string.Empty;
    public int ExpiryInMinutes { get; set; }
}

// Register in Program.cs
builder.Services.Configure<JwtSettings>(builder.Configuration.GetSection("JwtSettings"));

// Use in services
public class TokenService
{
    private readonly JwtSettings _jwtSettings;

    public TokenService(IOptions<JwtSettings> jwtSettings)
    {
        _jwtSettings = jwtSettings.Value;
    }
}
```

### 4. API Versioning

```csharp
// Install: Asp.Versioning.Http
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
});

[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[ApiVersion("1.0")]
public class UsersController : ControllerBase
{
    // v1 endpoints
}

[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[ApiVersion("2.0")]
public class UsersV2Controller : ControllerBase
{
    // v2 endpoints
}
```

### 5. Health Checks

```csharp
// Program.cs
builder.Services.AddHealthChecks()
    .AddDbContextCheck<ApplicationDbContext>()
    .AddSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")!);

app.MapHealthChecks("/health");
```

### 6. Swagger/OpenAPI Configuration

```csharp
// Program.cs
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "My API",
        Version = "v1",
        Description = "API documentation"
    });

    // Add JWT authentication
    options.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
    {
        Description = "JWT Authorization header using the Bearer scheme.",
        Name = "Authorization",
        In = ParameterLocation.Header,
        Type = SecuritySchemeType.ApiKey,
        Scheme = "Bearer"
    });

    options.AddSecurityRequirement(new OpenApiSecurityRequirement
    {
        {
            new OpenApiSecurityScheme
            {
                Reference = new OpenApiReference
                {
                    Type = ReferenceType.SecurityScheme,
                    Id = "Bearer"
                }
            },
            Array.Empty<string>()
        }
    });

    // Include XML comments
    var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
    var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
    options.IncludeXmlComments(xmlPath);
});

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
```

---

## Code Review Checklist

- [ ] Code follows Clean Architecture principles
- [ ] SOLID principles are applied
- [ ] Proper error handling and logging
- [ ] All methods have XML documentation
- [ ] Async/await used correctly
- [ ] Database queries are optimized
- [ ] Proper validation implemented
- [ ] Security best practices followed
- [ ] Unit tests written and passing
- [ ] No hardcoded values (use configuration)
- [ ] Proper dependency injection
- [ ] API endpoints properly documented
- [ ] No SQL injection vulnerabilities
- [ ] Sensitive data not logged

---

## Additional Resources

- [.NET Documentation](https://learn.microsoft.com/en-us/dotnet/)
- [Entity Framework Core Documentation](https://learn.microsoft.com/en-us/ef/core/)
- [ASP.NET Core Documentation](https://learn.microsoft.com/en-us/aspnet/core/)
- [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [SOLID Principles](https://www.digitalocean.com/community/conceptual-articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design)

---

**Last Updated:** January 2026  
**Version:** 1.0
