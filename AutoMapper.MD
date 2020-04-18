# 介绍

AutoMapper 是个数据映射类库，具本作用如下：

> AutoMapper的作用是把一个对象转化为另一个对象，避免每次都去转化。

> 使用DTO实现表现层与领域Model的解耦，用AutoMapper来实现DTO与领域Model的相互转换
  
## 什么是DTO？
> DTO（Data Transfer Object）就是数据传输对象，说白了就是一个对象，只不过里边全是数据而已。

## 使用场景

> API接口输入输出

> ORM数据转换  

# 官方

官网 [Automapper.org](https://automapper.org)

源码 [Github](https://github.com/AutoMapper/AutoMapper)

文档 [Docs](https://docs.automapper.org/en/stable/)

版本：v9.0.0

# 使用

## 引用
~~~
Install-Package AutoMapper
Install-Package AutoMapper.Extensions.Microsoft.DependencyInjection
~~~

## 案例
*********************************************************
### 数据转换 - 场景一

ConfigureServices 中绑定
~~~
public void ConfigureServices(IServiceCollection services)
{
    services.AddAutoMapperSetup();

    // ...
}
~~~

启动服务
~~~
public static class AutoMapperSetup
{
    public static void AddAutoMapperSetup(this IServiceCollection services)
    {
        if (services == null) throw new ArgumentNullException(nameof(services));

        services.AddAutoMapper(typeof(AutoMapperConfig));
        AutoMapperConfig.RegisterMappings();
    }
}
~~~

绑定配置文件
~~~
public class AutoMapperConfig
{
    public static MapperConfiguration RegisterMappings()
    {
        return new MapperConfiguration(cfg =>
        {
            cfg.AddProfile(new CustomProfile());
        });
    }
}
~~~

数据配置
~~~
public class CustomProfile : Profile
{
    /// <summary>
    /// 配置构造函数，用来创建关系映射
    /// </summary>
    public CustomProfile()
    {
        CreateMap<Person, People>();

        // 类型字段系统
        CreateMap<User, UserDto>();

        // 扁平化映射
        CreateMap<Product, ProductDto>();

        // 投影
        CreateMap<Book, BookDto>()
            .ForMember(desc => desc.dddd, opt => opt.MapFrom(src => src.BuyTime.Ticks)) // 类型自动转换
            .ForMember(desc => desc.UserXX, opt => opt.MapFrom(src => new UserXX() { AgeX = src.BuyTime.Second, NameX = "fff" }))
            .ForMember(desc => desc.Date, opt => opt.MapFrom(src => src.BuyTime.Date))
            .ForMember(desc => desc.Hour, opt => opt.MapFrom(src => 4)) // 设默认值
            .ForMember(desc => desc.Minute, opt => opt.MapFrom(src => src.BuyTime.Minute));

        // 条件映射 - 
        CreateMap<Azi, UserDto>().ForMember(
            desc => desc.Age,
            opt => opt.Condition(src => src.Age < 21) // Condition - 为条件
        );

        // 可设置默认值
        CreateMap<Source, Destination>().ForMember(desc => desc.like, opt => opt.MapFrom(src => "azi"));

        // 可忽视多余值
        CreateMap<Destination, Source>().ForMember(desc => desc.Width, opt => opt.Ignore()) // 忽略字段Width，不赋值
            .ForMember(desc => desc.like2, opt => opt.MapFrom(src => src.like));

        // 空值替换 - 对象必需为null时才能转换
        CreateMap<Source, Destination>().ForMember(desc => desc.Name, opt => opt.NullSubstitute("azi"));

        // 其他设置  - 没测试，使用场景不多
        CreateMap<User, Azi>()
            .BeforeMap((src, desc) => src.Age += 50)
            .AfterMap((src, desc) => desc.Age -= 35) // 设置转换前后的行为
            .ReverseMap(); // 设置反向映射
    }
}
~~~

实体类
~~~
public class Person
{
    public int Age { set; get; }
    public string Name { set; get; }
    public decimal Salary { set; get; }
    public bool Sex { set; get; }
    public DateTime Birthday { set; get; }
}
public class People
{
    public int Age { set; get; }
    public string Name { set; get; }
    public decimal Salary { set; get; }
    public bool Sex { set; get; }
    public DateTime Birthday { set; get; }
}

public class User
{
    public int Id { set; get; }
    public string Name { set; get; }
    public int Age { set; get; }
    public User(int i)
    {
        Id = i;
        Name = "muphy-" + i;
        Age = 15 + i;
    }

    public User()
    {
        Id = 1;
        Name = "muphy";
        Age = 15;
    }

    public override string ToString()
    {
        return "[Name:" + Name + ",Age:" + Age + "]";
    }
}

public class UserDto
{
    public string Name { set; get; }
    public int Age { set; get; }

    public override string ToString()
    {
        return "[Name:" + Name + ",Age:" + Age + "]";
    }
}

public class Azi : User
{
    public string Like { set; get; }
    public Azi(int i) : base(i) { Like = "QiaoFeng"; }
    public Azi() { }

    public override string ToString()
    {
        return "[Name:" + Name + ",Age:" + Age + "]";
    }
}

public class Product
{
    public Supplier Supplier { set; get; }
    public decimal GetPrice()
    {
        return 10;
    }

    public Product()
    {
        Supplier = new Supplier();
    }
}

public class Supplier
{
    public string Name { get; set; }
    public Supplier()
    {
        Name = "伟创力";
    }
}

public class ProductDto
{
    public string SupplierName { set; get; }
    public decimal Price { set; get; }
    public override string ToString()
    {
        return "[Price:" + Price + ",SupplierName:" + SupplierName + "]";
    }
}

public class Book : Product
{
    public DateTime BuyTime { get { return DateTime.Now; } }
}

public class BookDto : ProductDto
{
    public DateTime Date { get; set; }
    public int Hour { get; set; }
    public int Minute { get; set; }

    public string dddd { get; set; }

    public UserXX UserXX { get; set; }

    public override string ToString()
    {
        return "[Price:" + Price + ",SupplierName:" + SupplierName + ",Date:" + Date + "]";
    }
}

public class UserXX
{
    public string NameX { set; get; }
    public int AgeX { set; get; }
}

public class Source
{
    public string like2;

    public Source(bool b = false)
    {
        if (b == true) { Name = ""; }
    }

    public string Name { get; set; } = "muphy";
    public int Width { set; get; }
}

public class Destination
{
    public string Name { get; set; }
    public string like { get; set; }

    public override string ToString()
    {
        return Name + " Like " + like;
    }
}

public class Foo
{
    public int Bar { get; set; }
    public int Baz { get; set; }
}
~~~

数据转换，ApiController接口中使用
~~~
[ApiController]
[Route("[controller]")]
public class TestController : ControllerBase
{
    private readonly IMapper mapper;

    public TestController(IMapper mapper)
    {
        this.mapper = mapper; 
    }

    [HttpGet]
    public object Get()
    {    
        // 对象
        Person p = new Person() { Age = 12, Name = "欧阳锋", Birthday = DateTime.Now, Sex = true, Salary = 10000 };
        People peo = new People();//不能为null
        // 转换
        mapper.Map(p, peo);
        // 调用
        Console.WriteLine(peo.Name);

        // 数据转换
        UserDto dto = mapper.Map<UserDto>(new User(1)); // 常使用

        // 数据转换
        var obj0 = mapper.Map<Person>(p); // 常使用

        // 扁平化映射，也叫智能映射
        var obj1 = mapper.Map<ProductDto>(new Product());

        // 投影，字段指定字段赋值（常使用）
        var book = new Book();
        var obj2 = mapper.Map<BookDto>(book); // 可以设默认值（指定的值）

        // 投条投影, 条件为：Age < 21 则赋值
        var aziA = new Azi(10); // Age = 15 + 10 = 25
        var aziB = new Azi(3);  // Age = 15 + 3 = 18
        var obj3 = mapper.Map<UserDto>(aziA); // 25 - 没达到条件
        var obj4 = mapper.Map<UserDto>(aziB); // 18 - 达到条件

        var source = new Source() { Width = 123 };
        var obj5 = mapper.Map<Destination>(source);
        var source1 = new Source(true) { Width = 123 };
        var obj6 = mapper.Map<Destination>(source1); // 对象必需为null时才能转换 

        var obj7 = mapper.Map<Source>(new Destination() { like = "ddff11", Name = "443" }); // 忽略某个字段

        // 泛型
        dynamic foo = new ExpandoObject();
        foo.Bar = 5;
        foo.Baz = 6;

        // 以下的写法会报错
        // var foo = new { Bar = 12, Baz = 23 };

        var mapperX = new MapperConfiguration(c => { }).CreateMapper();
        var result = mapperX.Map<Foo>(foo);
        result.Bar = 500;
        result.Baz = 222;
        dynamic foo2 = mapperX.Map<ExpandoObject>(result);
        Console.WriteLine(foo2.Bar);

        Console.ReadKey();
    }
}
~~~

### 数据转换 - 场景二
*********************************************************
直接初始化后使用，建议在实体类中定义这个方法
~~~
public V Map<T, V>(T source)
{
    var mapper = new MapperConfiguration(c => c.CreateMap<T, V>()).CreateMapper();
    return mapper.Map<V>(source);
}
~~~

接口中使用
~~~
[HttpGet]
public object Get()
{    
    // 对象
    Person p = new Person() { Age = 12, Name = "欧阳锋", Birthday = DateTime.Now, Sex = true, Salary = 10000 };
    People peo = Map<Person, People>(p);
}
~~~

### 验证机制 AssertConfigurationIsValid
*********************************************************
> 开始验证机制后，如果两个实体类字段不同，则抛出异常。 

> 如果指定字段，或使用Ignore（），不会抛出异常。

> 程序发布后执行，验证机制会失效（官方文档），具体没有测试过。

~~~
class Program
{
    static void Main(string[] args)
    {
        var config = new MapperConfiguration(cfg =>cfg.CreateMap<Source, Destination>());
        config.AssertConfigurationIsValid(); // 使用验证机制
        Console.ReadKey();
    }
}
public class Source
{
    public int SomeValue { get; set; }
}
public class Destination
{
    public int SomeValuefff { get; set; }
}
~~~

跳过验证
~~~
// 忽略某个字段
var config  = new MapperConfiguration(cfg =>
    {
        // cfg.CreateMap<Source, Destination>(MemberList.Source); // 只验证源（所有源）
        cfg.CreateMap<Source2, Destination2>(MemberList.None);    // 跳过验证
        // 此外还有一个 MemberList.Destination 表示验证目标对象所有字段都要配置
    }
);

// 配置匹配验证
config.AssertConfigurationIsValid();
~~~

### 反转映射 ReverseMap
*********************************************************
实体A与实体B可以相互转换
~~~
var person = Mapper.Map<Person>(new Person() { aaa = "11111"});
public class Order {
  public decimal Total { get; set; }
  public Customer Customer { get; set; }
}
public class Customer {
  public string Name { get; set; }
}
~~~

可以把它变成DTO
~~~
public class OrderDto {
  public decimal Total { get; set; }
  public string CustomerName { get; set; }
}
~~~

可以映射两个方向，包括unflattening
~~~
var mapper = new MapperConfiguration(cfg => {
    cfg.CreateMap<Order, OrderDto>()
       .ReverseMap(); // 加了此项，可以反向转换
}).CreateMapper();
~~~

通过调用ReverseMap，AutoMapper创建一个反向映射配置，包括unflattening
~~~
var customer = new Customer
{
    Name = "Bob"
};
var order = new Order
{
    Customer = customer,
    Total = 15.8m
};
var orderDto = mapper.Map<Order, OrderDto>(order);
orderDto.CustomerName = "Joe";
mapper.Map(orderDto, order); // 反向转换
Console.WriteLine(order.Customer.Name); // 输出Joe
~~~

Unflattening仅配置为ReverseMap。如果你想要松散，你必须配置Entity- > Dto然后调用ReverseMap从Dto- > 创建一个unflattening类型的地图配置Entity。

### 列表与数组
*********************************************************
AutoMapper 只需要配置元素类型，而不是任何可能使用的数组或列表类型。例如，我们可能有一个简单的源和目标类型
~~~
public class Source
{
    public int Value { get; set; }
}
public class Destination
{
    public int Value { get; set; }
}
~~~

支持所有基本通用集合类型
~~~
var config = new MapperConfiguration(cfg => cfg.CreateMap<Source, Destination>()); // 绑定
var sources = new[]
{
    new Source { Value = 5 },
    new Source { Value = 6 },
    new Source { Value = 7 }
};

// 以下五种数组都可以转换
IEnumerable<Destination> ienumerableDest = Mapper.Map<Source[], IEnumerable<Destination>>(sources);
ICollection<Destination> icollectionDest = Mapper.Map<Source[], ICollection<Destination>>(sources);
IList<Destination> ilistDest = Mapper.Map<Source[], IList<Destination>>(sources);
List<Destination> listDest = Mapper.Map<Source[], List<Destination>>(sources);
Destination[] arrayDest = Mapper.Map<Source[], Destination[]>(sources);
~~~

支持的源集合类型
~~~
IEnumerable
IEnumerable<T>
ICollection
ICollection<T>
IList
IList<T>
List<T>
Arrays
~~~

处理空集合
> 映射集合属性时，如果源值为null，AutoMapper 会将目标字段映射到空集合，而不是将目标值设置为null。这符合实体框架和框架设计指南的行为，它们认为C＃引用，数组，列表，集合，字典和 IEnumerables 永远不应该为null。

> AllowNullCollections 配置映射器时，可以通过将该属性设置为true 来更改此行为。
~~~
var config = new MapperConfiguration(cfg => {
    cfg.AllowNullCollections = true;
    cfg.CreateMap<Source, Destination>();
});
~~~

集合中的多态元素类型 
> 很多时候，我们的源类型和目标类型都可能有类型层次结构。AutoMapper支持多态数组和集合，如果找到则使用派生的源/目标类型。
~~~
public class ParentSource
{
    public int Value1 { get; set; }
}
public class ChildSource : ParentSource
{
    public int Value2 { get; set; }
}
public class ParentDestination
{
    public int Value1 { get; set; }
}
public class ChildDestination : ParentDestination
{
    public int Value2 { get; set; }
}
~~~

AutoMapper 仍然需要对子映射进行显式配置，因为AutoMapper无法“猜测”要使用的特定子目标映射。以下是上述类型的示例
~~~
var mapper = new MapperConfiguration(c => {
    c.CreateMap<ParentSource, ParentDestination>() // 绑定父类类型
        .Include<ChildSource, ChildDestination>(); // && 绑定字类类型
    c.CreateMap<ChildSource, ChildDestination>();  // 绑定类型
}).CreateMapper();
var sources = new[]
    {
        new ParentSource(),
        new ChildSource(),
        new ParentSource()
    };
var destinations = mapper.Map<ParentSource[], ParentDestination[]>(sources);
~~~

### 嵌套映射
*********************************************************
当映射引擎执行映射时，它可以使用各种方法之一来解析目标成员值。其中一种方法是使用另一种类型映射，其中源成员类型和目标成员类型也在映射配置中配置。

这使我们不仅可以展平我们的源类型，还可以创建复杂的目标类型。例如，我们的源类型可能包含另一种复杂类型。
~~~
public class OuterSource
{
    public int Value { get; set; }
    public InnerSource Inner { get; set; }
}
public class InnerSource
{
    public int OtherValue { get; set; }
}
~~~

我们可以简单地将 OuterSource.Inner.OtherValue 展平为一个 InnerOtherValue 属性，但我们可能还想为 Inner 属性创建相应的复杂类型
~~~
public class OuterDest
{
    public int Value { get; set; }
    public InnerDest Inner { get; set; }
}
public class InnerDest
{
    public int OtherValue { get; set; }
}
~~~

在这种情况下，我们需要配置其他源/目标类型映射
~~~
var config = new MapperConfiguration(cfg => {
    cfg.CreateMap<OuterSource, OuterDest>(); // 绑定父类
    cfg.CreateMap<InnerSource, InnerDest>(); // 绑定子类
});
config.AssertConfigurationIsValid(); // 开启验证机制
var source = new OuterSource
{
    Value = 5,
    Inner = new InnerSource { OtherValue = 15 }
};
var mapper = config.CreateMapper();
var dest = mapper.Map<OuterSource, OuterDest>(source);
// dest.Value == 5;
// dest.Inner==null;
// dest.Inner.OtherValue == 15;
~~~

* 配置类型的顺序无关紧要
* 调用Map不需要指定任何内部类型映射，只需要指定用于传入源值的类型映射

### 自定义类型转换器
*********************************************************
> 有时，您需要完全控制一种类型到另一种类型的转换。这通常是当一种类型看起来与另一种类型不同时，转换函数已经存在，并且您希望从“更宽松”类型转变为更强类型，例如字符串的源类型到目标类型Int32。

源类型
~~~
public class Source
{
    public string Value1 { get; set; }
    public string Value2 { get; set; }
    public string Value3 { get; set; }
}
~~~

映射到
~~~
public class Destination
{
    public int Value1 { get; set; }
    public DateTime Value2 { get; set; }
    public Type Value3 { get; set; }
}
~~~

如果我们尝试按原样映射这两种类型，AutoMapper 会抛出异常（在映射时和配置检查时），因为 AutoMapper 不知道从字符串到 int，DateTime 或 Type 的任何映射。要为这些类型创建地图，我们必须提供自定义类型转换器，我们有三种方法
~~~
void ConvertUsing(Func<TSource, TDestination> mappingFunction);
void ConvertUsing(ITypeConverter<TSource, TDestination> converter);
void ConvertUsing<TTypeConverter>() where TTypeConverter : ITypeConverter<TSource, TDestination>;
~~~

第一个选项是任何获取源并返回目标的函数（也有几个重载）。这适用于简单的情况，但对于较大的情况则变得难以处理。在更困难的情况下，我们可以创建自定义ITypeConverter <TSource，TDestination>
~~~
public interface ITypeConverter<in TSource, TDestination>
{
    TDestination Convert(TSource source, TDestination destination, ResolutionContext context);
}
~~~

并为 AutoMapper 提供自定义类型转换器的实例，或者只提供 AutoMapper 将在运行时实例化的类型。然后，我们上面的源/目标类型的映射配置变为
~~~
[Test]
public void Example()
{
    var config = new MapperConfiguration(cfg => {
      cfg.CreateMap<string, int>().ConvertUsing(s => Convert.ToInt32(s));
      cfg.CreateMap<string, DateTime>().ConvertUsing(new DateTimeTypeConverter()); // 转换
      cfg.CreateMap<string, Type>().ConvertUsing<TypeTypeConverter>(); // 绑定泛型类转换
      cfg.CreateMap<Source, Destination>();
    });
    config.AssertConfigurationIsValid();
    var source = new Source
    {
        Value1 = "5",
        Value2 = "01/01/2000",
        Value3 = "AutoMapperSamples.GlobalTypeConverters.GlobalTypeConverters+Destination"
    };
    var mapper = config.CreateMapper();
    Destination result = mapper.Map<Source, Destination>(source);
    result.Value3.ShouldEqual(typeof(Destination));
}
public class DateTimeTypeConverter : ITypeConverter<string, DateTime>
{
    public DateTime Convert(string source, DateTime destination, ResolutionContext context)
    {
        return System.Convert.ToDateTime(source);
    }
}
public class TypeTypeConverter : ITypeConverter<string, Type>
{
    public Type Convert(string source, Type destination, ResolutionContext context)
    {
          return Assembly.GetExecutingAssembly().GetType(source);
    }
}
~~~

> 在第一个映射中，从string到Int32，我们只使用内置的Convert.ToInt32函数（作为方法组提供）。接下来的两个使用自定义ITypeConverter实现。

> 自定义类型转换器的真正强大之处在于，AutoMapper 可以在任何映射类型上找到源/目标对时使用它们。我们可以构建一组自定义类型转换器，其他映射配置使用它们，而无需任何额外配置。在上面的示例中，我们永远不必再次指定字符串/ int转换。如果必须在类型成员级别配置自定义值解析器，则自定义类型转换器的范围是全局的。

### 投影 Projection
*********************************************************
投影（Projection）将源转换为目标，而不是展平对象模型。如果没有额外的配置，AutoMapper需要一个展平的目标来匹配源类型的命名结构。如果要将源值投影到与源结构不完全匹配的目标中，则必须指定自定义成员映射定义。例如，我们可能想要转换此源结构
~~~
public class CalendarEvent
{
    public DateTime Date { get; set; }
    public string Title { get; set; }
}
public class CalendarEventForm
{
    public DateTime EventDate { get; set; }
    public int EventHour { get; set; }
    public int EventMinute { get; set; }
    public string Title { get; set; }
}
~~~

因为目标属性的名称与源属性（CalendarEvent.Date需要CalendarEventForm.EventDate）不完全匹配，所以我们需要在类型映射配置中指定自定义成员映射
~~~
// 实体
var calendarEvent = new CalendarEvent
{
    Date = new DateTime(2008, 12, 15, 20, 30, 0),
    Title = "Company Holiday Party"
};

// 配置 AutoMapper
var mapper = new MapperConfiguration(cfg =>
    cfg.CreateMap<CalendarEvent, CalendarEventForm>()
    .ForMember(dest => dest.EventDate, opt => opt.MapFrom(src => src.Date.Date))
    .ForMember(dest => dest.EventHour, opt => opt.MapFrom(src => src.Date.Hour))
    .ForMember(dest => dest.EventMinute, opt => opt.MapFrom(src => src.Date.Minute))).CreateMapper();

// 转换 mapping
CalendarEventForm form = mapper.Map<CalendarEvent, CalendarEventForm>(calendarEvent);
form.EventDate.Equals(new DateTime(2008, 12, 15));
form.EventHour.Equals(20);
form.EventMinute.Equals(30);
form.Title.Equals("Company Holiday Party");
~~~

> 每个自定义成员配置都使用操作委托来配置每个单独的成员。在上面的示例中，我们使用该MapFrom选项来执行自定义源到目标成员映射。该MapFrom方法将lambda表达式作为参数，稍后在映射期间对其进行求值。所述MapFrom表达可以是任何lambda表达式。Func<TSource, object>

### 自定义值解析器
*********************************************************
虽然 AutoMapper 涵盖了相当多的目标成员映射方案，但是有1到5％的目标值需要一些帮助才能解析。很多时候，这个自定义值解析逻辑是域逻辑，可以直接在我们的域上。但是，如果此逻辑仅适用于映射操作，则会使我们的源类型混乱并产生不必要的行为。在这些情况下，AutoMapper 允许为目标成员配置自定义值解析器。例如，我们可能希望在映射期间获得计算值
~~~
public class Source
{
    public int Value1 { get; set; }
    public int Value2 { get; set; }
}
public class Destination
{
    public int Total { get; set; }
}
~~~

无论出于何种原因，我们希望Total为源Value属性的总和。由于某些其他原因，我们不能或不应该将此逻辑放在我们的Source类型上。要提供自定义值解析器，我们需要首先创建一个实现IValueResolver的类型
~~~
public interface IValueResolver<in TSource, in TDestination, TDestMember>
{
    TDestMember Resolve(TSource source, TDestination destination, TDestMember destMember, ResolutionContext context);
}

// ResolutionContext 包含当前解析操作的所有上下文信息，例如源类型，目标类型，源值等。一个示例实现
public class CustomResolver : IValueResolver<Source, Destination, int>
{
    public int Resolve(Source source, Destination destination, int member, ResolutionContext context)
    {
        return source.Value1 + source.Value2;
    }
}
~~~

一旦我们有了 IValueResolver 实现，我们就需要告诉 AutoMapper 在解析特定目标成员时使用这个自定义值解析器。我们有几种方法可以告诉 AutoMapper 使用自定义值解析器，包括以下三种
~~~
MapFrom<TValueResolver>
MapFrom(typeof(CustomValueResolver))
MapFrom(aValueResolverInstance)
~~~

在下面的示例中，我们将使用第一个选项，通过泛型告诉AutoMapper自定义解析器类型
~~~
var config = new MapperConfiguration(cfg => cfg.CreateMap<Source, Destination>()
     .ForMember(dest => dest.Total, opt => opt.MapFrom<CustomResolver>()));
config.AssertConfigurationIsValid();
var source = new Source
{
    Value1 = 5,
    Value2 = 7
};
var mapper = config.CreateMapper();
var result = mapper.Map<Source, Destination>(source);
result.Total.ShouldEqual(12);
~~~

### 空值替换
*********************************************************
如果源值在成员链的任何位置为null，则空替换允许您为目标成员提供备用值。这意味着它不是从null映射，而是从您提供的值映射。
就是如果源里面为null 则用默认值替换。如果不为空 那就是不为空的值。
~~~
class Program
{
    static void Main(string[] args)
    {
        var config = new MapperConfiguration(cfg => cfg.CreateMap<Source, Destination>()
            .ForMember(destination => destination.Value2, opt => opt.NullSubstitute("Other Value")));
        var mapper = config.CreateMapper();
        var source = new Source { Value2 = null };
        var dest = mapper.Map<Source, Destination>(source);
        source.Value2 = "Not null";
        dest = mapper.Map<Source, Destination>(source);
        Console.ReadKey();
    }
}
public class Source
{
    public string Value2 { get; set; }
}
public class Destination
{
    public string Value2 { get; set; }
}
~~~

### 匹配执行之前和之后
*********************************************************
有时，您可能需要在映射发生之前或之后执行自定义逻辑。这些应该是罕见的，因为在AutoMapper之外进行这项工作更为明显。您可以创建全局之前 / 之后的地图操作。
~~~
var mapper = new MapperConfiguration(cfg => {
  cfg.CreateMap<Source, Dest>()
    .BeforeMap((src, dest) => src.Value = src.Value + 10)
    .AfterMap((src, dest) => dest.Value = "John");
}).CreateMapper();
~~~

或者您可以在映射期间创建地图回调之前 / 之后。
~~~
class Program
{
    static void Main(string[] args)
    {
        var mapper = new MapperConfiguration(cfg => {
            cfg.CreateMap<Source, Dest>()
                .BeforeMap((src, dest) => src.Value = "前缀_" + src.Value )
                .AfterMap((src, dest) => dest.Value = dest.Value + "_后缀");
        }).CreateMapper();
        var s = new Source() { Value = "程序小儿"};
        var d = mapper.Map<Dest>(s); // d 打印出："前缀_程序小儿_后缀"
        Console.ReadKey();
    }
}
public class Source
{
    public string Value { get; set; }
}
public class Dest
{
    public string Value { get; set; }
}
~~~

> 当您需要在地图操作之前/之后输入上下文信息时，后一种配置很有用。

### 条件映射 Condition(...)
*********************************************************
AutoMapper 允许您在映射该属性之前为必须满足的属性添加条件。
这可以在我们尝试从int映射到unsigned int的情况下使用。
~~~
public class Foo {
    public int baz;
}
public class Bar {
    public uint baz;
}
~~~

在下面的映射中，只有在源对象中大于或等于0时才会映射属性baz。
~~~
var mapper = new MapperConfiguration(cfg => {
  cfg.CreateMap<Foo,Bar>()
    .ForMember(dest => dest.baz, opt => opt.Condition(src => (src.baz >= 0)));
}).CreateMapper();
~~~

测试代码
~~~
public class Program
{
    public static void Main(string[] args)
    {
        var mapper = new MapperConfiguration(cfg => {
            cfg.CreateMap<Foo, Bar>()
                .ForMember(dest => dest.baz, opt => opt.Condition(src => (src.baz >= 0)));
        }).CreateMapper();

        // 条件符合的
        var f = new Foo() { baz = 1 };
        var d = mapper.Map<Bar>(f);

        // 条件不符合的
        var f2 = new Foo() { baz = -1 };
        var d2 = mapper.Map<Bar>(f2); // 抛出异常 AutoMapper.AutoMapperMappingException:“Error mapping types.”         
        Console.ReadKey();
    }
}
public class Foo
{
    public int baz;
}
public class Bar
{
    public uint baz;
}
~~~

前提条件同样还有一种 PreCondition 方法。不同之处在于，在解决源值之前，它在映射过程中运行得更快（想想MapFrom）。因此调用前置条件，然后我们决定哪个将成为映射的源（解析），然后调用条件，最后分配目标值。
~~~~
var mapper = new MapperConfiguration(cfg => {
  cfg.CreateMap<Foo, Bar>()
    .ForMember(dest => dest.baz, opt => {
        opt.PreCondition(src => (src.baz >= 0));
        opt.MapFrom(src => {
            // Expensive resolution proccess that can be avoided with a PreCondition
        });
    });
}).CreateMapper();
~~~~

### dynamic 和 ExpandoObject 映射
*********************************************************
AutoMapper 可以映射到动态对象/从动态对象映射，无需任何显式配置。
~~~
public class Program
{
    public static void Main(string[] args)
    {
        // 泛型
        dynamic foo = new ExpandoObject();
        foo.Bar = 5;
        foo.Baz = 6;

        // 以下的写法会报错
        // var foo = new { Bar = 12, Baz = 23 };

        var mapper = new MapperConfiguration(c => { }).CreateMapper();
        var result = mapper.Map<Foo>(foo);
        result.Bar = 500;
        result.Baz = 222;
        dynamic foo2 = mapper.Map<ExpandoObject>(result);
        Console.WriteLine(foo2.Bar);
    }
}
public class Foo
{
    public int Bar { get; set; }
    public int Baz { get; set; }
}
~~~

> 类似地，您可以直接从 Dictionary <string，object> 映射到对象，AutoMapper 将使用属性名称排列键。

### 泛型类型映射
*********************************************************
AutoMapper 可以支持开放的泛型类型映射。为打开的泛型类型创建一个映射。
~~~
public class Source<T> {
    public T Value { get; set; }
}
public class Destination<T> {
    public T Value { get; set; }
}

// Create the mapping
var mapper = new MapperConfiguration(cfg => 
    cfg.CreateMap(typeof(Source<>), typeof(Destination<>))).CreateMapper();
~~~

您不需要为已关闭的泛型类型创建映射。AutoMapper 将在运行时将开放式通用映射中的任何配置应用于闭合映射。
~~~
var source = new Source<int> { Value = 10 };
var dest = mapper.Map<Source<int>, Destination<int>>(source);
dest.Value.ShouldEqual(10);
~~~

代码测试
~~~
public class Program
{
    static void Main(string[] args)
    {
        // Create the mapping
        var config = new MapperConfiguration(cfg => cfg.CreateMap(typeof(Source<>),typeof(Destination<>)));
        var source = new Source<int> { Value = 10 };
        var mapper = config.CreateMapper();
        var dest = mapper.Map<Source<int>, Destination<int>>(source);
        Console.ReadKey();
    }
}
public class Source<T>
{
    public T Value { get; set; }
}
public class Destination<T>
{
    public T Value { get; set; }
}
~~~