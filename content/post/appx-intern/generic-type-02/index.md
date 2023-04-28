---
title: '《APPX時賦科技》實習遊記（Ⅱ）－C# .NET中的泛型及其相關應用（中）'
summary: '延續上一篇，介紹System.Type在泛型上的應用'
date: '2023-04-28'
draft: false
slug: 'generic-type-02'
author: '柯基汪汪'
usePageBundles: true
featureImage: '泛型中.jpg'
toc: true

series:
  - APPX Intern
categories:
  - Software Development
tags:
  - Backend
  - C# .NET
  - Generic type
---

> 此文同步發表於[Medium](https://medium.com/appxtech/appx%E6%99%82%E8%B3%A6%E7%A7%91%E6%8A%80-%E5%AF%A6%E7%BF%92%E9%81%8A%E8%A8%98-%E2%85%B1-c-net%E4%B8%AD%E7%9A%84%E6%B3%9B%E5%9E%8B%E5%8F%8A%E5%85%B6%E7%9B%B8%E9%97%9C%E6%87%89%E7%94%A8-%E4%B8%AD-fccfe4d4e639)

這篇主要延續上一篇的內容，來仔細介紹一下在實際操作泛型時經常會用到的Type函式庫。由於本篇著重於「取得關於泛型型別的相關資訊」，因此比較無法帶入實際例子，不過文中也附上一些程式碼供各位參考，對泛型有點概念並想繼續了解的朋友可以往下看～

### 前情提要

泛型是一種概念。主要實現方法是使用型別參數 ( Type parameter ) 取代原先在函式、類別、介面等等之中不變的型別。對於傳入型別的相關限制，可使用泛型約束，讓程式在編譯時期即可攔下不合法的型別。

### 實作上的考量

上一篇有提到一些泛型使用上會遇到的問題：

1. 型別是否可為Null？
2. 型別是否存在該變數？
3. 型別是否存在該方法？
4. 型別是否相容函式內其他呼叫？

對於型別的檢查上，其實也沒有那麼複雜。如先前所述，泛型的實現說白了，就是將型別以參數的形式傳入，所以檢查型別事實上就是檢查參數的內容。倘若我們要檢查傳入參數的奇偶性，會怎麼做？

```C#
enum Parity
{
  Odd = 1,
  Even = 0
}
Parity ParityOf(int n)
{
  return (Parity)(n % 2);
}
// 呼叫方法
ParityOf(3)  == Parity.Odd  // true
ParityOf(20) == Parity.Even // true
ParityOf(17) == Parity.Even // false
```
    
> 當然你也可以直接判斷n % 2的結果，此處僅為引導範例。

問題來了，型別也可以這樣做嗎？答案是可以的。

```C#
typeof(int)    == 5.GetType();          // true
typeof(double) == 5.3.GetType();        // true
typeof(int)    == 2147483648.GetType(); // false
```

System.Type便是為型別分析而生的函式庫，下面讓我們來介紹一下。

### Type

#### typeof & GetType()

想像一下，假如你是位面試官，你現在要開始面試一位應徵者，你的第一個問題會是什麼？

一定是先問姓名或自我介紹，總不會劈頭就問你什麼時候能來上班吧？

泛型中的型別參數此時就像一位剛走進來的應徵者，你要了解它的一切，你必須先知道它叫什麼。

針對型別本身，可以使用`typeof`取得型別；對於以型別創建出的物件，則可以使用object中的`GetType()`來取得型別，範例如下：

```C#
class MyClass {}
void SayMyType<T>(T foo)
{
  Console.WriteLine($"typeof(T): {typeof(T)}, object.GetType(): {foo.GetType()}");
}
// 取得型別
Type typeFromClass = typeof(MyClass);  // 從型別本身取得
var myClassInstance = new MyClass();
Type typeFromInstance = myClassInstance.GetType(); // 從物件中取得
// 呼叫
SayMyType(2147483648); // typeof(T): System.UInt32, object.GetType(): System.UInt32
SayMyType("hello world!"); // typeof(T): System.String, object.GetType(): System.String
SayMyType(myClassInstance); // typeof(T): [namespace].MyClass, object.GetType(): [namespace].MyClass
```

> 注意：Type與型別本身不同，不可用來作為型別參數傳入。

知道了`typeof`和`GetType()`的用途後，來猜猜這段程式碼會輸出什麼？

```C#
public class MyClass<T> {}
if(typeof(MyClass<>) == typeof(MyClass<int>))
{
  Console.WriteLine("MyClass<> is equal to MyClass<int>!");
}
else
{
  Console.WriteLine("MyClass<> and MyClass<int> are not the same!");
}
```

答案是 "MyClass<> and MyClass<int> are not the same!"。

對Type還不熟的使用者，第一次可能會因為這件事而小碰壁（至少我自己就是）。在Type眼中，泛型的`MyClass<>`和帶入型別參數的`MyClass<int>`是兩個不同的型別，同樣地，使用這兩個型別創建物件後再使用`GetType()`，也是一樣的結果。這個問題在使用反射時會經常遇到，所以務必特別注意，下篇講反射時再給各位解說。

#### GetGenericTypeDefinition() & GetGenericArguments() & MakeGenericType()

針對像是`List<string>`, `Dictionary<string, int>`等等的型別，Type自然也沒有冷落它們，我們依舊可以用很直覺的方法取得泛型類別本身及其型別參數。

以`Dictionary<string, int>`為例，可使用以下方式取得型別：

```C#
// Dictionary<string, int>
Type type = typeof(Dictionary<string, int>);
Type typeDef = type.GetGenericTypeDefinition();
Console.WriteLine(typeDef);

// string, int
Type[] args = type.GetGenericArguments();
foreach(Type arg in args)
{
  Console.Write($"{arg} ");
}
Console.WriteLine();

// TKey, TValue
Type[] typeArgs = typeDef.GetGenericArguments();
foreach(Type typeArg in typeArgs)
{
  Console.Write($"{typeArg} ");
}
Console.WriteLine();

/* output:

System.Collections.Generic.Dictionary`2[TKey,TValue]
System.String System.Int32 
TKey, TValue

*/
```

對Type使用`GetGenericTypeDefinition`，可以取得定義泛型的類別本身，如`List<T>`，`Dictionary<TKey, TValue>`等等。若要取得`Dictionary<string, int>`中的string, int，則是使用`GetGenericArguments`取得`Type[]`，若對未帶入型別參數的`Dictionary<TKey, TValue>`使用，則得到 `TKey`, `TValue` 型別參數本身。

相反地，能夠將`Dictionary<string, int>`拆出來，也能將它們組回去：

```C#
Type dict = typeof(Dictionary<,>);
Type str  = typeof(string);
Type num  = typeof(int);

Console.WriteLine(dict.MakeGenericType(str, num));

/* output:

System.Collections.Generic.Dictionary`2[System.String,System.Int32]

*/
```

對於泛型類別的Type本身，可呼叫`MakeGenericType` 並依序帶入型別參數的Type，便可得到泛型型別帶入型別參數後的Type。至此，再也不用為泛型型別參數的內容及轉換而摸不著頭緒了。
#### Type.Is …

前面我們已經得知了應徵者的姓名，現在就可以開始詢問各種問題了。在Type中，大部分的屬性都幫我們做好身家調查了，我們只需要呼叫函式就可以得到想要的資訊，以下簡單舉例幾個statement（族繁不及備載，可自行查閱微軟文件）：

```C#
public class MyClass<T>
{
  private int _pubVar;
  public void TypeTest()
  {
    Console.WriteLine(typeof(int[]).IsArray);   // 是否為陣列
    Console.WriteLine(typeof(MyClass<>).IsGenericType);   // 是否為泛型
    Console.WriteLine(typeof(T).IsGenericType);  // 由於呼叫後會傳入型別，此處為false
    Console.WriteLine(typeof(MyClass<>).IsPublic); // 是否為Public
    // 巢狀結構內不可使用IsPublic判斷，改用IsNestedPublic
    Console.WriteLine((new MyClass<int>())._pubVar.GetType().IsNestedPublic); // false
    ...
  }
};
```

了解這些屬性後，再引用上面的小測驗，猜猜這段程式碼的輸出：

```C#
public class MyClass<T> {}
if(typeof(MyClass<>).IsGenericType)
{
  Console.WriteLine("MyClass<> is generic type!");
}
else
{
  Console.WriteLine("MyClass<> is fake!");
}
```

答案是 “ MyClass<> is generic type! ”。應該是再簡單不過。

Recall前面所提，記得MyClass<>和MyClass<int>是不同的型別，那麼以下程式碼的輸出呢？

```C#
public class MyClass<T> {}
if(typeof(MyClass<int>).IsGenericType)
{
  Console.WriteLine("MyClass<int> is generic type!");
}
else
{
  Console.WriteLine("MyClass<int> is not generic type for sure");
}
```

答案是 "MyClass<int> is generic type!"。

很神奇地，`MyClass<>`和`MyClass<int>`雖然在型別上分道揚鑣，但卻同樣被歸類於泛型，即使`MyClass<int>`已傳入型別參數也一樣（不知為何有種本是同根生，相煎何太急的既視感）。

#### is, as

在實作泛型時，有時我們只看重型別是否有繼承某個父類別或介面。針對子類別與父類別之間的關係判斷與轉換，可分別使用`is`, `as`來進行，用法上也非常接近英文語法。以下為示範程式碼：

```C#
class Human{}
class Author : Human{}
class Cat {}

Author corgi = new();
Console.WriteLine(corgi is Human); // true
Human person = corgi as Human; // 可使用person操作Human中的方法
```

`is`前接子類別，後接父類別，根據繼承關係與否回傳boolean。`as`則是將前者轉換成後者，成功回傳後者型別，失敗則回傳`null`，因此`as`使用後一定要檢查回傳值是否為`null`。

關於as的轉換，有以下的等價性：

```C#
corgi as Human;
// 等價於
corgi is Human ? (Human)(corgi) : (Human)null;
```

看似很合理，但在此有其它的情境，請看以下的擴充範例：

```C#
class Human {}
class Teenager : Human{}
class Student : Teenager{}

Human corgi = new Student();
Teenager teen = corgi as Teenager;
if(corgi != null)
  Console.WriteLine(corgi.GetType()); // Student
```

**corgi明明被宣告成Human，為何還可以轉換成子類別Teenager？**

首先，我們要先釐清`corgi`的型別是什麼。在多型的概念中，子類別可以指派給父類別的變數，而不失其內容。此處雖然`corgi`先被宣告成了`Human`，但由於指派了子類別`Student`給它，因此它變成了「披著`Human`皮的`Student`」，本質上不管是用`GetType`還是用`is`, `as`來驗證，它終究是`Student`類別。（這也是為什麼我一直不使用「轉型」兩字，型別本身並沒有轉換）

因此，`as`的運算便會從「`Human`轉換成`Teenager`」（不合理）變成「`Student`轉換成`Teenager`」（合理）。雖然直覺上有點違反常理，但至少此時不會讓編譯器對我們的程式碼大吼大叫。

> 尤其當程式碼冗長，或想對參數做`as`的轉換時，難以保證目前的變數儲存的是否是「子類別的instance」，因此使用`as`後，務必檢查結果是否等於null，以檢驗轉換成功與否。

回到主題，來舉個上述工具用在泛型上例子。若要寫一個輸出函式，但參數可以接受任何一般數值、字串以及**任何可迭代 ( iterable ) 數據結構**的：

```C#
public void Print<T>(T input)
{
  IEnumerable arr = input as IEnumerable;
  if(arr == null || input.GetType() == typeof(string))
    Console.WriteLine(arr);
  else
    foreach(var x in arr)
    {
      Console.Write($"{x} ");
    }
    Console.WriteLine();
}
```

此例中，先將泛型參數`input`使用`as`轉換成`IEnumerable`。若轉換失敗則`arr`為`null`，代表型別可能是int, double等不可枚舉的型別（在此先不討論自訂類別），直接輸出就結束了。若轉換成功則`arr`不為`null`，代表型別有實作`IEnumerable`的內容，可使用`foreach`並以空格分隔來輸出各個元素，以得到漂漂亮亮的輸出。

另外由於string也是有實作`IEnumerable`的類別，但輸出上我們不期望每個字元分隔輸出，因此在轉換失敗的條件式中多檢查了型別參數，若是字串也直接輸出。

#### Type建構黑科技

來個情境題，若今天在實作一個泛型函式時，他的定義如下：

```C#
// 忘記泛型約束的回到上一篇重看！！！
public void Foo<T>(T bar) where T : new()
```

在函式內要創建T型別的物件時，該怎麼做？

```C#
T t = new();
```

這是理所當然的。換個情境，若今天函式改成如下的定義：

```C#
public void Foo<T>(T dict) where T : IDictionary;
```

根據`IDictionary`的宣告，要對其進行鍵值對插入，必須使用Add並傳入鍵與值。此時，必定得建構一個Key吧？

問題來了，建構需要型別本身，但現在能獲得的型別本身只有型別參數T，該怎麼獲得T之中Key的型別，並建構物件來傳入Add函式？

答案是，有一種只需要Type即可在執行期間建構物件的函式。

依上述的函式定義，可以如以下方式實作插入：

```C#
public void Foo<T>(T dict) where T : IDictionary
{
  Type type = typeof(T);
  // 檢查T是否為泛型，且是否有傳入鍵值型別作為型參
  if(!type.IsGenericType || type.GetGenericArguments().Length != 2) return;
 
  Type keyType = type.GetGenericArguments[0];
  Type valueType = type.GetGenericArguments[1];
  // 建構物件
  object key = Activator.CreateInstance(keyType);
  object value = Activator.CreateInstance(valueType);
  /*
    這裡是關於鍵值的操作
  */
  dict.Add(key, value);
}
```

`Activator.CreateInstance()`中可以帶入Type來建構相對應的物件，不論是Value type或Reference type，皆可創建物件並回傳該物件的參考。針對Value type，會回傳該型別的預設值（參見下方default的補充說明），對於Reference type，則使用無參數的建構式創建物件。

若想指定建構式，也可透過帶入對應的參數，讓編譯器自己找最符合的建構式，以下是簡單的範例：

```C#
// string沒有無參數建構子，因此一定得帶入參數
char[] carray = new char[4] {'t', 'e', 's', 't'};
object s1 = Activator.CreateInstance(typeof(string), 'A', 8);
object s2 = Activator.CreateInstance(typeof(string), carray);
Console.WriteLine(s1);  // AAAAAAAA
Console.WriteLine(s2);  // test
```

透過`CreateInstance`，可以使用Type並選擇期望的建構式來建構物件，未來使用泛型建構物件時，便不用再憂愁如何取得型別本身了。

#### 補充：default

雖然說是補充，但`default`在實作泛型時也是常用的工具。顧名思義，`default`可以取得型別的預設值，如int為0、bool 為 false、其它Reference type皆為null。

> 注意！此處與 switch case 中的 default 不同：
>
> switch case 中的 default : 關鍵字 ( keyword )
>
> 其它的 default : 運算子 ( operator ) or 常值 ( literal )

以下是一個 default operator 及 literal 的例子：

```C#
Console.WriteLine(default(double));  // 0.0
Console.WriteLine(default(Complex)); // <0; 0>
List<int> arr = new() {1, 2, 3};
Console.WriteLine(arr is null); // False
arr = default;
Console.WriteLine(arr is null); // True
```

`default`可做為一元運算子，取得括號內型別的預設值，也可做為 literal，利用指派的方式將物件還原為預設值，特別常用於函式的參數預設值。

實作泛型方面，尤其是函式需要回傳泛型物件時，可透過 `default(T)` 的方式，來符合函式對回傳型別的規範，也可以藉此避免呼叫端產生 Exception 的情形，不論是在Debug或功能設計上，都非常好用。

### 小結

本來想在這篇一次講完Type和Reflection的，但礙於篇幅限制，又要再拆到下一篇了……QQ

下一篇會講解如何使用反射及相關應用，並對泛型進行優劣分析，有興趣的朋友再麻煩移駕到最後一篇！