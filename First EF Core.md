# First EF Core

### 新建

##### 创建实体类Student，添加required、字段长度等特性

```c#
 public class Student
    {
        public int Id { get; set; }
        [Display(Name ="姓名")]
        [Required(ErrorMessage ="请输入名字"),MaxLength(50,ErrorMessage ="名字太长")]
        public string  Name { get; set; }
        [Display(Name ="年级信息")]
        [Required]
        public ClassNameEnum? ClassName { get; set; }
        [Display(Name ="邮箱")]
        [Required]
        public string  Email { get; set; }


}
```



##### 创建`项目XXDbContext`类，继承`DbContext`类。

构造函数需继承基类，传递配置信息

`public ` BPMDbContext(DbContextOptions<BPMDbContext> options)`
            `: base(options)`
        `{`
        }`

添加Dbset<TEntity>属性

`public DbSet<Student> Students { get; set; }`

##### 在Startup中的ConfigureServices中AddContextPool

```c#
        services.AddDbContextPool<AppDbContext>(
            optionsAction: options => options.UseSqlServer(_configuration.GetConnectionString("StudentDBConnection"))
            );
```


##### 在 程序包管理控制台中输入`Add-Migration，Update-Database`等(about_EntityFrameworkCore 命令help)



### 添加种子数据

创建静态类`ModelBuilderExtesions`，在`ModelBuilderExtesions`中添加

    public static class ModelBuilderExtesions
    {
        public static void Seed(this ModelBuilder modelBuilder)
        {
            modelBuilder.Entity<Student>().HasData(
    new Student
    {
        Id = 1,
        Name = "amon",
        ClassName = ClassNameEnum.Third,
        Email = "chen@lonsid.cn"
    },
    new Student
    {
        Id = 2,
        Name = "chenjianlin",
        ClassName = ClassNameEnum.First,
        Email = "cyj@lonsid.cn"
    }
    );
        }
    }
在`DbContext`中添加

```c#
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
       modelBuilder.Seed();
    }
```
### 同步领域模型与数据库结构

##### 回滚

`Update-Database`要移除的已应用到数据库的迁移的上一步迁移名称

`Remove-Migration` 移除最后一次迁移