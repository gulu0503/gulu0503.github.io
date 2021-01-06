---
layout: post
title: "ASP.NET Core - 依賴注入（三）"
subtitle: 'IoC容器'
author: "Gulu"
header-style: text
date: 2020-07-03 12:00:00
tags:
  - ASP.NET Core
  - IoC
---

前面的章節我們透過`new ProductController(new ProductService())`的方式完成了非常基本的依賴注入，但是一般實務上我們系統的物件間的依賴關係更加複雜，不太可能用這種方式來管理。

會造成依賴關係變複雜的原因很多，這邊先說明「依賴樹」和「生命週期」。

## 依賴樹
以前面的範例來說`ProductService`一般都還會再依賴於其他類型，如`IOptions<SystemSetting>`、`IProductService`，`ProductService`則可能依賴於如`DemoDbContext`，而呈現一個較複雜的樹狀依賴關係(相較於前面的範例)。
```csharp
    public class ProductService:IProductService{
        private readonly IOptions<SystemSetting>  _systemSetting;
        private readonly IProductRepository _productRepository;
        public ProductService(IOptions<SystemSetting> systemSetting,IProductRepository productRepository){
            _systemSetting = systemSetting;
            _productRepository = productRepository;
        }
        public int GetProductCount(){
            //return product count
        }
    }
    
    public class ProductRepository:IProductRepository{
        private readonly DemoDbContext _dbContext;
        public ProductRepository(DemoDbContext dbContext){
        }
    }
```

## 生命週期
針對需求的不同，我們會希望每個注入的依賴物件有不同的生命週期，像是ASP.NET Core DI框架提供的生命週期模式有Transient、Scoped、Singleton。
- Transient：每次進行注入動作都會new一個新實例。
- Scoped：為每個範圍(通常就是指一個Request)new一個新實例。
- Singleton：服務實例只會再程式啟動時new一個實例，整個程式運行期間都使用這個實例。

這邊我們以前面範例的幾個物件來說明上面的三個生命週期。
- IOptions<SystemSetting>：系統參數通常是整個系統共用，所以我們一般會設為Singleton模式，避免每次都要重新讀取。
- DemoDbContext：Ef的DbContext一般是建議一個Request對應一個DbContext(One context per request)，所以一般是設為Scoped模式。
- IProductRepository、IProductService：如前面範例每次new一個新的實例，設為Transient模式。

## IoC容器(IoC Container)
如前所述這些複雜的依賴關係，我們會希望有一個東西能夠來幫助我們管理這些關係，這個東西就是IoC容器。

### 服務註冊
我們透過在應用程式啟動的時候，將所有我們需要用到的依賴物件(服務)實例的註冊訊息(依賴關係、生命週期等...)給容器物件，來進行服務的註冊。

如下範例，在ASP.NET Core的DI框架中，其IoC容器是一個IServiceProvider類型的物件。我們建立一個ServiceCollection實例，將註冊訊息設定進去，並透過`BuildServciceProvider()`方法取得建立一個IoC容器(IServiceProvider的實例)。

```csharp
    public class Program
    {
        public static void Main(string[] args)
        {
            var configuration = new ConfigurationBuilder()
            .AddJsonFile("appSettings.json").Build();   
        
            var serviceProvider = new ServiceCollection()
            .AddTransient<ProductController,ProductController>
            .AddTransient<IProductService,ProductService>
            .AddTransient<IProductRepository,ProductRepository>
            .Configure<SystemSetting>(options => 
                configuration
                .GetSection("SystemSetting")
                .Bind(options)
            );
            .AddDbContext<DemoContext>(options => 
                options.UseSqlite("i am connection string")
            ).BuildServciceProvider();
        }
    }
```

### 服務消費
我們最終是希望由容器能夠依我們設定的註冊訊息來提供服務實例，我們只要調用`IServiceProvider`的`GetService`方法即可由容器取得對應的服務實例。

如`serviceProvider.GetService<ProductController>()`，容器會依我們前面設定的註冊訊息，提供`ProductController`的服務實例。

```csharp
    public class Program
    {
        public static void Main(string[] args)
        {
            var configuration = new ConfigurationBuilder()
            .AddJsonFile("appSettings.json").Build();   
        
            var serviceProvider = new ServiceCollection()
            .AddTransient<ProductController,ProductController>
            .AddTransient<IProductService,ProductService>
            .AddTransient<IProductRepository,ProductRepository>
            .Configure<SystemSetting>(options => 
                configuration
                .GetSection("SystemSetting")
                .Bind(options)
            );
            .AddDbContext<DemoContext>(options => 
                options.UseSqlite("i am connection string")
            ).BuildServciceProvider();
            
            var controller = serviceProvider.GetService<ProductController>();
        }
    }
```