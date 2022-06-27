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
