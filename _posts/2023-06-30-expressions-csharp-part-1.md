---
layout: post
title: Understanding Expressions in C# - Pt. 1
excerpt_separator: <!--more-->
---

I've been encountering expression trees regularly these days, particularly in using Entity Framework and LINQ Expressions.<!--more--> 

The syntax for expression trees can be overwhelming at first, particularly for beginners. To understand this type of syntax more, I found some sample code while reading through the [C# docs](https://learn.microsoft.com/en-us/dotnet/csharp/tour-of-csharp/program-building-blocks#virtual-override-and-abstract-methods).

Let's take a look at the sample code. Like the article explains, this example is similar to, but not related to the expression tree type.

```cs
public abstract class Expression
{
    public abstract double Evaluate(Dictionary<string, object> vars);
}

public class Constant : Expression
{
    double _value;
    
    public Constant(double value)
    {
        _value = value;
    }
    
    public override double Evaluate(Dictionary<string, object> vars)
    {
        return _value;
    }
}

public class VariableReference : Expression
{
    string _name;
    
    public VariableReference(string name)
    {
        _name = name;
    }
    
    public override double Evaluate(Dictionary<string, object> vars)
    {
        object value = vars[_name] ?? throw new Exception($"Unknown variable: {_name}");
        return Convert.ToDouble(value);
    }
}

public class Operation : Expression
{
    Expression _left;
    char _op;
    Expression _right;
    
    public Operation(Expression left, char op, Expression right)
    {
        _left = left;
        _op = op;
        _right = right;
    }
    
    public override double Evaluate(Dictionary<string, object> vars)
    {
        double x = _left.Evaluate(vars);
        double y = _right.Evaluate(vars);
        switch (_op)
        {
            case '+': return x + y;
            case '-': return x - y;
            case '*': return x * y;
            case '/': return x / y;
            
            default: throw new Exception("Unknown operator");
        }
    }
}
```

From the example shown above, the `Expression` class is a simple abstract class. It contains a single method `Evaluate` which accepts a dictionary and returns a double value. So immediately we know that the class is limited to working with doubles.

The three classes that follow, `Constant`, `VariableReference`, and `Operation` all inherit from the `Expression` base class. Based on their names, their purpose is clear. But let's take a look at each one a little more closely.

## Building Expression Trees

The `Constant` class represents a constant value. The value is assigned during construction, and it never changes. The `Evaluate` method simply returns the constant. Based on inheritance alone, we know that the three classes are all expressions, including the constant itself. By most definitions, even a single value itself is an expression.

In the next class, `VariableReference` is meant to refer to just that. It's an object that stores a string `_name`, which can be used to reference a value from a dictionary. The `Evaluate` method is overriden to return the `object`from the dictionary where `_name` is the key. If `_name` is not found then the method throws an exception.

The `Operation` class involves operands and an operator. In the case of our class, we're limited to an only left and right operand, and a `char` value for the operator. The operators themselves are limited to addition, subtraction, multiplication, and division. 

It's inside the `Operation` constructor where things get interesting. This constructor accepts two parameters of type `Expression`. At first, passing single values to each parameter seems appropriate. But operations themselves can be expressions. So can variable references. So although the operation class is limited to two operands and an operator, we can build an expression tree to represent more complex expressions. 

```cs
Constant ten = new Constant(10.0);
Constant fifteen = new Constant(15.0);
Operation added = new Operation(ten, '+', fifteen);

Dictionary<string, object> dict = new Dictionary<string, object>
{
    { "quarter", 0.25 },
    { "half", 0.5 }
};

VariableReference quarter = new VariableReference("quarter");
Operation quartered = new Operation(added, '*', quarter);
quartered.Evaluate(dict);
```
The sample code builds an expression tree from a series of five expressions: two constants, an operation, a variable reference, and a final operation called `quartered`. This final operation is the root of the expression tree, because it is the top-level expression whereby the entire operation begins. The above code can be expressed mathematically as follows:
```
(10.0 + 15.0) * 0.25
```
Although the `Operation` class only accepts two operands, we can build mathematical expressions with as many operands as we would like so long as the values passed to the `Operation` class are also of type `Expression`. 

Each class hints at the core definition of an expression. A single value is itself an expression. A reference to a value is also an expression, and any number of values with operands (an operation) also represent expressions. 

The root of the sample above lies in the `Operation` class. By passing in other operations as parameters to a single root operation, we can build complex expression trees using very simple base code.

{% include utterances.html %}
