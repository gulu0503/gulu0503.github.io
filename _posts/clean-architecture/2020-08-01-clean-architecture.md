---
layout: post
title: "Clean Architecture實作心得（一）"
subtitle: 'Clean Architecture'
author: "Gulu"
header-style: text
tags:
  - 架構
---

[Robert C. Martin的Clean Architecture](https://www.tenlong.com.tw/products/9789864342945)這本書剛出的時候，立刻以腦粉的心態手刀下單了一本回家防身。
前陣子把這本書拿出來看，快把整本書看完的時候，突然驚覺剛買的時候就已經看過一遍了，~~可見當初根本就沒看懂~~。

剛好在Github上又看到有高手把這個架構實作出來，所以就試著把手邊的專案改成Clean Architecture的架構。

## 先說說Clean Architecture
![CleanArchitecture](../_site/img/in-post/clean-architecture/CleanArchitecture.jpg)
Clean Architecture跟其他的架構(如常見的n-layers、六角架構等...)，都有著相同的目標，那就是關注點分離。
這些架構透過將軟體分層來實現這種分離，而Clean Architecture的分離概念就體現在上面那張同心圓的圖上。
### 分層
在這個同心圓中，外圈是機制，內圈是策略。
### 依賴規則
> 原始碼依賴關係只能指向內部，朝向更高層級的策略。
依賴規則是這個架構的核心規範，內圈應該要對外圈的事情一無所知：
1. 外圈宣告的名稱包含函式、類別、變數或其他被命名的軟體實體，不應該在內圈的程式碼中出現。
2. 外圈宣告的資料格式也不應該被內圈所使用。
### 同心圓的各圈
接著由內到外看看這四圈分別代表的意義
#### Entities(實體層)
Entity指的可以是具有方法的物件，也可以是資料結構和函式，Entity封裝了企業的關鍵規則，如果你不是企業只是一個應用程式，那它就是應用程式的業務物件。
總之實體層包含了最一般性和最高層級的策略，當外部變化(如：UI操作改變、更換DB等...)時，這層是最不可能改變的。
#### Use Cases(使用案例層)
使用案例層包含了**應用程式特定**的業務規則，它封裝並實作系統所有的使用案例。
這一層變化時，不應該去影響到實體。而資料庫、UI等其他外部因素變化時，也不應該影響到這一層。
#### Interface adapters(介面轉接層)
這一層的軟體是一組轉接器，把資料從適合使用案例和實體的格式轉為適合某些外部代理(如資料庫或UI)的格式。
如果資料庫是一個SQL資料庫，那麼所有的SQL都應該寫在這一層。
#### Frameworks and Drivers(框架和驅動層)
最外層通常由框架和工具組成（如資料庫和Web框架），一般來說會在這邊額外寫的程式碼大部分都是向內圈溝通用的程式碼。

## 實作
程式碼：https://github.com/gulu0503/DeployTool
### 分層與依賴規則
#### 分層
- DeployTool.SharedKernel
  這一層放置一些其他專案也會用到的程式碼。
- DeployTool.Core
  這一層主要放業務規則相關的程式碼。
- DeployTool.Infrastructure
  這一層主要對應到Interface adapters相關的程式碼，這個專案沒有使用資料庫，用文字檔當作資料來源，所以讀取這些檔案和Parse的程式碼就寫在這一層。
- DeployTool.Winform
  使用Winform當UI框架。
#### 依賴規則
實際要真的做到符合上面的依賴規則，如果用以前的習慣來開發，其實會有點卡，大概列出下面幾點：
1. 套件支援度的問題
> 在這個依賴規則下，內圈應該要負責業務邏輯（抽象），外圈才負責細節（實作）。
也就是說如果套件把內圈要用到的介面和外圈包在一起，而你要想符合這條規則，那就必須要自己額外再包一層抽象給內圈使用。
不過好在現在.NET Core大部分的基礎套件都有把抽象跟實作拆成兩個套件。
2. 程式碼撰寫的習慣需要改變
> 這邊舉一個EF的例子。
假設要設定資料表中某個欄位的長度，我以前的習慣是用MaxLength Attribule來做設定。
而這個方式在這邊就不太合用，以這個方案為例，EFPackages會安裝在Infrastructure專案，而對應到EntityPOCO類別所在的Core專案則不認識MaxLength Attribule。是把EF安裝在Core專案也不是一個好的選擇，因為Core專案該只要管好業務規則部分，而不應該知道EF這個東西。假設來外圈不想使用改Dapper，Core專案是不應該需要為此做出何修改的。
因此在這個架構下，使用EF時，會比較大量地使用Fluent Ap而棄用Attribute的方式。
3. 跟第三方程式庫結婚?
`Robert C. Martin`說`Don’t marry the framework.`，因為把第三方程式庫耦合到我們內圈的程式碼中，未來如果第三方套件發生重大更新，那外圈的程式不就也都要跟著改了？
但是實際遵循這樣的規則寫起來又很痛苦，比如說Parse JOSN是一個很基本的需求，卻因為內圈沒有安裝JSON.NET之類的程式庫而導致達成這個需求很困難。
後來上網查一下別人針對這點的討論，發現有個論點很有道理－
我們必須要認真的考慮「結婚」這個詞，當我們將一個第三方的程式庫或是框架放在內圈來嫁給它時，這意味著我們認定未來都會跟它一直在一起，幾乎很難擺脫它了。
所以當要把一個第三方程式庫放進內圈時，必須審慎的評估，然後負起責任－「不論生老病死，貧窮疾病．．．．．．都會對它不離不棄」。

## 參考資料
- [Implementing Clean Architecture - Frameworks vs. Libraries
](http://www.plainionist.net/Implementing-Clean-Architecture-Frameworks/)