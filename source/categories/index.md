---
title: categories
date: 2022-04-12 19:11:41
---

# 查询表达式是什么？
查询表达式是以查询语法表示的查询。 查询表达式是一流的语言构造。 它如同任何其他表达式一样，可以在 C# 表达式有效的任何上下文中使用。 查询表达式由一组用类似于 SQL 或 XQuery 的声明性语法所编写的子句组成。 每个子句进而包含一个或多个 C# 表达式，而这些表达式可能本身是查询表达式或包含查询表达式。

查询表达式必须以 from 子句开头，且必须以 select 或 group 子句结尾。 在第一个 from 子句与最后一个 select 或 group 子句之间，可以包含以下这些可选子句中的一个或多个：where、orderby、join、let，甚至是其他 from 子句。 还可以使用 into 关键字，使 join 或 group 子句的结果可以充当相同查询表达式中的其他查询子句的源。

# 查询变量
在 LINQ 中，查询变量是存储查询而不是查询结果的任何变量。 更具体地说，查询变量始终是可枚举类型，在 foreach 语句或对其 IEnumerator.MoveNext 方法的直接调用中循环访问时会生成元素序列。
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





# 参考文献

https://learn.microsoft.com/zh-cn/dotnet/csharp/language-reference/keywords/query-keywords

https://learn.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/linq/introduction-to-linq-queries