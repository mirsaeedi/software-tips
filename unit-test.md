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

* Unit-tests are here to give us confidence in refactoring and modifying code. But not well-designed tests do quite the opposite, because each change to the code breaks lots of tests, and it makes developers to not be interested in improving the code. To reduce the coupling between tests and code, we need to make sure tests are not depenendent on implementation details of the code. Excess use of mocks and `mock.Verify` [couples](https://enterprisecraftsmanship.com/posts/most-important-tdd-rule/) tests to implementation details and impedes us from refactoring the code with confidence. Maintainable tests check whether the output is correct for specific input and it does not care about how the method under test is implemented.

* [Test doubles are only for extenal dependencies](https://enterprisecraftsmanship.com/posts/tdd-best-practices/). We should not mock dependencies if they are part of our code and do not communicate with the external world. Therefore, we should pass the actual dependencies to excersice all our code and the communication between components. Using mocks instead of actual classes prevents us from finding possible bugs that happen during interaction of components. Also, using mocks couples tests with implementation details which  makes tests fragile, as we need to change the test every time method signature changes.

* Only integration tests can give us confidence that the application we develop actually works as a whole. They are also a good substitute for mocks in the cases where you can’t test important business logic without involving external dependencies.

* With integration tests, check only a single happy path per application service method. Also, if there are any edge cases which cannot be covered with unit tests, check them as well.

* It takes place when you introduce additional code to your main code base in order to enable unit testing. Don’t introduce production code that doesn’t run in production. don’t expose state getters solely for satisfying a test. In the above example, the status field is not observable from the outside world and thus doesn’t belong to the class’s public API.

* such tests are not able to distinguish a bug from a legit refactoring. Structural Inspection is sometimes claimed to be able to prove the code base’s correctness. In reality, though, it sets your code in stone. The only thing it proves is that the SUT is implemented in one particular way. And that way may or may not be correct, you will need to introduce additional tests to verify that anyway. Tests that employ the Structural Inspection technique couple to the SUT’s implementation details and thus are fragile.

* Overall, try to constantly ask yourself a question: does this test verify some business requirement? If the answer is no, remove it. The most valuable tests are always the tests that have at least some connection to the business requirements your code base is ought to address.

* The most valuable tests are tests that verify the observable behavior as it seems to appear from the end user’s perspective. The closer you can get to this kind of verification, the better.

* There is an opinion that unit testing leads to a better design. I personally don’t think that unit testing in and of itself leads to anything. It’s true that your unit test suite can become a good litmus test which tells you there’s something wrong with the code base. If the code is hard to unit test, then it probably requires improvement. However, the sole existence of a unit test suite doesn’t provide any guarantees. I’ve seen many code bases which were a mess despite a good test coverage. Unit tests in and of themselves don’t lead to a better design.

* the single most important benefit of unit testing is confidence.
*  what is a valuable unit test? It is a test which:

Has a high chance of catching a regression bug.

Has a low chance of producing a false positive. A false positive is a situation where your test suite raises a false alarm: indicates an error, whereas, in the reality, everything works fine.

Provides fast feedback.

* We just need to shift our focus from hows of the SUT to its whats and verify the end result instead.

* The test pyramid is a way of thinking about how different kinds of automated tests should be used to create a balanced portfolio. Its essential point is that you should have many more low-level UnitTests than high level BroadStackTests running through a GUI.


* Even with good practices on writing them, end-to-end tests are more prone to non-determinism problems, which can undermine trust in them. In short, tests that run end-to-end through the UI are: brittle, expensive to write, and time consuming to run. So the pyramid argues that you should do much more automated testing through unit tests than you should through traditional GUI based testing.
* I always argue that high-level tests are there as a second line of test defense. If you get a failure in a high level test, not just do you have a bug in your functional code, you also have a missing or incorrect unit test.

* Test coverage is a useful tool for finding untested parts of a codebase. Test coverage is of little use as a numeric statement of how good your tests are.
* If you make a certain level of coverage a target, people will try to attain it. The trouble is that high coverage numbers are too easy to reach with low quality testing.
* Like most aspects of programming, testing requires thoughtfulness.
* The reason, of course, why people focus on coverage numbers is because they want to know if they are testing enough. Certainly low coverage numbers, say below half, are a sign of trouble. But high numbers don't necessarily mean much, and lead to ignorance-promoting dashboards. 
*  One sign you are testing too much is if your tests are slowing you down. If it seems like a simple change to code causes excessively long changes to tests, that's a sign that there's a problem with the tests. This may not be so much that you are testing too many things, but that you have duplication in your tests.
*  An object mother is a kind of class used in testing to help create example objects that you use for testing.
*  Many in our industry claim that any unit tests are better than none, but I disagree: a test suite can be a great asset, or it can be a great burden that contributes little. It depends on the quality of those tests, which seems to be determined by how well its developers have understood the goals and principles of unit testing. 
*  Avoid non-descriptive unit tests names such as Purchase() or OutOfStock(). Maintenance is hard if you don’t know what you’re trying to maintain.+
*  Name your unit tests clearly and consistently 
