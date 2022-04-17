---
title: "switch 的模式比對"
date: 2022-04-16T10:09:41+08:00
description: 本篇整理 switch 的各種模式比對（至 C# 9.0）。
draft: false
hideToc: false  #側欄目錄Switch預設開啟或隱藏
enableToc: true   #側欄目錄
enableTocContent: true   #頁面目錄
tocLevels: ["h2", "h3", "h4"]
author: Zoey
authorEmoji: 👻
pinned: false
tags:
- switch
- pattern matching
categories:
- C#
---

本篇整理 switch 的各種模式比對（至 C# 9.0），主要參考[微軟文章(Patterns)](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns)。

測試環境：
> .NET Framework 4.7.2 (C# 7.3)
> .NET 6.0 (C# 10)

如果標記 C# 9.0，預設在 .NET 5.x 以上才可使用；C# 8.0 則是 .NET Core 3.x 後可以用。

基本架構：執行運算式，由上而下檢查執行結果符合哪個狀況，再執行裡面的陳述式。

``` csharp
switch (運算式)
{
    case 狀況一:
        Console.WriteLine("狀況一");
        break;
    case 狀況二:
        Console.WriteLine("狀況二");
        break;
    default: //其他狀況
        Console.WriteLine("其他");
        break;
}
```

## Declaration pattern 宣告模式 (C# 7.0)
當運算結果不為 null 時，檢查此結果的 runtime 型別，如果可找到對應的狀況，則轉型此結果並指派給宣告的變數。如果運算結果有可能為 null，可增加狀況 null 的檢查。

``` csharp
static void ShowTypeAndValue(object obj)
{
    switch (obj)
    {
        case int n:
            Console.WriteLine($"int:{n}");
            break;
        case string s:
            Console.WriteLine($"string:{s}");
            break;
        case null:
            throw new ArgumentNullException(nameof(obj));
        default:
            throw new ArgumentException("Unknown type", nameof(obj));
    }
}
```

宣告模式也可以用在運算結果是可為 Null 的實質型別變數，檢查其基礎類型；或是 boxed 的實質型別，存在某轉型方式。

如果只檢查型別，不需要使用轉型後的變數，可用 `_` 替代宣告的變數名稱，或是在 C# 9.0 後可用「類型模式」。

``` csharp
static void ShowType(object obj)
{
    switch (obj)
    {
        case int _:
            Console.WriteLine($"int");
            break;
        case string _:
            Console.WriteLine($"string");
            break;
        case null:
            throw new ArgumentNullException(nameof(obj));
        default:
            throw new ArgumentException("Unknown type", nameof(obj));
    }
}
```

## Type pattern 類型模式（C# 9.0）
類似宣告模式，但是只想檢查運算結果的型別時可使用。

``` csharp
static void ShowType2(object obj)
{
    switch (obj)
    {
        case int:
            Console.WriteLine($"int");
            break;
        case string:
            Console.WriteLine($"string");
            break;
        case null:
            throw new ArgumentNullException(nameof(obj));
        default:
            throw new ArgumentException("Unknown type", nameof(obj));
    }
}
```

## Constant pattern 常數模式 (C# 7.0)
檢查運算結果是否符合指定的狀況，而這些狀況會是常數的形式（任何的常數，例如整數、char、字串、布林值、Enum、常數欄位或是 null 等，C# 9.0 後可以用 not null）。

``` csharp
static decimal GetDiscount(int itemCount)
{
    switch (itemCount)
    {
        case 1:
            return 1m;
        case 2:
            return 0.95m;
        case 3:
            return 0.9m;
        case 4:
            return 0.8m;
        default:
            throw new ArgumentException($"Not supported number of items: {itemCount}", nameof(itemCount));
    }
}
```

## Relational pattern 關聯式模式（C# 9.0）
將運算結果與常數做比較，每個狀況中，可以用關聯運算子（`>`、`<`、`>=`、`<=`）來描述如何比較，而常數部分需要放在關聯運算子的右方；也可以混用常數模式，比較是否與常數值相等。這裡的常數可以用數值型別、char 或 Enum。

使用時要注意各狀況的順序排列，例如 `< 1` 後再檢查 `< 0`，則後面的 `< 0` 永遠不會執行到，編譯器會警告錯誤，無法編譯。

``` csharp
static string BMIResult(double bmi)
{
    switch (bmi)
    {
        case < 0:
            throw new ArgumentException($"BMI異常", nameof(bmi));
        case < 18.5:
            return "體重過輕";
        case < 24.0:
            return "健康體重";
        case < 27.0:
            return "體重過重";
        default:
            return "肥胖";
    }
}
```

可以加上邏輯模式中的 `and` 來檢查運算結果是否在某範圍內：
``` csharp
static string BMIResult2(double bmi)
{
    switch (bmi)
    {
        case < 0:
            throw new ArgumentException($"BMI異常", nameof(bmi));
        case >= 0 and < 18.5:
            return "體重過輕";
        case >= 18.5 and < 24.0:
            return "健康體重";
        case >= 24.0 and < 27.0:
            return "體重過重";
        default:
            return "肥胖";
    }
}
```

## Logical pattern 邏輯模式（C# 9.0）
在狀況裡面，可以用 `not`、`and`、`or` 的組合來表達邏輯概念，在沒有括弧的情況下，邏輯判斷優先順序 `not` > `and` > `or`。

##### not (否定)

``` csharp
static string TestNull(object obj)
{
    if (obj is not null)
        return "not null";

    return "null";
}
static string TestNull2(object obj)
{
    switch (obj)
    {
        case not null:
            return "not null";
        default:
            return "null";
    }
}
```

##### and (且)
``` csharp
static string BMIResult2(double bmi)
{
    switch (bmi)
    {
        case < 0:
            throw new ArgumentException($"BMI異常", nameof(bmi));
        case >= 0 and < 18.5:
            return "體重過輕";
        case >= 18.5 and < 24.0:
            return "健康體重";
        case >= 24.0 and < 27.0:
            return "體重過重";
        default:
            return "肥胖";
    }
}
```

##### or (或)
``` csharp
static string BMIResult3(double bmi)
{
    switch (bmi)
    {
        case < 0:
            throw new ArgumentException($"BMI異常", nameof(bmi));
        case < 18.5 or > 24.0:
            return "體重過輕或過重";
        default:
            return "健康體重";
    }
}
```

## Property pattern 屬性模式 (C# 8.0)
使用運算式結果的屬性或欄位值來進行比對，屬於遞迴模式（recursive pattern），可以嵌套 （nested）其他模式比對；例如在宣告模式或型別模式中，檢查型別外，亦可加上屬性模式來檢查其屬性或欄位的值是否符合；或者在屬性模式上套嵌屬性模式。

``` csharp
static double GetRectangleArea(Rectangle rectangle)
{
    switch (rectangle)
    {
        case { Width: 0 }:
        case { Height: 0 }:
            return 0;
        default:
            return rectangle.Height * rectangle.Width;
    }
}

static double GetArea(Shape shape)
{
    switch (shape)
    {
        case Rectangle { Width: 0 }:
        case Rectangle { Height: 0 }:
        case Circle { Radius: 0 }:
            return 0;
        case Rectangle rectangle:
            return rectangle.Height * rectangle.Width;
        case Circle circle:
            return Math.PI * circle.Radius * circle.Radius;
        default:
            throw new ArgumentException();
    }
}
```

## Positional pattern 位置模式 (C# 8.0)
Tuple 的應用。當運算式的結果是 Tuple 或其型別具有分解方法（Deconstruct method）時，可以針對每個元素進行比對。也屬於遞迴模式（recursive pattern），[微軟文件](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#positional-pattern)中有更多的範例。

``` csharp
static decimal GetTotal(decimal price, int count)
{
    switch (price, count)
    {
        case (>= 500, >= 10):
            return price * count * 0.8m;
        case (>= 500, > 0):
        case (>= 50, >= 10):
            return price * count * 0.9m;
        default:
            return price * count;
    }
}
```

## var pattern 隱含型別模式 (C# 7.0)
將運算式的結果指派給某個區域變數時，此變數型別可用隱含型別 `var` 表達，常用在取代 `default` 的狀況，藉以將運算的結果在 `default` 的狀況中再次使用。

``` csharp
static bool TestValidBMI(Person person)
{
    switch (person.GetBMI())
    {
        case < 0:
        case null:
            Console.WriteLine($"Invalid BMI value!");
            return false;
        case var bmi:
            Console.WriteLine($"Person with BMI {bmi} accepted!");
            return true;
    }
}
```


## Discard pattern 捨棄模式 (C# 8.0)
配合 [switch 運算式](#其他-switch-運算式-c-80)的使用，`default` 的部分用底線 `_` 替代，更為簡潔。

``` csharp
static int GetTypeLabel(object obj) => obj switch
{
    int n => 1,
    string s => 2,
    _ => 3,
};
```

## Parenthesized pattern 括弧模式（C# 9.0）
在任何模式的前後加上括弧，來強調或改變原本的邏輯順序。

``` csharp
static string BMIResult4(double bmi)
{
    switch (bmi)
    {
        case < 0:
            throw new ArgumentException($"BMI異常", nameof(bmi));
        case not (>= 18.5 and < 24.0):
            return "體重過輕或過重";
        default:
            return "健康體重";
    }
}
```

## 其他： Case guards 案例防護 (C# 7.0)
利用 `when` 來限定其他要符合的條件，`when` 後面加上布林運算式。

``` csharp
static decimal GetTotal2(decimal price, int count)
{
    switch (price)
    {
        case >= 500 when count >= 10:
            return price * count * 0.8m;
        case >= 500:
        case >= 50 when count >= 10:
            return price * count * 0.9m;
        default:
            return price * count;
    }
}
```

## 其他： switch 運算式 (C# 8.0)
在 C# 8.0 後可以使用 expression body 讓 switch 的寫法更簡潔，例如原本的：

``` csharp
static int GetTypeLabel(object obj)
{
    switch (obj)
    {
        case int n:
            return 1;
        case string s:
            return 2;
        default:
            return 3;
    }
}
```

可以寫成：
``` csharp
static int GetTypeLabel(object obj) => obj switch
{
    int n => 1,
    string s => 2,
    _ => 3,
};
```

運算式 (`obj`) 和 `switch` 的順序調換，減去 `case` 和 `:` 的使用，改用 `=>`；原本 `case` 之間則使用逗號 `,` 分隔，另外 `default` 用底線 `_` 表達（[捨棄模式](#discard-pattern-捨棄模式-c-80)）。

##### 相關連結：
1. [[Micorsoft Docs] Patterns](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns)
1. [[Micorsoft Docs] Selection statements](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/statements/selection-statements)
1. [[Micorsoft Docs] switch expression](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/switch-expression)
1. [[Micorsoft Docs] C# language versioning](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/configure-language-version)

