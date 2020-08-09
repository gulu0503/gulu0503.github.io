---
layout: post
title: "ASP.NET Core學前惡補篇 - 依賴注入（一）"
subtitle: '依賴反轉原則'
author: "Gulu"
header-style: text
date: 2020-07-01 12:00:00
tags:
  - ASP.NET Core
  - IoC
---

如果是從ASP.NET轉到ASP.NET Core的人，平常又沒有使用Autofac、Unity之類DI框架的經驗，那對於ASP.NET Core大量使用依賴注入應該會很不習慣。

有看到一些朋友實際去接觸時，被「依賴反轉」、「控制反轉」、「依賴注入」這些拗口的名詞搞得暈頭轉向而卻步，所以希望這篇文章可以幫助快速(應該吧@@)的了解這些概念。



## 依賴反轉原則（Dependency inversion principle，DIP）

首先是[SOLID原則](https://zh.wikipedia.org/wiki/SOLID_(%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E8%AE%BE%E8%AE%A1))中的「依賴反轉原則」，依賴反轉原則的定義如下：
> 1. 高層模組不應該依賴於低層模組，兩者都應該依賴於抽象介面。
> 2. 抽象介面不應該依賴於具體實現。而具體實現則應該依賴於抽象介面。

光看定義應該很難理解，我們從下面的程式碼範例來一步一步理解依賴反轉原則的定義：

```csharp
    public class ProductController : Controller
    {
        private readonly ProductService _productService;
        public ProductController()
        {
            _productService=new ProductService();
        }

        public IActionResult GetProductCount()
        {
            return View(_productService.GetProductCount());
        }
    }
    
    public class ProductService{
        public int GetProductCount(){
            //return product count
        }
    }
```

### 高層模組、低層模組
在上面這段程式碼中，主要有兩個類別`ProductController`、`ProductService`，在這個例子高層模組指的是`ProductController`，低層模組指的是`ProductService`。

我們可以簡單的理解因為`ProductController`調用`ProductService`所以`ProductController`是高層模組而`ProductService`是低層模組。

更好的理解是，`ProductController`是業務需求，而`ProductService`則是對於這個業務需求提供的實現，因此`ProductController`是高層模組。

### 依賴
```csharp
private readonly ProductService _productService;
```
在程式碼的最上面，我們可以看到`_productService`是一個`ProductService`類型的參考，而`ProductController`則會依賴於這個參考，來調用像是`GetProductCount`方法。

### 抽象
抽象在`.Net`指的通常就是「介面(Interface)」或「抽象類別(Abstract Class)」。
```csharp
    public class ProductService:IProductService{
        public int GetProductCount(){
            //return product count
        }
    }
    
    public interface IProductService{
        public int GetProductCount();
    }
```
如上面的程式碼所示，`IProductService`是`ProductService`的抽象，`IProductService`僅以抽象化的語意來呈現「要做的事情」，`ProductService`則是要「要做的事情」的具體實現。

### 反轉(依賴反轉)
理解了依賴反轉原則定義中的高層模組、低層模組、依賴、抽象後，我們來看看依賴反轉的「反轉」指的是什麼。

我們剛剛說上面的程式碼`ProductController`依賴於`ProductService`，所以「反轉」指的是變成`ProductService`依賴`ProductController`嗎？

其實不是的，所謂的「依賴反轉」是指讓`ProductController`依賴於抽象，而`ProductService`也依賴於抽象。

現在我們把前面的程式碼改成下面這樣：
```csharp
    public class ProductController : Controller
    {
        private readonly IProductService _productService;
        public ProductController()
        {
            _productService=new ProductService();
        }

        public IActionResult GetProductCount()
        {
            return View(_productService.GetProductCount());
        }
    }
    
    public class ProductService:IProductService{
        public int GetProductCount(){
            //return product count
        }
    }
    
    public interface IProductService{
        public int GetProductCount();
    }
```

我們可以看到`_productService`由一個`ProductService`類型的參考，變成了一個`IProductService`介面的參考。

#### 高層模組依賴介面
`ProductController`也從依賴於`ProductService`類型的參考來調用`GetProductCount`方法改為依賴`IProductService`來調用。

#### 低層模組依賴介面
```csharp
_productService=new ProductService();
```
`ProductService`的實例(instance)則是由原先指向`ProductService`類型的參考改為指向`IProductService`介面的參考。


