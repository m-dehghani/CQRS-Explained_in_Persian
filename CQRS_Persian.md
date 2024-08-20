## **در طراحی دامنه‌محور (DDD) با مثال‌هایی  #C**
جداسازی مسئولیت فرمان و پرس‌وجو (CQRS) یک الگوی معماری است که مدل‌های خواندن و نوشتن داده‌ها را جدا می‌کند. این جداسازی  انعطاف‌پذیری و مقیاس‌پذیری بیشتر در برنامه‌ها را فراهم می‌کند.

#### **مفاهیم کلیدی:**

~ **فرمان‌ها**: این عملیات‌ها وضعیت سیستم را تغییر می‌دهند. به عنوان مثال، ایجاد یک سفارش جدید یا به‌روزرسانی اطلاعات مشتری.  

~ **پرس‌وجوها**: این عملیات‌ها داده‌ها را بدون تغییر وضعیت سیستم بازیابی می‌کنند. به عنوان مثال، دریافت جزئیات سفارش یا اطلاعات مشتری.

#### **مثال در #C:**

command:

```C#  
public class CreateOrderCommand  
{  
    public Guid OrderId { get; set; }  
    public string CustomerName { get; set; }  
    public List<OrderItem> Items { get; set; }  
}

public class CreateOrderCommandHandler : IRequestHandler<CreateOrderCommand, bool>  
{  
    private readonly IOrderRepository _orderRepository;

    public CreateOrderCommandHandler(IOrderRepository orderRepository)  
    {  
        _orderRepository = orderRepository;  
    }

    public async Task<bool> Handle(CreateOrderCommand request, CancellationToken cancellationToken)  
    {  
        var order = new Order(request.OrderId, request.CustomerName, request.Items);  
        await _orderRepository.AddAsync(order);  
        return true;  
    }  
}
```

query:

```C#  
public class GetOrderByIdQuery : IRequest<OrderDto>  
{  
    public Guid OrderId { get; set; }  
}

public class GetOrderByIdQueryHandler : IRequestHandler<GetOrderByIdQuery, OrderDto>  
{  
    private readonly IOrderRepository _orderRepository;

    public GetOrderByIdQueryHandler(IOrderRepository orderRepository)  
    {  
        _orderRepository = orderRepository;  
    }

    public async Task<OrderDto> Handle(GetOrderByIdQuery request, CancellationToken cancellationToken)  
    {  
        var order = await _orderRepository.GetByIdAsync(request.OrderId);  
        return new OrderDto  
        {  
            OrderId = order.Id,  
            CustomerName = order.CustomerName,  
            Items = order.Items.Select(i => new OrderItemDto { ItemId = i.ItemId, Quantity = i.Quantity }).ToList()  
        };  
    }  
}
```

### **خواندن و نوشتن در پایگاه داده در CQRS**

در الگوی CQRS، خواندن و نوشتن به پایگاه داده به صورت جداگانه انجام می‌شود تا عملکرد و مقیاس‌پذیری بهینه شود.

#### **نوشتن درون پایگاه داده (commands):**

فرمان‌ها مسئول به‌روزرسانی وضعیت سیستم هستند. در اینجا یک مثال از نحوه مدیریت یک فرمان در #C آورده شده است:

```C#  
public class CreateOrderCommand  
{  
    public Guid OrderId { get; set; }  
    public string CustomerName { get; set; }  
    public List<OrderItem> Items { get; set; }  
}

public class CreateOrderCommandHandler : IRequestHandler<CreateOrderCommand, bool>  
{  
    private readonly IOrderRepository _orderRepository;

    public CreateOrderCommandHandler(IOrderRepository orderRepository)  
    {  
        _orderRepository = orderRepository;  
    }

    public async Task<bool> Handle(CreateOrderCommand request, CancellationToken cancellationToken)  
    {  
        var order = new Order(request.OrderId, request.CustomerName, request.Items);  
        await _orderRepository.AddAsync(order);  
        return true;  
    }  
}
```

در این مثال، `CreateOrderCommandHandler` فرمان `CreateOrderCommand` را با افزودن یک سفارش جدید به پایگاه داده از طریق `IOrderRepository` مدیریت می‌کند.

#### **خواندن از پایگاه داده (queries):**

پرس‌وجوها مسئول بازیابی داده‌ها، بدون تغییر وضعیت هستند. در اینجا یک مثال از نحوه مدیریت یک پرس‌وجو در #C آورده شده است:

```C#  
public class GetOrderByIdQuery : IRequest<OrderDto>  
{  
    public Guid OrderId { get; set; }  
}

public class GetOrderByIdQueryHandler : IRequestHandler<GetOrderByIdQuery, OrderDto>  
{  
    private readonly IOrderRepository _orderRepository;

    public GetOrderByIdQueryHandler(IOrderRepository orderRepository)  
    {  
        _orderRepository = orderRepository;  
    }

    public async Task<OrderDto> Handle(GetOrderByIdQuery request, CancellationToken cancellationToken)  
    {  
        var order = await _orderRepository.GetByIdAsync(request.OrderId);  
        return new OrderDto  
        {  
            OrderId = order.Id,  
            CustomerName = order.CustomerName,  
            Items = order.Items.Select(i => new OrderItemDto { ItemId = i.ItemId, Quantity = i.Quantity }).ToList()  
        };  
    }  
}
```

در این مثال، `GetOrderByIdQueryHandler` پرس‌وجوی `GetOrderByIdQuery` را با بازیابی جزئیات یک سفارش از پایگاه داده از طریق `IOrderRepository` مدیریت می‌کند.

### **ارتباط بین پایگاه‌های داده در CQRS**

[در الگوی CQRS، عملیات خواندن و نوشتن اغلب به پایگاه‌های داده جداگانه ارسال می‌شوند تا عملکرد و مقیاس‌پذیری بهینه شود1](https://learn.microsoft.com/en-us/azure/architecture/patterns/cqrs)[2](https://www.redhat.com/architect/illustrated-cqrs)[3](https://ibm-cloud-architecture.github.io/refarch-eda/patterns/cqrs/).

در اینجا نحوه ارتباط بین پایگاه‌های داده توضیح داده شده است:

### پایگاه داده نوشتن ###

~
    پایگاه داده نوشتن برای مدیریت فرمان‌ها استفاده می‌شود. این پایگاه داده برای عملیات نوشتن بهینه شده و ممکن است شامل اعتبارسنجی‌های پیچیده و منطق تجاری باشد.  
    مثال: هنگامی که یک سفارش جدید ایجاد می‌شود، مدیریت فرمان جزئیات سفارش را به پایگاه داده نوشتن می‌نویسد.  

### پایگاه داده خواندن  ###

 ~
    پایگاه داده خواندن برای مدیریت پرس‌وجوها استفاده می‌شود. این پایگاه داده برای عملیات خواندن بهینه شده و ممکن است به گونه‌ای ساختار یافته باشد که از پرس‌وجوهای کارآمد پشتیبانی کند.  
    مثال: هنگامی که جزئیات سفارش بازیابی می‌شود، مدیریت پرس‌وجو داده‌ها را از پایگاه داده خواندن بازیابی می‌کند.  

### همگام‌سازی ###

~
    پایگاه‌های داده نوشتن و خواندن باید برای اطمینان از سازگاری همگام‌سازی شوند. این کار می‌تواند از طریق مکانیزم‌های مختلفی مانند event sourcing یا افزونگی  داده‌ها انجام شود.  
    مثال: هنگامی که یک سفارش جدید ایجاد می‌شود، یک رویداد منتشر می‌شود تا پایگاه داده خواندن با جزئیات سفارش جدید به‌روزرسانی شود.

### **توسعه روش CQRS با ماکروسرویس‌ها**


توسعه الگوی CQRS با ماکروسرویس‌ها شامل جدا کردن عملیات خواندن و نوشتن به سرویس‌های مختلف است. در اینجا یک راهنمای گام به گام به همراه مثال‌های کد در C# آورده شده است:

### **گام ۱: تعریف  domain models**

Order.cs

```C#  
public class Order  
{  
    public Guid Id { get; set; }  
    public string CustomerName { get; set; }  
    public List<OrderItem> Items { get; set; }  
}

public class OrderItem  
{  
    public Guid ItemId { get; set; }  
    public int Quantity { get; set; }  
}
```

### **گام ۲: ایجاد command-service**

CreateOrderCommand.cs

```C#  
public class CreateOrderCommand  
{  
    public Guid OrderId { get; set; }  
    public string CustomerName { get; set; }  
    public List<OrderItem> Items { get; set; }  
}
```

CreateOrderCommandHandler.cs

```C#  
public class CreateOrderCommandHandler : IRequestHandler<CreateOrderCommand, bool>  
{  
    private readonly IOrderRepository _orderRepository;

    public CreateOrderCommandHandler(IOrderRepository orderRepository)  
    {  
        _orderRepository = orderRepository;  
    }

    public async Task<bool> Handle(CreateOrderCommand request, CancellationToken cancellationToken)  
    {  
        var order = new Order  
        {  
            Id = request.OrderId,  
            CustomerName = request.CustomerName,  
            Items = request.Items  
        };  
        await _orderRepository.AddAsync(order);  
        return true;  
    }  
}
```

IOrderRepository.cs

```C#  
public interface IOrderRepository  
{  
    Task AddAsync(Order order);  
}
```

OrderRepository.cs

```C#  
public class OrderRepository : IOrderRepository  
{  
    private readonly DbContext _context;

    public OrderRepository(DbContext context)  
    {  
        _context = context;  
    }

    public async Task AddAsync(Order order)  
    {  
        _context.Orders.Add(order);  
        await _context.SaveChangesAsync();  
    }  
}
```

### **گام ۳: ایجاد query-service**

GetOrderByIdQuery.cs

```C#  
public class GetOrderByIdQuery : IRequest<OrderDto>  
{  
    public Guid OrderId { get; set; }  
}
```

GetOrderByIdQueryHandler.cs

```C#  
public class GetOrderByIdQueryHandler : IRequestHandler<GetOrderByIdQuery, OrderDto>  
{  
    private readonly IOrderRepository _orderRepository;

    public GetOrderByIdQueryHandler(IOrderRepository orderRepository)  
    {  
        _orderRepository = orderRepository;  
    }

    public async Task<OrderDto> Handle(GetOrderByIdQuery request, CancellationToken cancellationToken)  
    {  
        var order = await _orderRepository.GetByIdAsync(request.OrderId);  
        return new OrderDto  
        {  
            OrderId = order.Id,  
            CustomerName = order.CustomerName,  
            Items = order.Items.Select(i => new OrderItemDto { ItemId = i.ItemId, Quantity = i.Quantity }).ToList()  
        };  
    }  
}
```

OrderDto.cs

```C#  
public class OrderDto  
{  
    public Guid OrderId { get; set; }  
    public string CustomerName { get; set; }  
    public List<OrderItemDto> Items { get; set; }  
}

public class OrderItemDto  
{  
    public Guid ItemId { get; set; }  
    public int Quantity { get; set; }  
}
```

IOrderRepository.cs (برای سرویس پرس‌وجو)

```C#  
public interface IOrderRepository  
{  
    Task<Order> GetByIdAsync(Guid orderId);  
}
```

OrderRepository.cs (برای سرویس پرس‌وجو)

```C#  
public class OrderRepository : IOrderRepository  
{  
    private readonly DbContext _context;

    public OrderRepository(DbContext context)  
    {  
        _context = context;  
    }

    public async Task<Order> GetByIdAsync(Guid orderId)  
    {  
        return await _context.Orders.Include(o => o.Items).FirstOrDefaultAsync(o => o.Id == orderId);  
    }  
}
```

### **گام ۴:ایجاد ماکروسرویس‌ها**

 **ماکروسرویس command:**  

~ ماکروسرویس فرمان:  

    -ایجاد یک پروژه جدید ASP.NET Core Web API  
    -افزودن command handlers 
    -پیکربندی  پایگاه داده و تزریق وابستگی 

~ ماکروسرویس پرس‌وجو:  

    -ایجاد یک پروژه جداگانه ASP.NET Core Web API
    -افزودن query handlers  
    -پیکربندی  پایگاه داده و تزریق وابستگی

### **گام ۵: پیکربندی پایگاه داده**

DbContext.cs

```C#  
public class OrderDbContext : DbContext  
{  
    public DbSet<Order> Orders { get; set; }  
    public DbSet<OrderItem> OrderItems { get; set; }

    public OrderDbContext(DbContextOptions<OrderDbContext> options) : base(options) { }  
}
```

### **گام ۶: همگام‌سازی پایگاه‌های داده**

برای اطمینان از سازگاری بین پایگاه‌های داده فرمان و پرس‌وجو، می‌توانید از مکانیزم‌های event sourcing یا تکرار داده‌ها استفاده کنید. به عنوان مثال، هنگامی که یک سفارش جدید ایجاد می‌شود، یک رویداد منتشر می‌شود تا پایگاه داده خواندن با جزئیات سفارش جدید به‌روزرسانی شود.

Event.cs

```C#  
public class OrderCreatedEvent  
{  
    public Guid OrderId { get; set; }  
    public string CustomerName { get; set; }  
    public List<OrderItem> Items { get; set; }  
}
```

EventPublisher.cs

```C#  
public class EventPublisher  
{  
    public void Publish(OrderCreatedEvent orderCreatedEvent)  
    {  
        // کد برای انتشار رویداد به یک پیام‌رسان مانند RabbitMQ یا Kafka  
    }  
}
```

EventSubscriber.cs

```C#  
public class EventSubscriber  
{  
    private readonly IOrderRepository _orderRepository;

    public EventSubscriber(IOrderRepository orderRepository)  
    {  
        _orderRepository = orderRepository;  
    }

    public void Subscribe()  
    {  
        // کد برای اشتراک در رویداد از یک پیام‌رسان مانند RabbitMQ یا Kafka  
        // هنگامی که یک رویداد دریافت می‌شود، پایگاه داده خواندن را به‌روزرسانی کنید  
    }

    public async Task Handle(OrderCreatedEvent orderCreatedEvent)  
    {  
        var order = new Order  
        {  
            Id = orderCreatedEvent.OrderId,  
            CustomerName = orderCreatedEvent.CustomerName,  
            Items = orderCreatedEvent.Items  
        };  
        await _orderRepository.AddAsync(order);  
    }  
}
```

### **گام ۷: پیکربندی تزریق وابستگی**

Startup.cs (ماکروسرویس فرمان)

```C#  
public class Startup  
{  
    public void ConfigureServices(IServiceCollection services)  
    {  
        services.AddDbContext<OrderDbContext>(options =>  
            options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));  
        services.AddScoped<IOrderRepository, OrderRepository>();  
        services.AddMediatR(typeof(CreateOrderCommandHandler).Assembly);  
        services.AddControllers();  
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)  
    {  
        if (env.IsDevelopment())  
        {  
            app.UseDeveloperExceptionPage();  
        }

        app.UseRouting();  
        app.UseEndpoints(endpoints =>  
        {  
            endpoints.MapControllers();  
        });  
    }  
}
```

Startup.cs (ماکروسرویس پرس‌وجو)

```C#  
public class Startup  
{  
    public void ConfigureServices(IServiceCollection services)  
    {  
        services.AddDbContext<OrderDbContext>(options =>  
            options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));  
        services.AddScoped<IOrderRepository, OrderRepository>();  
        services.AddMediatR(typeof(GetOrderByIdQueryHandler).Assembly);  
        services.AddControllers();  
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)  
    {  
        if (env.IsDevelopment())  
        {  
            app.UseDeveloperExceptionPage();  
        }

        app.UseRouting();  
        app.UseEndpoints(endpoints =>  
        {  
            endpoints.MapControllers();  
        });  
    }  
}
```

### **گام ۸: تست ماکروسرویس‌ها**

~ ماکروسرویس فرمان:  
    از ابزارهایی مانند Postman برای ارسال درخواست POST به ماکروسرویس فرمان برای ایجاد یک سفارش جدید استفاده کنید.  
    اطمینان حاصل کنید که سفارش به پایگاه داده نوشتن اضافه شده است.  

~ ماکروسرویس پرس‌وجو:  
    از ابزارهایی مانند Postman برای ارسال درخواست GET به ماکروسرویس پرس‌وجو برای بازیابی جزئیات سفارش استفاده کنید.  
    اطمینان حاصل کنید که جزئیات سفارش از پایگاه داده خواندن بازیابی شده است.  
