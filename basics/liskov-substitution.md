# Liskov Substitution Principle 

> Ability to replace any instance of a parent class with an instance of one of its child classes without negative side effects 

## Example

```swift
import XCTest

class Shout {
    func talk(_ phrase: String) -> String {
        return "\(phrase.uppercased())!!!!"
    }
}

class Whipser: Shout {
    override func talk(_ phrase: String) -> String {
        return phrase.lowercased()
    }
}

class MyTests: XCTestCase {
    let phrase = "hello"
    let shoutedPhrase = "HELLO!!!!"

    func testShoutingClass() {
        let shouting = Shout()
        XCTAssertEqual(shouting.talk(phrase), shoutedPhrase)
    }

    func testWhipserClass() {
        let whipser = Whipser()
        /// Whiper class inherits from Shout, but breaks funtionality of a shouting class by not shouting anymore...
        XCTAssertEqual(whipser.talk(phrase), shoutedPhrase)
    }
}

MyTests.defaultTestSuite.run()

```
