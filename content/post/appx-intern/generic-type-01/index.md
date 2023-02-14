---
title: '《APPX時賦科技》實習遊記（Ⅰ）－C# .NET中的泛型及其相關應用（上）'
summary: '介紹C# .NET中的泛型，並以實際情況來舉例解說其應用'
date: '2023-02-14'
draft: false
slug: 'generic-type-01'
author: '柯基汪汪'
usePageBundles: true
featureImage: '泛型上.png'

series:
  - APPX Intern
categories:
  - Software Development
tags:
  - Backend
  - C# .NET
  - Generic type
---

> 此文同步發表於[Medium](https://corgi-creeperjong.medium.com/)

最近在時賦科技的後端實習中，漸漸開始接觸到實際的專案了。與此同時，為了降低日後的維護成本，Coding不再只是以當下需求為目的，要做的更是預想未來可能發生異動的地方，並且思考現在如何架構程式，才能讓整個系統的可維護性更高。

在這樣的條件下，學一些新的技術是無可避免的。其中，泛型 ( Generic Type ) 是我先前學過，但並未實際應用在開發上的，今天就讓我來簡單介紹一下什麼是泛型，並給各位帶過幾個應用情境～

### 什麼是泛型？

用講的太慢了，直接進入例子。

假如今天有位高中生，他想透過程式來幫他算物體的動能，公式為`m * v * v / 2`，想當然爾，他會寫一個這樣的Function：

```C#
public static int GetKineticEnergy(int m, int v)
{
  return m * v * v / 2; // 非常的簡單，非常的直白
}
```

這樣的寫法解決了一半的問題，不過好景不常，有天他遇到了小數的m及v，加上他用的C#是強型別的語言，那該如何改寫呢？相信各位都猜到了。

沒錯，就是方法重載 ( Method overload ) ！（哈哈想不到吧）

```C#
public static int GetKineticEnergy (int m, int v)
{
  return m * v * v / 2; // 非常的簡單，非常的直白
}
public static double GetKineticEnergy (double m, double v)
{
  return m * v * v / 2; // 接收到double參數時，會自動選擇執行此函式
}
```

問題來了。這樣的寫法固然可以運作，可是如果...

1. 出現其他型態 ( long, long long, decimal, short... )
2. m與v的型態不同
3. 一覺醒來發現在平行宇宙，那邊的計算公式是m * m * v / 3

難道要把每種資料型態的排列組合都重載一次，再請一群平行宇宙的工程師把公式一個一個修改掉嗎？

這時，泛型 ( Generic Type ) 的存在就很重要了。

先來直接看看泛型的寫法：

```C#
public static T3 GetKineticEnergy<T1, T2, T3> (T1 m, T2 v)
{
  return m * v * v / 2;
}
```

> 務必注意，此處的寫法僅為概念解說，C#泛型並不支援算術運算子，下篇會再提及！

呼叫方法：

```C#
// m: long, v: long, 解: long
GetKineticEnergy<long, long, long>(m, v);
// m: int, v: long, 解: double
GetKineticEnergy<int, long, double>(m, v);
```

就像函式中的m與v的值，是呼叫函式後才決定的一樣，**泛型的概念就是讓型別由呼叫端傳入，待函式被呼叫後才決定參數的型態**。

換句話說，上面兩個呼叫的範例會讓函式變成以下型態去執行：

```C#
public static long GetKineticEnergy (long m, long v)
{
  return m * v * v / 2;
}
public static double GetKineticEnergy (int m, long v)
{
  return m * v * v / 2;
}
```

> 此處也僅為概念解說，實際編譯時不會產生兩種程式碼來執行。

這樣的寫法不僅可以增加程式對於不同型態的彈性，若今天真的跌進平行宇宙，改公式也不需要逐一修改重載函式了。

### 泛型的應用

除了上面應用在函式的例子，泛型可以用在幾乎任何關乎型別的地方。

#### 泛型類別

對於一個類別，若想要將裡面的成員寫成泛型，可以這樣做：

```C#
public class MyClass<T>
{
  // Your code
}
```

舉個例子，若我想自己寫一個stack的資料結構，可以這樣做：

```C#
public class MyStack<T>
{
  private List<T> cont = new(); // 容器本人
  public void push(T cur){
    cont.Add(cur);
  }
  public T pop(){
    return cont.RemoveAt(cont.Count-1);
  }
}
```

透過建構時傳入泛型參數，可定義該實例（通常是容器）所儲存的資料型別，這樣不僅維持了彈性，還可以有效維護資料結構的型別安全，防止其他非預期的型態傳入。若使用`object`等父類別作為型態去儲存，除了無法統一資料結構儲存的型態，使用時還要進行boxing和unboxing（object與其他型別互相轉換的過程），寫法和效能上都非常不美觀。

#### 泛型介面

將類別抽象化的介面，也可以做成泛型。

假如我要為上面自己的stack寫一個有`push`和`pop`方法的介面，可以這樣寫：

```C#
public interface IStack<T> where T : struct
{
  public void push(T cur);
  public T pop();
}
```

類別則改成這個樣子：

```C#
public class MyStack<T> : IStack<T> where T : struct
{
  private List<T> cont = new(); // 容器本人
  public void push(T cur){
    cont.Add(cur);
  }
  public T pop(){
    T ret = cont.Last();
    cont.RemoveAt(cont.Count-1);
    return ret;
  }
}
```

同樣的概念，這樣的寫法大大增加介面實作的彈性，任何實作`IStack`介面的類別，可以自行決定要傳入及回傳的資料型態。比較不同的是，這裡使用到泛型約束的概念，用來對泛型做相關的限制，後面會再提到。

#### 泛型委託

又是一個假如。假如我要寫一個排序函式，並且比較函式及型態可透過參數傳入，可以這樣寫：

```C#
// 宣告委託
public delegate bool CmpDel<T>(T x, T y);
// 排序函式
public static void MySort(List<T> arr, CmpDel<T> cmp)
{
  // Sort algorithm
}
// 自訂比較函式，篇幅關係此處以int舉例
public static bool MyCmp(int a, int b)
{
  // Return true if a < b
}
// 呼叫方法
MySort<int>(arr, MyCmp); // 由於是Call by reference，可直接對arr進行後續操作
```

同樣的做法也可以透過委託用在`event`的概念上，由於概念相似，在此就不舉例說明了。

用過委託的人一定都清楚，委託的宣告不外乎就是定義傳入和回傳的參數，因此System內建了三個經典的泛型委託供我們使用：`Action`, `Func`, `Predicate`。

`Action`與`Func`分別是宣告無回傳值及有回傳值的委託，兩者皆可帶入最多16個參數，也就是最多16個型別參數，`Func` 又因有回傳值，可帶入最多17個型別參數。上述的例子透過`Func`可以改成這樣：

```C#
// 不需宣告委託
// 排序函式
public static void MySort(List<T> arr, Func<T, T, bool> cmp)
{
  // Func<T, T, bool>為傳入兩個泛型參數，回傳布林值

  // Sort algorithm
}
// 下略
```

`Action`沒有回傳值，適合作為**執行某項操作**的委託函式。`Predicate`則是常用在**檢查該型別的變數是否符合某種規範**的委託函式，其固定回傳布林值。

### 泛型存在的問題

看完例子開始使用前，必須先了解一下泛型可能帶來的問題。

泛型在寫功能通用的函式時固然好用，但也有一些使用時要注意的點，這些問題大多都圍繞在「**型別是否與函式相容？**」。至於相容的程度，可以大略分成下面三種：

1. 型別是否可為Null？
2. 型別是否存在該變數？
3. 型別是否存在該方法？
4. 型別是否相容函式內其他呼叫？

幸好，以上問題都可以透過內建的方法解決，像是泛型約束、反射、`System.Type`等等，都是使用泛型時常用的好夥伴。

### 泛型約束

泛型是個海納百川的型別，不過在程式實作上，我們經常沒有辦法處理所有型別，更何況自定義的類別。因此，泛型約束在一定程度上限制了傳入的型別參數，讓編譯器可以在出事前，及時幫我們攔住不合法的型別。關於約束的類型，官方文件給得還算齊全（雖然是機翻），這裡我就用白話加一點範例給各位說明。

在上面的泛型介面的範例中，使用到`struct`的條件約束，除此之外，還有各種形式的約束，格式如下。其中針對一個型別參數，可以加上多個條件，對於多個型別參數，`where`也可以重複使用，給不同的型別參數加上約束：

```C#
class ClassName<T1, T2, ...>
  where T1 : <condition1>, <condition2>, <condition3> ...
  where T2 : ...
  ...
{}
```

以下是幾個常用的條件：

* `struct` : 型別參數必須是Value type，也就是預設為Pass by value的型別，像int, float, long long，以及以enum和struct宣告的型別。
* `class` : 型別參數必須是Reference type，也就是預設為Pass by reference的型別，像string, object，以及以class, interface, delegate宣告的型別。可加上?來允許可為null的型別。

Pass by value和Pass by reference各有用武之地。如上述MyStack的例子，常理上Stack中的內容不該被類別外的操作修改，因此會在類別的型別參數加上struct的條件約束，以保證Push進Stack的內容皆是複製品，而不是參考。

* `notnull` : 型別參數必須不可為null，不論Value type或Reference type。

務必注意是「帶入的型別」本身不接受null，而非「型別參數」不可為null。

* `new()` : 型別參數必須有「`public`且不帶參數的建構子」，不可以與`struct`並用。

由於Value type不可能帶有建構子，因此不可與`struct`同時約束（總沒遇過連宣告int都要用`new()`吧）。

* 類別名稱 : 型別參數必須是類別本身，或是繼承自該類別。可加上?來允許可為null的型別。
* 介面名稱 : 型別參數必須是介面本身，或是實作該介面。可加上?來允許可為null的型別。
* 另一個型別參數 : 型別參數必須是此型別參數，或是繼承/實作該型別參數。

以上面Stack為例，若我期望設計一個類別，用來維護好幾個不同結構的Stack（一般的Stack、Monotonic stack等等），且Stack內僅可存放為int，那麼可以這樣寫：

```C#
class MyStackCollection<T> where T : class, IStack<int>
{
  List<T> cont = new();
  // 其他維護操作
}
```

範例中，先使用了`class`來約束`T`只能是參考型別，使用參考傳遞 ( Pass by reference ) ，否則一整個Stack用值傳遞實在是太耗效能。再來，使用`IStack<int>`限制傳入的型別參數必須實作int版本的`IStack`介面，至於實作的內容則不限制，才可讓類別內的`cont`容器儲存各種實作版本的Stack。

值得一提的是，做為限制的介面也可以採用泛型的形式。舉個例子：

```C#
public void BubbleSort<T>(List<T> arr) where T : IComparable<T>
{
  for(int i = 0;i < arr.Count;++i)
  {
    for(int j = i+1;j < arr.Count;++j)
    {
      if(arr[i].CompareTo(arr[j]) > 0)
      {
        swap(arr[i], arr[j]);
      }  
    }
  }
}
```

這裡用了泛型介面來約束了型別參數，如果照我們前面所提到關於介面的約束，`where T : IComparable<T>`翻成中文應該是「`T`必須實作`IComparable<T>`這個介面」。有些人聽了可能會覺得很奇怪，自己都還沒定義好，怎麼還可以當別人的型別參數？

先來講介面本身。`IComparable<T>`中宣告了`CompareTo`這個方法，任何類別只要繼承此介面並實作`CompareTo`，皆可以進行同型別間的比較，而具體上`CompareTo`長這樣：

```C#
public int CompareTo (T? other);
/*
  this < other  => return <  0
  this == other => return == 0
  this > other  => return >  0
*/
```

這樣`where T : IComparable<T>`就變非常合理了！「`T`必須實作`IComparable<T>`這個介面」此時就相當於「`T`一定得實作`CompareTo`這個『與另一個型別同為T的參數比大小』的函式」，再白話一點就是「`T`必須先被定義好如何比大小」，對於排序這種一定需要比較的演算法來看，這種約束是再合理不過了。

### 小結

這篇簡單介紹了何謂泛型、在各個語法上的應用、泛型約束及一些實際例子。若是想單純了解概念的朋友們，這篇應該也算綽綽有餘。下一篇我會提到更多實際使用泛型上會遇到的問題及相關的解決工具，包括`System.Type`、`Reflection`等等，想實際動手操作的朋友可以繼續往下一篇閱讀！
