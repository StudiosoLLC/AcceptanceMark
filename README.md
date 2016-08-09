# AcceptanceMark

AcceptanceMark is a tool for running Acceptance Tests in Xcode, inspired by [Fitnesse](http://fitnesse.org/).

### Fitnesse advantages

* Easy to write business rules in tabular form in Markdown files.
* All shareholders can write Fitnesse tests.
* Convenient Test Report.

### Fitnesse disadvantages

* Does not integrate well with XCTest.
* Requires to run a separate server.
* Difficult to configure and run locally / on CI.

### The solution: AcceptanceMark

AcceptanceMark is the perfect tool to write Fitnesse-style acceptance tests, while integrating seamlessly with XCTest.

* Write tests in markdown with tables.
* Auto-generated XCTest boilerplate code for tests with **strong-typed input/outputs**. 
  * **You only write your business logic in the test runners**.
* Easy to integrate with both Unit Tests and UI Tests.

### How does this work?

Write your own test sets, like so:

```
image-tests.md

## Image Loading

| name:String   || loaded:Bool  |
| ------------- || ------------ |
| available.png || true         |
| missing.png   || false        |
```

_NOTE: The double-pipe syntax (`||`) is used as a delimiter between inputs and outputs. All test sets should have **exactly one** delimiter._

Run AcceptanceMark as an XCode pre-compilation phase, which generates all the required test harness:

```
/*
 * File Auto-Generated by AcceptanceMark - DO NOT EDIT
 * input file: image-tests.md
 * generated file: ImageTests_ImageLoadingTests.swift
 *
 * Test specification
 *
 * ## Image Loading
 *
 * | name:String   || loaded:Bool  |
 * | ------------- || ------------ |
 * | available.png || true         |
 * | missing.png   || false        |
 *
 */

import XCTest
import AcceptanceMark

struct ImageTests_ImageLoadingInput {
    let name: String
}

struct ImageTests_ImageLoadingResult: Equatable {
    let loaded: Bool
}

protocol ImageTests_ImageLoadingTestRunnable {
    func run(input: ImageTests_ImageLoadingInput) throws -> ImageTests_ImageLoadingResult
}


class ImageTests_ImageLoadingTests: XCTestCase {
    
    var testRunner: ImageTests_ImageLoadingTestRunnable!
    
    override func setUp() {
        // MARK: Implement the ImageTests_ImageLoadingTestRunner() class!
        testRunner = ImageTests_ImageLoadingTestRunner()
    }
    
    func testImageLoading_0() {
        
        let input = ImageTests_ImageLoadingInput(name: "available.png")
        let expected = ImageTests_ImageLoadingResult(loaded: true)
        let result = try! testRunner.run(input: input)
        XCTAssertEqual(expected, result)
    }
    
    func testImageLoading_1() {
        
        let input = ImageTests_ImageLoadingInput(name: "missing.png")
        let expected = ImageTests_ImageLoadingResult(loaded: false)
        let result = try! testRunner.run(input: input)
        XCTAssertEqual(expected, result)
    }
}

func == (lhs: ImageTests_ImageLoadingResult, rhs: ImageTests_ImageLoadingResult) -> Bool {
    return lhs.loaded == rhs.loaded
}
```

Finally, write your test runner:

```
// User generated file. Put your test runner implementation here.
class ImageTests_ImageLoadingTestRunner: ImageTests_ImageLoadingTestRunnable {

    func run(input: ImageTests_ImageLoadingInput) throws -> ImageTests_ImageLoadingResult {
        // Your business logic here
        return ImageTests_ImageLoadingResult(loaded: true)
    }
}
```

### Notes

* Note the functional style of the test runner. It is simply a method that takes a stronly-typed input value, and returns a strongly-typed output value. **No state, no side effects**.

* As it is common for XCTests to have a `setUp()` method that is typically used to initialise each test with some state. While this _will_ be supported in **AcceptanceMark**, a more **functional style is preferred and encouraged**.

## FAQ

* _**Q**: I want to be able to have more than one table for each `.md` file. Is this possible?_
* **A**: Yes, as long as the file is structured as **Heading**, **Table**, **Heading**, **Table**, **...**, **AcceptanceMark** will generate multiple swift test files by using a `<filename>_<heading>.swift` naming convention. This way, each table gets its own swift classes all in one file. Note that **heading names should be unique per-file**. Whitespaces and punctuation will be stripped from headings.

* _**Q**: I want to preload application data/state for each test in a table (this is done with builders in Fitnesse). Can I do that?_
* **A**: This is in the roadmap. While the specification for this may change, one possible way of doing this is by allowing more than one table for each Heading, with the convention that the **last** table represents the input/output set, while all previous tables represent data to be preloaded.
Until this is implemented, all preloading can be done directly in the test runner's `setUp()` method.

```
## Image Loading

| Country:String | Code:Bool |
| -------------- | --------- |
| United Kingdom | GB        |
| Italy          | IT        |
| Germany        | DE        |
 
| name:String   || loaded:Bool  |
| ------------- || ------------ |
| available.png || true         |
| missing.png   || false        |

```

* _**Q**: I want to preload a JSON file for all tests running on a given table. Can I do that?_
* **A**: You could do that directly by adding the JSON loading code directly in the test runner's `setUp()` method. For extra flexibility you could specify the JSON file name as an input parameter of your test set, and have your test runner load that file from the bundle.


## LICENSE

MIT License. See the [license file](LICENSE.md) for details.

