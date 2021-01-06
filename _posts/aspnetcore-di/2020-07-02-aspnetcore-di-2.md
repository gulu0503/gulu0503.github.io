---
layout: post
title: "ASP.NET Core - 依賴注入（二）"
subtitle: '控制反轉 & 依賴注入'
author: "Gulu"
header-style: text
date: 2020-07-02 12:00:00
tags:
  - ASP.NET Core
  - IoC
---

## 控制反轉（Inversion of Control，縮寫為IoC）

### 控制
控制反轉的「控制」指的是「控制權」－「依賴物件的取得」之控制權。
```csharp
_productService=new ProductService();
```
像前面的範例，我們在`ProductController`中利用`new ProductService()`實例化`ProductService`以取得`ProductService`這個依賴物件。

### 反轉(控制反轉)
所以反轉指的是我們不在`ProductController`裡透過`new`來實例化`ProductService`，而是把這個控制權給反轉－轉移出去。
以前面的程式碼範例來說，就是我們在`ProductController`僅定義其需要`IProductService`的依賴，而不在其程式碼透過`new`來實例化`ProductService`。

## 依賴注入（Dependency Injection，縮寫為DI）
前面的控制反轉是一個設計概念，比較主要的實現方式有「依賴注入」、「依賴查找」，這邊只說比較常見的「依賴注入」。

### 注入
在剛才的程是碼範例中，我們僅在`ProductController`定義需要`IProductService`的依賴，而「依賴物件的取得」之控制權則是交由外部。
依賴注入則是指外部取得這個「依賴物件」後，以注入的方式給`ProductController`。
```csharp
    public class ProductController : Controller
    {
        private readonly IProductService _productService;
        public ProductController(IProductService productService)
        {
            _productService=productService;
        }

        public IActionResult GetProductCount()
        {
            return View(_productService.GetProductCount());
        }
    }
```
依賴注入比較常見有三種建構式注入、方法注入、屬性注入，上面的程式碼範例是ASP.NET Core最常用的建構式注入。
上面的範例`ProductController`在建構式上定義一個`IProductService`型別的建構子（定義其需要`IProductService`），而原本`IProductService`對應的依賴物件`ProductService`則在外部實例化後，由`ProductController`的建構式注入。

```csharp
    public class Program
    {
        public static void Main(string[] args)
        {
            var controller=new ProductController(new ProductService());
        }
    }
```

上面的程式碼展示了我們將控制權轉移到了外部的Client端，在外部建立了`ProductService`的實例，並透過`ProductController`的建構式注入。