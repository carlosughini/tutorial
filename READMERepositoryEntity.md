#Canducci EntityFramework Repository

[![Canducci.EntityFramework.Repository](http://i1194.photobucket.com/albums/aa377/netdragoon1/1452823324_icon-89-document-file-sql_zpslt5kmpu9.png)](https://www.nuget.org/packages/CanducciEntityFrameworkRepository/)

[![NuGet](https://img.shields.io/nuget/dt/CanducciEntityFrameworkRepository.svg?style=plastic&label=downloads)](https://www.nuget.org/packages/CanducciEntityFrameworkRepository/)
[![NuGet](https://img.shields.io/nuget/v/CanducciEntityFrameworkRepository.svg?style=plastic&label=version)](https://www.nuget.org/packages/CanducciEntityFrameworkRepository/)

## NUGET

```Csharp

PM> Install-Package CanducciEntityFrameworkRepository

```

##HOW TO?

Use _namespace_ `using Canducci.EntityFramework.Repository.Contracts;`

The `BaseEFEntities`: `DbContext` and `Notice`: `DbSet`.

```Csharp

using Canducci.EntityFramework.Repository.Contracts;
namespace WebApplication1.Models
{
    
    //abstract class
    public abstract class RepositoryNoticeContract: Repository<Notice, BaseEFEntities>,
                                                    IRepository<Notice, BaseEFEntities>
    {
        public RepositoryNoticeContract(BaseEFEntities ctx)
            : base(ctx)
        {
        }
    }

    //class concrete
    public sealed class RepositoryNotice : RepositoryNoticeContract
    {
        public RepositoryNotice(BaseEFEntities ctx) 
            : base(ctx)
        {
        }
    }
}

```

____

__Configure in the Unity:__

```Csharp
public class UnityConfig
{
    
    private static Lazy<IUnityContainer> container = new Lazy<IUnityContainer>(() =>
    {
        var container = new UnityContainer();
        RegisterTypes(container);
        return container;
    });
    
    public static IUnityContainer GetConfiguredContainer()
    {
        return container.Value;
    }
    

    public static void RegisterTypes(IUnityContainer container)
    {
        
        container.RegisterType<BaseEFEntities>();            
        container.RegisterType<RepositoryNoticeContract, RepositoryNotice>();

    }
}

```

____

__Controller__

```Csharp
public class NoticesController : Controller
{
    //Dependency Injection
    public readonly RepositoryNoticeContract repository;    

    public NoticesController(RepositoryNoticeContract repository)
    {

        this.repository = repository;        

    }    

```    

__Completed Controller__

```Csharp
public class NoticesController : Controller
{
    
    public readonly RepositoryNoticeContract repository;

    public readonly RepositoryTagsContract repositoryTags;

    public NoticesController(RepositoryNoticeContract repository, 
                             RepositoryTagsContract repositoryTags)
    {
        this.repository = repository;
        this.repositoryTags = repositoryTags;
    }        
    
    [AcceptVerbs("GET","POST")]
    public async Task<ActionResult> Index(int? page, string filter)
    {

        int rows = 2;
        ViewBag.filter = filter;

        if (!string.IsNullOrEmpty(filter))
        {
            return View(await repository
                .PaginationAsync(x => x.Title.Contains(filter), 
                x => x.Id, (page ?? 1), rows));
        }

        return View(await repository
            .PaginationAsync(x => x.Id, (page ?? 1), rows));
    }

    [HttpGet()]
    public ActionResult Create()
    {

        ViewBag.TagId = new SelectList(repositoryTags.List(x => x.Description), 
                                "Id", "Description");
        return View();

    }

    [HttpPost()]
    public async Task<ActionResult> Create(Notice notice)
    {        

        await repository.AddAsync(notice);

        return RedirectToAction("Index");

    }

    [HttpGet()]
    public async Task<ActionResult> Edit(int id)
    {

        Notice notice = await repository.FindAsync(id);

        ViewBag.TagId = new SelectList(repositoryTags.List(x => x.Description), 
                                "Id", "Description", notice.TagId);

        return View(notice);

    }

    [HttpPost()]
    public async Task<ActionResult> Edit(int id, Notice notice)
    {

        await repository.EditAsync(notice);            
        return RedirectToAction("Index");

    }

    [HttpGet()]
    public async Task<ActionResult> Delete(int id)
    {

        return View(await repository.FindAsync(id));

    }

    [HttpPost()]
    public async Task<ActionResult> Delete(int id, Notice notice)
    {

        await repository.DeleteAsync(id);

        return RedirectToAction("Index");

    }

}

```   

____
__Methods__

_Example class_

```Csharp
public class Tags
{        
    public int Id { get; set; }
    public string Description { get; set; }        
}
```
_Repository of class Tags_

```Csharp
BaseEFContext Context = new BaseEFContext();
RepositoryTagsContract repTags = new RepositoryTags(Context);
``` 
____

####Add

```Csharp
T Add(T Model);        
IEnumerable<T> Add(IEnumerable<T> models);
//NET > 4 (Async Method).
Task<T> AddAsync(T Model);
Task<IEnumerable<T>> AddAsync(IEnumerable<T> models);

```
_Usage:_


```Csharp
Tags tag = new Tags();
tag.Description = "Example";

repTags.Add(tag);
```

```Csharp
Tags tag0 = new Tags();
tag0.Description = "Example 0";

Tags tag1 = new Tags();
tag1.Description = "Example 1";

repTags.Add(new List<Tags>() { tag0, tag1 });
```

____

####Edit

```Csharp
bool Edit(T model);
//NET > 4 (Async Method).
Task<bool> EditAsync(T model);
```

_Usage_

```Csharp
Tags tag = repTags.Find(1);

if (tag != null)
{
    tag.Description = "Example Edit 1";
}

repTags.Edit(tag);
```

Or

```Csharp
Tags tag = repTags.Find(1);

if (tag != null)
{
    tag.Description = "Example Edit 1";
    repTags.Save();
}

```

____

####Delete
```Csharp
bool Delete(T model);
bool Delete(IEnumerable<T> model);
bool Delete(params object[] key);
bool Delete(Expression<Func<T, bool>> where);
//NET > 4 (Async Method).
Task<bool> DeleteAsync(IEnumerable<T> model);
Task<bool> DeleteAsync(T model);
Task<bool> DeleteAsync(params object[] key);
Task<bool> DeleteAsync(Expression<Func<T, bool>> where);
````
_Usage_

```Csharp
Tags tag = repTags.Find(1);

if (tag != null)
{
    repTags.Delete(tag);
}
```
or
```Csharp
repTags.Delete(1);
```
or
```Csharp
IEnumerable<Tags> tagsList = repTags.List(x => x.Id == 1, o => o.Id, false);
repTags.Delete(tagsList);
```
or
```Csharp
repTags.Delete(x => x.Id == 1);
```

____

####Find
```Csharp
T Find(params object[] key);
T Find(Expression<Func<T, bool>> where);
//NET > 4 (Async Method).
Task<T> FindAsync(params object[] key);
Task<T> FindAsync(Expression<Func<T, bool>> where);
```
_Usage_
```Csharp
Tags tag = repTags.Find(1);
```
or
```Csharp
Tags tag1 = repTags.Find(x => x.Id == 1);
```

____

####All
```Csharp
IEnumerable<T> All(bool AsNoTracking = true);
IEnumerable<T> All<Tkey>(Expression<Func<T, bool>> where, 
            Expression<Func<T, Tkey>> orderBy, bool AsNoTracking = true);                    
IList<TResult> All<TResult, Tkey>(Expression<Func<T, bool>> where, 
            Expression<Func<T, Tkey>> orderBy, Expression<Func<T, TResult>> select, 
            bool AsNoTracking = true);

//NET > 4 (Async Method).
Task<IList<T>> AllAsync(bool AsNoTracking = true);
Task<IList<T>> AllAsync<Tkey>(Expression<Func<T, bool>> where, 
            Expression<Func<T, Tkey>> orderBy, bool AsNoTracking = true);
Task<IList<TResult>> AllAsync<TResult, Tkey>(Expression<Func<T, bool>> where, 
            Expression<Func<T, Tkey>> orderBy, Expression<Func<T, TResult>> select, 
            bool AsNoTracking = true);
```
_Usage_
```Csharp
IEnumerable<Tags> tagList = repTags.All(); //or repTags.All(false)
IEnumerable<Tags> tagList = repTags.All(x => x.Id == 1, o => o.Description);
//With viewModel
IEnumerable<ViewModel> viewModelList = repTags.All(
    x => x.Id == 1, 
    o => o.Description, 
    x => new ViewModel() { Id = x.Id, Title = x.Description }
);
```

###List
```
IList<T> List(bool AsNoTracking = true);

IList<T> List<Tkey>(Expression<Func<T, Tkey>> orderBy, bool AsNoTracking = true);

IList<T> List<Tkey>(Expression<Func<T, Tkey>> orderBy, int page, int total = 10, bool AsNoTracking = true);

IList<T> List<Tkey>(Expression<Func<T, bool>> where, Expression<Func<T, Tkey>> orderBy, bool AsNoTracking = true);

IList<T> List<Tkey>(Expression<Func<T, bool>> where, Expression<Func<T, Tkey>> orderBy, int page, int total = 10, 
        bool AsNoTracking = true);

IList<TResult> List<TResult, Tkey>(Expression<Func<T, Tkey>> orderBy, 
        Expression<Func<T, TResult>> select, bool AsNoTracking = true);

IList<TResult> List<TResult, Tkey>(Expression<Func<T, bool>> where, 
        Expression<Func<T, Tkey>> orderBy, Expression<Func<T, TResult>> select, bool AsNoTracking = true);

IList<TResult> List<TResult, Tkey>(Expression<Func<T, bool>> where, 
        Expression<Func<T, Tkey>> orderBy, Expression<Func<T, TResult>> select, int page, int total = 10, 
        bool AsNoTracking = true);

//NET > 4 (Async Method).
Task<IList<T>> ListAsync(bool AsNoTracking = true);

Task<IList<T>> ListAsync<Tkey>(Expression<Func<T, Tkey>> orderBy, bool AsNoTracking = true);

Task<IList<T>> ListAsync<Tkey>(Expression<Func<T, Tkey>> orderBy, int page, int total = 10,bool AsNoTracking = true);

Task<IList<T>> ListAsync<Tkey>(Expression<Func<T, bool>> where, Expression<Func<T, Tkey>> orderBy, bool AsNoTracking = true);

Task<IList<T>> ListAsync<Tkey>(Expression<Func<T, bool>> where, Expression<Func<T, Tkey>> orderBy, int page, int total = 10, 
        bool AsNoTracking = true);

Task<IList<TResult>> ListAsync<TResult, Tkey>(Expression<Func<T, Tkey>> orderBy, 
        Expression<Func<T, TResult>> select, bool AsNoTracking = true);

Task<IList<TResult>> ListAsync<TResult, Tkey>(Expression<Func<T, bool>> where, 
        Expression<Func<T, Tkey>> orderBy, Expression<Func<T, TResult>> select, bool AsNoTracking = true);

Task<IList<TResult>> ListAsync<TResult, Tkey>(Expression<Func<T, bool>> where, 
        Expression<Func<T, Tkey>> orderBy, Expression<Func<T, TResult>> select, int page, int total = 10, 
        bool AsNoTracking = true);
```
_Usage_
```Csharp
IList<Tags> tagList = repTags.List(); //or repTags.List(false);

IList<Tags> tagList = repTags.List(o => o.Description);            

IList<Tags> tagList = repTags.List(x => x.Id == 1, o => o.Description);

IList<Tags> tagList = repTags.List(o => o.Description, 1, 10);

IList<Tags> tagList = repTags.List(x => x.Id == 1, o => o.Description, 1, 10);

IList<ViewModel> tagListViewModel = repTags.List(o => o.Description, 
            x => new ViewModel() { Id = x.Id, Title = x.Description });

IList<ViewModel> tagListViewModel = repTags.List(x => x.Id == 1, o => o.Description, 
            x => new ViewModel() { Id = x.Id, Title = x.Description });

IList<ViewModel> tagListViewModel = repTags.List(x => x.Id == 1, o => o.Description, 
            x => new ViewModel() { Id = x.Id, Title = x.Description }, 1, 10);

```

____

####Pagination
```Csharp
IPagedList<T> Pagination<TOrderBy>(Expression<Func<T, TOrderBy>> orderBy, int page, int total = 10);

IPagedList<TResult> Pagination<TResult, TOrderBy>(Expression<Func<T, TOrderBy>> orderBy, Expression<Func<T, TResult>> select,
             int page, int total = 10);

IPagedList<T> Pagination<TOrderBy>(Expression<Func<T, bool>> where, Expression<Func<T, TOrderBy>> orderBy, int page,
            int total = 10);

IPagedList<TResult> Pagination<TResult, TOrderBy>(Expression<Func<T, bool>> where, 
            Expression<Func<T, TOrderBy>> orderBy, Expression<Func<T, TResult>> select, int page, int total = 10);

//NET > 4 (Async Method).
Task<IPagedList<T>> PaginationAsync<TOrderBy>(Expression<Func<T, TOrderBy>> orderBy, int page, int total = 10);

Task<IPagedList<TResult>> PaginationAsync<TResult, TOrderBy>(Expression<Func<T, TOrderBy>> orderBy,
            Expression<Func<T, TResult>> select, int page, int total = 10);

Task<IPagedList<T>> PaginationAsync<TOrderBy>(Expression<Func<T, bool>> where, Expression<Func<T, TOrderBy>> orderBy, 
            int page, int total = 10);

Task<IPagedList<TResult>> PaginationAsync<TResult, TOrderBy>(Expression<Func<T, bool>> where, 
            Expression<Func<T, TOrderBy>> orderBy, Expression<Func<T, TResult>> select, int page, int total = 10);
#endif
```
_Usage_
```Csharp
IPagedList<Tags> Pagination = repTags.Pagination(o => o.Id, 1, 10);

IPagedList<Tags> Pagination = repTags.Pagination(x => x.Id == 1, o => o.Id, 1, 10);

IPagedList<ViewModel> Pagination = repTags.Pagination(o => o.Id, x => 
                        new ViewModel() { Id = x.Id, Title = x.Description }, 1, 10);

IPagedList<ViewModel> Pagination = repTags.Pagination(x => x.Id == 1, o => o.Id, x => 
                        new ViewModel() { Id = x.Id, Title = x.Description }, 1, 10);
```