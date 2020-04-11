# ABP First Code

### 创建实体类

ABP框架提供了一个定义了**Id**属性的**Entity**类（泛型Entity<T>）。

在领域层中创建Entities文件夹，添加实体类。

### 创建DbContext

在EF中的XXDbContext文件中添加IDbset。

### 创建数据库迁移

`Add-Migration`命令创建新的迁移类和`Update-Database`命令更新数据库

### 定义仓储

ABP使用泛型`IRepository`接口为每个实体创建了一个自动的仓储，定义了select、insert、getall等一些通用方法。

如果默认的仓储方法无法实现功能，则自己在Core项目层中创建一个**IRepositories**文件夹，存放所有的实体仓储接口。

接口扩展了ABP框架的泛型IRepository接口，因此，ICityRepository默认继承定义了所有这些方法

```c#
   public interface ICityRepository:IRepository<Cities>
    {
        List<Cities> GetCitiesWithProvince(string provinceCode);
    }
```

### 实现仓储

实现这些仓储接口是在基础设施层即**EntityFramework**层,在基础设施层中名为“Repositories”的文件夹存放实现仓储接口的类，ABP模板已经帮我们创建了一个仓储基类`ChargeStationRepositoryBase`，以后如果我们要给仓储实现添加公共的方法时，就可以直接添加到该基类中。

在**Repositories**文件夹里定义ICityRepository的实现类CityRepository

```c#
public class CityRepository:ChargeStationRepositoryBase<Cities>,ICityRepository
{
    public CityRepository(IDbContextProvider<ChargeStationDbContext> dbContextProvider):base(dbContextProvider)
    {
        
    }
    public List<Cities> GetCitiesWithProvince(string provinceCode)
    {
        var query = GetAll();//GetAll()返回一个IQueryable<T>，我们可以通过它来查询
          var query2 = Context.Cities.AsQueryable();//也可以直接使用EF的DbContext对象
          var query3 = Table.AsQueryable();//另一种选择：直接使用Table属性代替"Context.Cities"，都是一样的。
          if (!string.IsNullOrEmpty(provinceCode))
        {
            query = query.Where(c => c.ProvinceCode == provinceCode);
        }
        return query.ToList();
    }
      public async Task<List<Cities>> GetCitiesWithProvinceAsync(string provinceCode)
        {
            return await GetAllListAsync(c => c.ProvinceCode == provinceCode);
        }
}
```


CityRepository继承了泛型的`ChargeStationRepositoryBase`类，而且实现了仓储接口。这里要显示声明实现类的有参数的构造函数，使用泛型的`IDbContextProvider`将数据库上下文的子类`ChargeStationContex`t传给父类的构造函数。

`GetCitiesWithProvince`方法是我们实现仓储接口ICityRepository中的方法，也是我们自己特殊需要的方法。

在仓储中，我们可以自由地使用Context（EF的DbContext）对象和数据库。ABP框架为我们管理数据库连接，事务，创建和释放DbContext，因而不用我们自己处理了。

`GetAll()`方法返回类型`IQueryable<T>`和`GetAllList()`方法返回`List<T>`的区别：

IQueryable<T>:

Repository对象以外调用`GetAll`这个方法会开启数据库连接。`IQueryable<T>`允许**延迟**执行,只有使用`ToList()`、`foreach()`迭代时,才会**实际**执行数据库的查询。我们可以使用ABP所提供的UnitOfWork特性在调用的方法上来实现。注意,Application Service方法预设都已经是UnitOfWork。因此,使用了GetAll方法就不需要如同Application Service的方法上添加UnitOfWork特性。

List<T>:

返回的该结果说明已经执行了数据库查询或者已经从内存中取出了数据（如果内存中有的话）。

### 构建应用层服务

将每一个实体对应的服务层的东西创建在一个文件夹中，命名规则“实体类名复数+App”，包含Dtos文件夹（存放于展现层交互的dto）、接口、方法。

![image-20200411114301571](C:\Users\chenjianlin.LONSID\AppData\Roaming\Typora\typora-user-images\image-20200411114301571.png)

ABP可以自动帮助我们进行数据校验。当然，我们也可以添加数据注解进行校验。校验之后，还可以实现IShouldNormalize接口来设置缺省值。

```c#
public class CreateCityInput : IInputDto, IShouldNormalize
    {
        [Required]
        public string Name { get; set; }
        [Required]
        public string Code { get; set; }
        [Required]
        public string ProvinceCode { get; set; }
        public DateTime UpdatedTime { get; set; }
        public string UpdatedBy { get; set; }

        public void Normalize()
        {
            if (UpdatedTime==null)
            {
                UpdatedTime=DateTime.Now;
            }
        }
    }
}
```

定义一个服务接口IXXAppService，命名规范是”I+实体类单数+AppService"

接下来实现应用服务接口