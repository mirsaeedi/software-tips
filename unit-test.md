# Unit test

* Art: Wrting good unit tests requires a different skill-set compared to regular coding.

* Fast: Unit tests should be fast to get the feedback as quick as possible, otherwise developers lose their interest in executing tests regularly. 

* All tests should be run randomly in parallel without a specific order. It allows to shorten the execution time of the test suit and get feedback quickly. Isolation, side-effect

* **AAA**: A unit test usually has three phases that are known as Arrange, Act and Assert, or simply AAA. In tests, seperate each section using a comment line.

* [**Constrained Non-Determinism**](https://blog.ploeh.dk/2009/03/05/ConstrainedNon-Determinism/): Try not to hard-code dummy values in tests. Instead, we can use well-defined, but random, input, because when input is random, 
we do not accidentally hard-code any assumptions. For example, for generating random strings, we can use `Guid.NewGuid().ToString()`.


* Few things are more non-deterministic than a call to the system clock. Each time you call it, you get a new result, and any tests that depend on it can thus change. Ask for all the todos due in the next hour, and you regularly get a different answe. Avoid using `DateTime.Now` or any other static method for getting time, because static methods are not mockable and time will chnage between test runs, and it might result in 
different non-deterministic behaviors. Instead, redesign your code, to get `DataTime` or `IDateTimeService` as a parameter. I'd argue for finding a way to use code analysis to detect any direct calls to the system clock and failing the build right there.

* Static is evil. Using static methods of other classes in your code, makes your code non-testable, e.g., `DateTime.Now`, `Guid.NewGuid()`, `Directory.Exists(string path)`, 
`Environment.GetEnvironmentVariable("PATH")`, or a static singleton instance of a custom class.

* Use `new` with caution in your code. It makes your code coupled to a specific concrete type, as a result, you cannot mock that dependency if it's required for testing. Consider
injecting the dependency through a parameter to receive it from the caller. It also makes your code more readable as it's dependencies are explicit.

* [Non-deterministic](https://martinfowler.com/articles/nonDeterminism.html) (flaky) tests, that sometimes pass and sometimes fail without any noticeable change in the code, tests, or environment, can completely destroy the value of an automated regression suite. Such tests fail, then you re-run them and they pass. Test failures for such tests are seemingly random. Quarantine them intially but fix them soon.

* The root cuase of non-determinism is twofold, side effects and using external resources. For example, if a test modifies a shared resource, e.g., file, database, or variable, that is used by another test, we might observe a different behavior each time we run the test suit as the test execution is concurrent and randomly ordered. Besides that, using external resources, such as time, 
network, file system, and database, might bring non-determinism to your tests, because consuming these resources might fail for a variety of reasons, e.g., timeout, permission error, or disk failure.

* Therefore I find it's really important to focus on keeping tests isolated. Properly isolated tests can be run in any sequence.

* One trick that's handy when you're using databases, is to conduct your tests inside a transaction, and then to rollback the transaction at the end of the test. That way the transaction manager cleans up for you, reducing the chance of errors.

* David Hansson introduced the concept of test-induced design damage introduced in a [series of discussion](http://martinfowler.com/articles/is-tdd-dead/) with Martin Fowler and Kent Beck. It basically says that damaging your code in inevitable when you make it testable. In other words, testablity introduces a degree of complexity to your code. For example, you must avoid using static methods of other classes in your code or you must pass all dependencies explicitly through the constructor/method parameters.

* Passing dependencies through parameters make your tests fragile, because each change to the constructor/method signature breaks all associated tests. It's a [pain in the neck](https://enterprisecraftsmanship.com/posts/test-induced-design-damage-or-why-tdd-is-so-painful/) and makes your test solution less maintainable.

* Writing and maintaining unit-tests is expensive. Hence, we should be careful where we invest our time and how we can have best return of investment. Khorikov [advises](https://enterprisecraftsmanship.com/posts/painless-tdd/) us to only unit-test domain logic and business-critical parts of the code. Trival code or a code that directly communicates with external dependencies, e.g., network or database, does not need to be unit-tested. The goal of unit-testing is not 100% coverage. The goal is to ensure refactoring and making new changes does not introduce unexpected bugs.

* Unit tests with long list of arrangements are less maintainable, because a change to the method signature of the SUT or any of depenencies breaks the tests.


