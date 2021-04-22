# Unit test

* Art: Wrting good unit tests requires a different skill-set compared to regular coding.

* Fast: Unit tests should be fast to get the feedback as quick as possible, otherwise developers lose their interest in executing tests regularly. 

* All tests should be run randomly in parallel without a specific order. It allows to shorten the execution time of the test suit and get feedback quickly. Isolation, side-effect

* **AAA**: A unit test usually has three phases that are known as Arrange, Act and Assert, or simply AAA. In tests, seperate each section using a comment line.

* [**Constrained Non-Determinism**](https://blog.ploeh.dk/2009/03/05/ConstrainedNon-Determinism/): Try not to hard-code dummy values in tests. Instead, we can use well-defined, but random, input, because when input is random, 
we do not accidentally hard-code any assumptions. For example, for generating random strings, we can use `Guid.NewGuid().ToString()`.

* Avoid using `DateTime.Now` or any other static method for getting time, because static methods are not mockable and time will chnage between test runs, and it might result in 
different non-deterministic behaviors. Instead, redesign your code, to get `DataTime` or `IDateTimeService` as a parameter.

* Static is evil. Using static methods of other classes in your code, makes your code non-testable, e.g., `DateTime.Now`, `Guid.NewGuid()`, `Directory.Exists(string path)`, 
`Environment.GetEnvironmentVariable("PATH")`, or a static singleton instance of a custom class.

* Use `new` with caution in your code. It makes your code coupled to a specific concrete type, as a result, you cannot mock that dependency if it's required for testing. Consider
injecting the dependency through a parameter to receive it from the caller. It also makes your code more readable as it's dependencies are explicit.

* [Non-deterministic](https://martinfowler.com/articles/nonDeterminism.html) (flaky) tests, that sometimes pass and sometimes fail without any noticeable change in the code, tests, or environment, can completely destroy the value of an automated regression suite. Such tests fail, then you re-run them and they pass. Test failures for such tests are seemingly random. Quarantine them intially but fix them soon.

* The root cuase of non-determinism is twofold, side effects and using external resources. For example, if a test modifies a shared resource, e.g., file, database, or variable, that is used by another test, we might observe a different behavior each time we run the test suit as the test execution is concurrent and randomly ordered. Besides that, using external resources, such as time, 
network, file system, and database, might bring non-determinism to your tests, because consuming these resources might fail for a variety of reasons, e.g., timeout, permission error, or disk failure.
