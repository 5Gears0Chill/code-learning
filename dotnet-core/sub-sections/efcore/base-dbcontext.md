# Designing a Clean Approach to the DbContext

Implementing the repository pattern to the best of its ability requires a rugged framework upon which to build your application. The following article shows a useful approach when implementing the core functionalities used in EF Core.

The goal is to remove the DbContext from being injected into any repository. A repository can only make calls to one Database but any amount of tables within that database. Traditionally the DbContext is injected into the constructor of a repository and then is used to create queries. The CRUD functionalities of a repository are implemented in an interface `IRepository ` which would have a concrete implementation as to abstract the logic away from any specific repository.

The proposed solution is to abstract the core logic of CRUD to the DbContext such that more complex calls can be constructed in the repository. Additionally this allows us to control the creation and disposal of the DbContext rather that submitting it to the lifetime scope of the Solution.

**<u>Step 1: Define an implementation of the DbContext as `IDbContext`</u>**

```c#
/// <summary>
/// Represent DbContext
/// </summary>
public partial interface IDbContext : IDisposable
{
    DbSet<TEntity> Set<TEntity>() where TEntity : BaseEntity;

    Task<int> SaveChangesAsync();

    void SetEntityState<TEntity>(TEntity entity, int state) where TEntity : BaseEntity;
    /// <summary>
    /// Gets an entity by identifier
    /// </summary>
    /// <param name="id">Identifier</param>
    /// <returns>Entity</returns>
    Task<TEntity> GetByIdAsync<TEntity>(object id) where TEntity : BaseEntity;

    /// <summary>
    /// Inserts an entity async
    /// </summary>
    /// <param name="entity">Entity</param>
    /// <returns>Success Result</returns>
    Task<int> InsertAsync<TEntity>(TEntity entity) where TEntity : BaseEntity;

    /// <summary>
    /// Inserts multiple entities async
    /// </summary>
    /// <param name="entities">Entities</param>
    /// <returns>Success Result and Rows Affected</returns>
    Task<int> InsertEnumerableAsync<TEntity>(IEnumerable<TEntity> entities) where TEntity : BaseEntity;

    /// <summary>
    /// Update an entity async
    /// </summary>
    /// <param name="entity">Entity</param>
    /// <returns>Success Result and Rows Affected</returns>
    Task<int> UpdateAsync<TEntity>(TEntity entity) where TEntity : BaseEntity;

    /// <summary>
    /// Update multiple entities async
    /// </summary>
    /// <param name="entities">Entities</param>
    /// <returns>Success Result and Rows Affected</returns>
    Task<int> UpdateEnumerableAsync<TEntity>(IEnumerable<TEntity> entities) where TEntity : BaseEntity;

    /// <summary>
    /// Delete an entity async
    /// </summary>
    /// <param name="entity">Entity</param>
    /// <returns>Success Result and Rows Affected</returns>
    Task<int> DeleteAsync<TEntity>(TEntity entity) where TEntity : BaseEntity;

    /// <summary>
    /// Delete multiple entities async
    /// </summary>
    /// <param name="entities">Entities</param>
    /// <returns>Success Result and Rows Affected</returns>
    Task<int> DeleteEnumerableAsync<TEntity>(IEnumerable<TEntity> entities) where TEntity : BaseEntity;
}
```

 Note that `IDbContext` implements `IDisposable` which gives any concrete implementation of this interface the method `base.Dispose()` 

Additionally note that all methods are `async`. For more information about Asynchronous Programming refer to [Asynchronous Programming with DOTNET Core](../async-programming.md). The goal is to ensure that any method call to the database has the ability to be scaled to prevent locking. This includes all elements of CRUD.

**<u>Step 2: Defining a concrete implementation of the `IDbContext`</u>**

The next step involves creating a generic but definitive implementation of `Microsoft.EntityFrameworkCore.DbContext` as well as `IDbContext`. This will be known as `BaseDbContext` and all specific Contexts will implement this base.

```c#
public class BaseDbContext : DbContext, IDbContext
{
    public BaseDbContext()
    {
    }

    public BaseDbContext(DbContextOptions<MyDbContext> options)
        : base(options)
    {
    }

    public virtual new DbSet<TEntity> Set<TEntity>() where TEntity : BaseEntity
        => base.Set<TEntity>();

    public async Task<int> DeleteAsync<TEntity>(TEntity entity) where TEntity : BaseEntity
    {
        if (entity == null)
        {
            throw new ArgumentNullException(nameof(entity));
        }
        try
        {
            Remove(entity);
            return await SaveChangesAsync();
        }
        catch (DbUpdateException ex)
        {
            throw new Exception(await GetFullErrorTextAndRollbackEntityChanges(ex), ex);
        }
    }

    public async Task<int> DeleteEnumerableAsync<TEntity>(IEnumerable<TEntity> entities) where TEntity : BaseEntity
    {
        if (entities == null)
        {
            throw new ArgumentNullException(nameof(entities));
        }
        try
        {
            RemoveRange(entities);
            return await SaveChangesAsync();
        }
        catch (DbUpdateException ex)
        {
            throw new Exception(await GetFullErrorTextAndRollbackEntityChanges(ex), ex);
        }
    }
    public async Task<TEntity> GetByIdAsync<TEntity>(object id) where TEntity : BaseEntity
    {
        return await Set<TEntity>().FindAsync(id);
    }

    public async Task<int> InsertAsync<TEntity>(TEntity entity) where TEntity : BaseEntity
    {
        if (entity == null)
        {
            throw new ArgumentNullException(nameof(entity));
        }
        try
        {
            await AddAsync(entity);
            return await SaveChangesAsync();
        }
        catch (DbUpdateException ex)
        {
            throw new Exception(await GetFullErrorTextAndRollbackEntityChanges(ex), ex);
        }
    }

    public async Task<int> InsertEnumerableAsync<TEntity>(IEnumerable<TEntity> entities) where TEntity : BaseEntity
    {
        if (entities == null)
        {
            throw new ArgumentNullException(nameof(entities));
        }
        try
        {
            await AddRangeAsync(entities);
            return await SaveChangesAsync();
        }
        catch (DbUpdateException ex)
        {
            throw new Exception(await GetFullErrorTextAndRollbackEntityChanges(ex), ex);
        }
    }

    public Task<int> SaveChangesAsync()
        => base.SaveChangesAsync();

    public void SetEntityState<TEntity>(TEntity entity, int state) where TEntity : BaseEntity
    {
        Attach(entity).State = (EntityState)state;
    }

    public async Task<int> UpdateAsync<TEntity>(TEntity entity) where TEntity : BaseEntity
    {
        if (entity == null)
        {
            throw new ArgumentNullException(nameof(entity));
        }
        try
        {
            Update(entity);
            return await SaveChangesAsync();
        }
        catch (DbUpdateException ex)
        {
            throw new Exception(await GetFullErrorTextAndRollbackEntityChanges(ex), ex);
        }
    }

    public async Task<int> UpdateEnumerableAsync<TEntity>(IEnumerable<TEntity> entities) where TEntity : BaseEntity
    {
        if (entities == null)
        {
            throw new ArgumentNullException(nameof(entities));
        }
        try
        {
            UpdateRange(entities);
            return await SaveChangesAsync();

        }
        catch (DbUpdateException ex)
        {
            throw new Exception(await GetFullErrorTextAndRollbackEntityChanges(ex), ex);
        }
    }

    protected async Task<string> GetFullErrorTextAndRollbackEntityChanges(DbUpdateException ex)
    {
        if (this is DbContext dbContext)
        {
            var entries = dbContext.ChangeTracker.Entries()
                .Where(e => e.State == EntityState.Added || e.State == EntityState.Modified).ToList();
            entries.ForEach(entry =>
                            {
                                try
                                {
                                    entry.State = EntityState.Unchanged;
                                }
                                catch (InvalidOperationException)
                                {
                                    //ignored
                                }
                            });
        }

        try
        {
            await this.SaveChangesAsync();
            return ex.ToString();
        }
        catch (Exception e)
        {
            //if after the rollabck of changes the context is still not saving,
            //return the full text of the exception that occured when saving
            return e.ToString();
        }
    }  
}
```

The above class is essential for managing the state of EF Cores tracking mechanism as well as logging and saving changes to the Database.

This will serve as the base for all custom DbContexts.

## Final Implementation of the Db Context

The Final DbContext will have the following structure:

```c#
public partial class MyDbContext : BaseDbContext
{
    //Custom Logging to console
    public static readonly ILoggerFactory loggerFactory 
        = LoggerFactory.Create(builder => { builder.AddConsole(); });

    //Injected AppSettings
    private readonly IAppSettings _appSettings;

    //Constructor to project DI Injection
    public PaladinsDbContext(IAppSettings appSettings)
    {
        _appSettings = appSettings;
    }
 	//Constructor to project DI Injection and DbContextOptions (if any)
    public PaladinsDbContext(DbContextOptions<PaladinsDbContext> options, AppSettings appSettings)
        : base(options)
        {
            _appSettings = appSettings;
        }
	//DbSet representing a table in the database
    public virtual DbSet<User> User { get; set; }
    

    //Configuration for Db Context 
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        if (!optionsBuilder.IsConfigured)
        {            
            optionsBuilder
                .UseLoggerFactory(loggerFactory)
                .EnableSensitiveDataLogging() //logging for development
                .UseSqlServer(_appSettings.GetDataConnections().ConnectionString); //Connection String
        }

    }
    
    //Configurations for each table
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.ApplyConfiguration(new UserConfiguration());
        //Apply All Table Configurations
    }  
}
```


This concludes how to create a clean architecture approach to EF Core's DbContext.

