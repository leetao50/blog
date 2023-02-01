---
title: post
date: 2022-12-17 23:26:42
tags:
---
# 查询操作的三个部分
所有 LINQ 查询操作都由以下三个不同的操作组成：

1. 获取数据源。
2. 创建查询。
3. 执行查询。

下面的示例演示如何用源代码表示查询操作的三个部分。 为方便起见，此示例将一个整数数组用作数据源；但其中涉及的概念同样适用于其他数据源。 本主题的其余部分也会引用此示例。
~~~C#
class IntroToLINQ
{
    static void Main()
    {
        // The Three Parts of a LINQ Query:
        // 1. Data source.
        int[] numbers = new int[7] { 0, 1, 2, 3, 4, 5, 6 };

        // 2. Query creation.
        // numQuery is an IEnumerable<int>
        var numQuery =
            from num in numbers
            where (num % 2) == 0
            select num;

        // 3. Query execution.
        foreach (int num in numQuery)
        {
            Console.Write("{0,1} ", num);
        }
    }
}
~~~

## 数据源

上例中，数据源是一个数组，因此它隐式支持泛型 IEnumerable<T> 接口。 这一事实意味着该数据源可以用 LINQ 进行查询。 查询在 foreach 语句中执行，且 foreach 需要 IEnumerable 或 IEnumerable<T>。 支持 IEnumerable<T> 或派生接口（如泛型 IQueryable<T>）的类型称为可查询类型 。

可查询类型不需要进行修改或特殊处理就可以用作 LINQ 数据源。 如果源数据还没有作为可查询类型出现在内存中，则 LINQ 提供程序必须以此方式表示源数据。 例如，LINQ to XML 将 XML 文档加载到可查询的 XElement 类型中：

~~~C#
// Create a data source from an XML document.
// using System.Xml.Linq;
XElement contacts = XElement.Load(@"c:\myContactList.xml");
~~~
有关如何创建特定类型的数据源的详细信息，请参阅各种 LINQ 提供程序的文档。 但基本规则很简单：LINQ 数据源是支持泛型 IEnumerable<T> 接口或从中继承的接口的任意对象。

## 查询

查询指定要从数据源中检索的信息。 查询还可以指定在返回这些信息之前如何对其进行排序、分组和结构化。 查询存储在查询变量中，并用查询表达式进行初始化。 为使编写查询的工作变得更加容易，C# 引入了新的查询语法。

上一个示例中的查询从整数数组中返回所有偶数。 该查询表达式包含三个子句：from、where 和 select。 （如果熟悉 SQL，会注意到这些子句的顺序与 SQL 中的顺序相反。）from 子句指定数据源，where 子句应用筛选器，select 子句指定返回的元素的类型。

### 查询执行

#### 延迟执行
如前所述，查询变量本身只存储查询命令。 查询的实际执行将推迟到在 foreach 语句中循环访问查询变量之后进行。 此概念称为延迟执行，下面的示例对此进行了演示：

~~~C#
//  Query execution.
foreach (int num in numQuery)
{
    Console.Write("{0,1} ", num);
}
~~~
foreach 语句也是检索查询结果的地方。 例如，在上一个查询中，迭代变量 num 保存了返回的序列中的每个值（一次保存一个值）。

由于查询变量本身从不保存查询结果，因此可以根据需要随意执行查询。 例如，可以通过一个单独的应用程序持续更新数据库。 在应用程序中，可以创建一个检索最新数据的查询，并可以按某一时间间隔反复执行该查询以便每次检索不同的结果。

#### 强制立即执行

对一系列源元素执行聚合函数的查询必须首先循环访问这些元素。 Count、Max、Average 和 First 就属于此类查询。 由于查询本身必须使用 foreach 以便返回结果，因此这些查询在执行时不使用显式 foreach 语句。 另外还要注意，这些类型的查询返回单个值，而不是 IEnumerable 集合。 下面的查询返回源数组中偶数的计数：

~~~C#
var evenNumQuery =
    from num in numbers
    where (num % 2) == 0
    select num;

int evenNumCount = evenNumQuery.Count();
~~~

要强制立即执行任何查询并缓存其结果，可调用 ToList 或 ToArray 方法。
~~~C#
List<int> numQuery2 =
    (from num in numbers
     where (num % 2) == 0
     select num).ToList();

// or like this:
// numQuery3 is still an int[]

var numQuery3 =
    (from num in numbers
     where (num % 2) == 0
     select num).ToArray();
~~~

# 查询表达式基础

## 查询是什么及其作用是什么？
查询是一组指令，描述要从给定数据源检索的数据以及返回的数据应具有的形状和组织。 查询与它生成的结果不同。

从应用程序的角度来看，原始源数据的特定类型和结构并不重要。 应用程序始终将源数据视为 `IEnumerable<T>` 或 `IQueryable<T>` 集合。 

对于此源序列，查询可能会执行三种操作之一：

+ 检索元素的子集以生成新序列，而不修改各个元素。
~~~C#
IEnumerable<int> highScoresQuery =
    from score in scores
    where score > 80
    orderby score descending
    select score;
~~~
+ 如前面的示例所示检索元素的序列，但是将它们转换为新类型的对象.下面的示例演示从 int 到 string 的投影。 请注意 highScoresQuery 的新类型。
~~~C#
IEnumerable<string> highScoresQuery2 =
    from score in scores
    where score > 80
    orderby score descending
    select $"The score is {score}";
~~~
+ 检索有关源数据的单独值
  + 与特定条件匹配的元素数。
  + 具有最大或最小值的元素。
  + 与某个条件匹配的第一个元素，或指定元素集中特定值的总和。 例如，下面的查询从 scores 整数数组返回大于 80 的分数的数量：
~~~C#
int highScoreCount = (
    from score in scores
    where score > 80
    select score
).Count();
~~~
在前面的示例中，请注意在调用 Count 方法之前，在查询表达式两边使用了括号。 也可以通过使用新变量存储具体结果，来表示此行为。 这种方法更具可读性，因为它使存储查询的变量与存储结果的查询分开。
~~~C#
IEnumerable<int> highScoresQuery3 =
    from score in scores
    where score > 80
    select score;

int scoreCount = highScoresQuery3.Count();
~~~
在上面的示例中，查询在 Count 调用中执行，因为 Count 必须循环访问结果才能确定 highScoresQuery 返回的元素数。

## 查询表达式是什么？
查询表达式是以查询语法表示的查询。它如同任何其他表达式一样，可以在 C# 表达式有效的任何上下文中使用。

查询表达式必须以 from 子句开头，且必须以 select 或 group 子句结尾。 
在第一个 from 子句与最后一个 select 或 group 子句之间，可以包含以下这些可选子句中的一个或多个：where、orderby、join、let，甚至是其他 from 子句。 
还可以使用 into 关键字，使 join 或 group 子句的结果可以充当相同查询表达式中的其他查询子句的源。

## 查询变量
查询变量是存储查询而不是查询结果的任何变量。
下面的代码示例演示一个简单查询表达式，它具有一个数据源、一个筛选子句、一个排序子句并且不转换源元素。 该查询以 select 子句结尾。
~~~C#
// Data source.
int[] scores = { 90, 71, 82, 93, 75, 82 };

// Query Expression.
IEnumerable<int> scoreQuery = //query variable
    from score in scores //required
    where score > 80 // optional
    orderby score descending // optional
    select score; //must end with select or group

// Execute the query to produce the results
foreach (int testScore in scoreQuery)
{
    Console.WriteLine(testScore);
}

// Output: 93 90 82 82
~~~
在上面的示例中，scoreQuery 是查询变量，它有时仅仅称为查询。 查询变量不存储在 foreach 循环生成中的任何实际结果数据。 并且当 foreach 语句执行时，查询结果不会通过查询变量 scoreQuery 返回。 而是通过迭代变量 testScore 返回。 scoreQuery 变量可以在另一个 foreach 循环中进行循环访问。 只要既没有修改它，也没有修改数据源，便会生成相同结果。

查询变量可以存储采用查询语法、方法语法或是两者的组合进行表示的查询。 在以下示例中，queryMajorCities 和 queryMajorCities2 都是查询变量：
~~~C#
//Query syntax
IEnumerable<City> queryMajorCities =
    from city in cities
    where city.Population > 100000
    select city;

// Method-based syntax
IEnumerable<City> queryMajorCities2 = cities.Where(c => c.Population > 100000);
~~~
## 查询变量的显式和隐式类型化
通常提供查询变量的显式类型以便显示查询变量与 select 子句之间的类型关系。 但是，还可以使用 var 关键字指示编译器在编译时推断查询变量（或任何其他局部变量）的类型

## 开始查询表达式
查询表达式必须以 from 子句开头。 它指定数据源以及范围变量。 范围变量表示遍历源序列时，源序列中的每个连续元素。 范围变量基于数据源中元素的类型进行强类型化。 在下面的示例中，因为 countries 是 Country 对象的数组，所以范围变量也类型化为 Country。 因为范围变量是强类型，所以可以使用点运算符访问该类型的任何可用成员。
~~~C#
IEnumerable<Country> countryAreaQuery =
    from country in countries
    where country.Area > 500000 //sq km
    select country;
~~~

范围变量一直处于范围中，直到查询使用分号或 continuation 子句退出。

查询表达式可能会包含多个 from 子句。 在源序列中的每个元素本身是集合或包含集合时，可使用其他 from 子句。 例如，假设具有 Country 对象的集合，其中每个对象都包含名为 Cities 的 City 对象集合。 若要查询每个 Country 中的 City 对象，请使用两个 from 子句，如下所示：

~~~C#
IEnumerable<City> cityQuery =
    from country in countries
    from city in country.Cities
    where city.Population > 10000
    select city;
~~~
## 结束查询表达式
查询表达式必须以 group 子句或 select 子句结尾。

### group 子句
使用 group 子句可生成按指定键组织的组的序列。 键可以是任何数据类型。 例如，以下查询会创建包含一个或多个 Country 对象，并且其关键值是数值为国家/地区名称首字母的 char 类型。

~~~C#
var queryCountryGroups =
    from country in countries
    group country by country.Name[0];
~~~

### select 子句
使用 select 子句可生成所有其他类型的序列。 简单 select 子句只生成类型与数据源中包含的对象相同的对象的序列。 在此示例中，数据源包含 Country 对象。 orderby 子句只按新顺序对元素进行排序，而 select 子句生成重新排序的 Country 对象的序列。

~~~C#
IEnumerable<Country> sortedQuery =
    from country in countries
    orderby country.Area
    select country;
~~~

select 子句可以用于将源数据转换为新类型的序列。 此转换也称为投影。 在下面的示例中，select 子句对只包含原始元素中的字段子集的匿名类型序列进行投影。 请注意，新对象使用对象初始值设定项进行初始化。

~~~C#
// Here var is required because the query
// produces an anonymous type.
var queryNameAndPop =
    from country in countries
    select new
    {
        Name = country.Name,
        Pop = country.Population
    };
~~~

+ 选择每个源元素的子集
有两种主要方法来选择源序列中每个元素的子集：

若要仅选择源元素的一个成员，请使用点操作。 在以下示例中，假设 Customer 对象包含多个公共属性，包括名为 City 的字符串。 在执行时，此查询将生成字符串的输出序列。

~~~C#
var query = from cust in Customers  
            select cust.City;  
~~~

若要创建包含多个源元素属性的元素，可以使用带有命名对象或匿名类型的对象初始值设定项。 以下示例演示如何使用匿名类型封装每个 Customer 元素的两个属性：

~~~C#
var query = from cust in Customer  
            select new {Name = cust.Name, City = cust.City};  
~~~

+ 将内存中对象转换为 XML
LINQ 查询可以轻松地在内存中数据结构、SQL 数据库、ADO.NET 数据集和 XML 流或文档之间转换数据。 以下示例将内存中数据结构中的对象转换为 XML 元素。
~~~C#
class XMLTransform
{
    static void Main()
    {
        // Create the data source by using a collection initializer.
        // The Student class was defined previously in this topic.
        List<Student> students = new List<Student>()
        {
            new Student {First="Svetlana", Last="Omelchenko", ID=111, Scores = new List<int>{97, 92, 81, 60}},
            new Student {First="Claire", Last="O’Donnell", ID=112, Scores = new List<int>{75, 84, 91, 39}},
            new Student {First="Sven", Last="Mortensen", ID=113, Scores = new List<int>{88, 94, 65, 91}},
        };

        // Create the query.
        var studentsToXML = new XElement("Root",
            from student in students
            let scores = string.Join(",", student.Scores)
            select new XElement("student",
                       new XElement("First", student.First),
                       new XElement("Last", student.Last),
                       new XElement("Scores", scores)
                    ) // end "student"
                ); // end "Root"

        // Execute the query.
        Console.WriteLine(studentsToXML);

        // Keep the console open in debug mode.
        Console.WriteLine("Press any key to exit.");
        Console.ReadKey();
    }
}
~~~
此代码生成以下 XML 输出：
~~~XNL
<Root>  
  <student>  
    <First>Svetlana</First>  
    <Last>Omelchenko</Last>  
    <Scores>97,92,81,60</Scores>  
  </student>  
  <student>  
    <First>Claire</First>  
    <Last>O'Donnell</Last>  
    <Scores>75,84,91,39</Scores>  
  </student>  
  <student>  
    <First>Sven</First>  
    <Last>Mortensen</Last>  
    <Scores>88,94,65,91</Scores>  
  </student>  
</Root>
~~~

### 使用“into”延续
可以在 select 或 group 子句中使用 into 关键字创建存储查询的临时标识符。 如果在分组或选择操作之后必须对查询执行其他查询操作，则可以这样做。 在下面的示例中，countries 按 1000 万范围，根据人口进行分组。 创建这些组之后，附加子句会筛选出一些组，然后按升序对组进行排序。 若要执行这些附加操作，需要由 countryGroup 表示的延续。

~~~C#
// percentileQuery is an IEnumerable<IGrouping<int, Country>>
var percentileQuery =
    from country in countries
    let percentile = (int)country.Population / 10_000_000
    group country by percentile into countryGroup
    where countryGroup.Key >= 20
    orderby countryGroup.Key
    select countryGroup;

// grouping is an IGrouping<int, Country>
foreach (var grouping in percentileQuery)
{
    Console.WriteLine(grouping.Key);
    foreach (var country in grouping)
    {
        Console.WriteLine(country.Name + ":" + country.Population);
    }
}
~~~

## 筛选、排序和联接
在开头 from 子句与结尾 select 或 group 子句之间，所有其他子句（where、join、orderby、from、let）都是可选的。 任何可选子句都可以在查询正文中使用零次或多次。

### where 子句
使用 where 子句可基于一个或多个谓词表达式，从源数据中筛选出元素。 以下示例中的 where 子句具有一个谓词及两个条件。

~~~C#
IEnumerable<City> queryCityPop =
    from city in cities
    where city.Population < 200000 && city.Population > 100000
    select city;
~~~
### orderby 子句
使用 orderby 子句可按升序或降序对结果进行排序。 还可以指定次要排序顺序。 下面的示例使用 Area 属性对 country 对象执行主要排序。 然后使用 Population 属性执行次要排序。

~~~C#
IEnumerable<Country> querySortedCountries =
    from country in countries
    orderby country.Area, country.Population descending
    select country;
~~~

### join 子句

使用 join 子句可基于每个元素中指定的键之间的相等比较，将一个数据源中的元素与另一个数据源中的元素进行关联和/或合并。

联接操作是对元素属于不同类型的对象序列执行。 联接了两个序列之后，必须使用 select 或 group 语句指定要存储在输出序列中的元素。 

还可以使用匿名类型将每组关联元素中的属性合并到输出序列的新类型中。 下面的示例关联其 Category 属性与 categories 字符串数组中一个类别匹配的 prod 对象。 筛选出 Category 与 categories 中的任何字符串均不匹配的产品。select 语句投影属性取自 cat 和 prod 的新类型。

~~~C#
var categoryQuery =
    from cat in categories
    join prod in products on cat equals prod.Category
    select new
    {
        Category = cat,
        Name = prod.Name
    };
~~~

### let 子句
使用 let 子句可将表达式（如方法调用）的结果存储在新范围变量中。 在下面的示例中，范围变量 firstName 存储 Split 返回的字符串数组的第一个元素。

~~~C#
string[] names = { "Svetlana Omelchenko", "Claire O'Donnell", "Sven Mortensen", "Cesar Garcia" };
IEnumerable<string> queryFirstNames =
    from name in names
    let firstName = name.Split(' ')[0]
    select firstName;

foreach (string s in queryFirstNames)
{
    Console.Write(s + " ");
}

//Output: Svetlana Claire Sven Cesar
~~~

## 查询表达式中的子查询

查询子句本身可能包含查询表达式，这有时称为子查询。 每个子查询都以自己的 from 子句开头，该子句不一定指向第一个 from 子句中的相同数据源。 例如，下面的查询演示在 select 语句用于检索分组操作结果的查询表达式。

~~~C#
var queryGroupMax =
    from student in students
    group student by student.Year into studentGroup
    select new
    {
        Level = studentGroup.Key,
        HighestScore = (
            from student2 in studentGroup
            select student2.ExamScores.Average()
        ).Max()
    };
~~~
