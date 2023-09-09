---
layout: post
title: Understanding Expressions in C# - Pt. 2
excerpt_separator: <!--more-->
---

In my [previous post](https://ajoh504.github.io/2023/06/30/expressions-csharp-part-1.html), we looked at a simple example of expression trees. As the name implies, expression trees allow us to write code in a tree-like structure. <!--more-->Each expression acts as a node that interacts with other nodes in the tree.<!--more-->

To dig a little further, let's pick apart a method called `GetRange`, which allows the caller to filter data from a data set. `GetRange` uses the `Expression<TDelegate>` type. It will also use Entity Framework via a `DbSet<TEntity>` object:

```cs
public List<TEntity> GetRange(
    Expression<Func<TEntity, bool>> predicate,
    ) 
{
    IQueryable<TEntity> query = _dbSet.Where(predicate);
    return query.ToList();
}
```

### Using the Expression Type

The `predicate` parameter is the focal point. It's what allows the caller to determine how the results should be filtered. Let's take a look at the type, which at first glance is a bit verbose.

`Expression<Func<TEntity, bool>>` represents the type that must be passed to the `GetRange` method. For the usage, this type is effectively a delegate. From within the delegate, `TEntity` represents the type that must be contained in the data set. And the delegate must return true or false. 

To use this method for filtering, we can write a simple delegate or `Func` that can be passed to `GetRange`. The delegate will be passed to the `Where` method, which determines how the data set is filtered. 

### Building the Tree

The `predicate` delegate will be the expression tree. Let's look at an example of one such delegate that can be passed to the `GetRange` method.

```
public List<TEntity> GetMonitoredFiles(List<string> filePaths)
    => _dataSet.GetRange(f => filePaths.Contains<string>(f.Path));
```

The `GetMonitoredFiles` method above takes a collection `filePaths` as an argument. The method returns a new collection where any item contained in both `filePaths` and the data set are added to the return value. `f => filePaths.Contains<string>(f.Path)` is the delegate that will be passed to `GetRange`.

### Examining the Tree

In this example, the expression tree is fairly simple. The lambda itself is the root node, or the top-level node, of the expression tree: `f => filePaths.Contains<string>(f.Path)`. The tree only contains two sub nodes. 

`f` is the first sub node. It represents the lambda parameter. It can be thought of as a placeholder for the input value that the expression body will operate on. When the lambda is used against the data set, `f` acts as the parameter for each entry in the data set. It's also important to know that each argument that `f` represents is not evaluated until the lambda itself is finally called. 

`filePaths.Contains<string>(f.Path)` is the second sub node. It is the expression body of the lambda and it represents the actual function argument. When it is finally passed to the `GetRange` method, it will be used to filter data from the data set. In this case, the data is filtered by searching through `filePaths`. Every entry in the data set, represented as `f`, is passed to `filePaths.Contains()`. If `filePaths` contains `f.Path`, then it is added to the return value.

This is a small example of how expression trees can be used in C#. As I continue writing code, I'm finding that using LINQ and Entity Framework can be a lot of fun. In particular, these two libraries have exposed me to lambda expressions, something that I think will benefit my coding for the long term.
