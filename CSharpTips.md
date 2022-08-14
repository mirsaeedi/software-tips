# Tips and Tricks

* Use coalesce expression instead of conditional expression. [Link](https://github.com/JosefPihrt/Roslynator/blob/main/docs/analyzers/RCS1084.md)
* Disposable Pattern. [Link](https://docs.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1063)
* Use switch pattern matching
```C#
public readonly struct Point
{
    public Point(int x, int y) => (X, Y) = (x, y);
    
    public int X { get; }
    public int Y { get; }
}

static Point Transform(Point point) => point switch
{
    { X: 0, Y: 0 }                    => new Point(0, 0),
    { X: var x, Y: var y } when x < y => new Point(x + y, y),
    { X: var x, Y: var y } when x > y => new Point(x - y, y),
    { X: var x, Y: var y }            => new Point(2 * x, 2 * y),
    _ => null, // discard pattern https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#discard-pattern
};
```
* `var` pattern: A var pattern is useful when you need a temporary variable within a Boolean expression to hold the result of intermediate calculations. You can also use a var pattern when you need to perform additional checks in when case guards of a switch expression or statement, as the following example shows:
```C#
static bool IsAcceptable(int id, int absLimit) =>
    SimulateDataFetch(id) is var results 
    && results.Min() >= -absLimit 
    && results.Max() <= absLimit;
```
* `Property` pattern: 
```C#
static bool IsConferenceDay(DateTime date) => date is { Year: 2020, Month: 5, Day: 19 or 20 or 21 };
```


# C# Conding Conventions
Coding conventions serve the following purposes:

* They create a consistent look to the code, so that readers can focus on content, not layout.
* They enable readers to understand the code more quickly by making assumptions based on previous experience.
* They facilitate copying, changing, and maintaining the code.

## Links:

* [C# Coding Conventions](https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions)
* [.NET Runtime C# Coding Style](https://github.com/dotnet/runtime/blob/main/docs/coding-guidelines/coding-style.md)
* [ASP.NET C# Coding Guideline](https://github.com/dotnet/aspnetcore/wiki/Engineering-guidelines#coding-guidelines)

## Gist

* When writing positional records, use pascal casing for parameters as they're the public properties of the record.

```C#
public record PhysicalAddress(
    string Street,
    string City,
    string StateOrProvince,
    string ZipCode);
```

* Use camel casing ("camelCasing") when naming private or internal fields, and prefix them with _.
* When working with static fields that are private or internal, use the s_ prefix and for thread static use t_.
* Use parentheses to make clauses in an expression apparent, as shown in the following code.
```C#
if ((val1 > val2) && (val1 > val3))
{
    // Take appropriate action.
}
```
* Place the comment on a separate line, not at the end of a line of code.

* Begin comment text with an uppercase letter.

* End comment text with a period.

*  Insert one space between the comment delimiter (//) and the comment text, as shown in the following example.

```C#
// The following declaration creates a query. It does not run
// the query.
```
* Use implicit typing for local variables when the type of the variable is obvious from the right side of the assignment, or when the precise type is not important.
* Don't rely on the variable name to specify the type of the variable. It might not be correct.
* Use implicit typing to determine the type of the loop variable in for loops.
* Don't use implicit typing to determine the type of the loop variable in foreach loops. In most cases, the type of elements in the collection isn't immediately obvious. The collection's name shouldn't be solely relied upon for inferring the type of its elements.
* To avoid exceptions and increase performance by skipping unnecessary comparisons, use && instead of & and || instead of | when you perform comparisons
* Public fields should be used sparingly and should use PascalCasing with no prefix when used.
* We avoid this. unless absolutely necessary.
* We always specify the visibility, even if it's the default (e.g. private string _foo not string _foo). Visibility should be the first modifier (e.g. public abstract not abstract public).
* Avoid more than one empty line at any time. For example, do not have two blank lines between members of a type.
* Avoid spurious free spaces. For example avoid if (someVar == 0)..., where the dots mark the spurious free spaces. Consider enabling "View White Space (Ctrl+R, Ctrl+W)" or "Edit -> Advanced -> View White Space" if using Visual Studio to aid detection.
* We use PascalCasing to name all our constant local variables and fields. 
* Make all internal and private types static or sealed unless derivation from them is required. As with any implementation detail, they can be changed if/when derivation is required in the future.
* Windows uses \ and OS X and Linux use / to separate directories. Instead of hard-coding either type of slash, use Path.Combine() or Path.DirectorySeparatorChar. If this is not possible (such as in scripting), use a forward slash. Windows is more forgiving than Linux in this regard.
* By default all async methods must have the Async suffix.
* Passing cancellation tokens is done with an optional parameter with a value of default(CancellationToken), which is equivalent to CancellationToken.None (one of the few places that we use optional parameters). 
* The class name of an extension method container (also known as a "sponsor type") should generally follow the pattern of <Feature>Extensions, <Target><Feature>Extensions, or <Feature><Target>Extensions.
* The contents of every unit test should be split into three distinct stages, optionally separated by these comments.
 ```
// Arrange  
// Act  
// Assert     
 ```
* Always specify an EventId. Include a numeric ID and a name. The name should be a PascalCasedCompoundWord (i.e. no spaces, and each "word" within the name starts with a capital letter).
    * In production code, use "pre-compiled logging functions".
    * Prefer defining pre-compiled messages in a static class named Log that is a nested class within the class you are logging from. Messages that are used by multiple components can be defined in a shared class (but this is discouraged).
    * Consider separating the Log nested class into a separate file by using partial classes. Name the additional file [OriginalClassName].Log.cs.
    
