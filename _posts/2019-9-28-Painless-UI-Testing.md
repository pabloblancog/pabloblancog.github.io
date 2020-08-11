---
layout: post
title: Painless UI Testing
---

UI Testing is a great thing to keep apps maintainable and reliable during its whole lifecycle, but sometimes could become a big pain.

When test cases don't follow best practices, they become difficult to maintain when the test plans grow.
Some good practices can be folloed to avoid these issues:

#### - Test case naming
#### - Test case structure
#### - UI elements: Access and interaction
#### - Reusing code
#### - Better assertions

### Test case naming
Test cases names must be intuitive and understandable. A test case name can follow a syntax like this:
```swift
func test_<what_to_test>_<conditions_of_test>() {
    ...
}
```
For example, a test case to test sending the message *Hello*:
```swift
func test_sendMessage_hello() {
    ...
}
```
  
### Test case structure
Test cases should follow a fixed structure, so the test case can be easily followed step by step. I propose the Gherkin structure:
```
Given the scenario X
when I make the action Y
then I got the result Z
```
In a test case that sends the message *Hello*:
```swift
func test_sendMessage_hello() {
    // Given
    let messageToSend = "Hello"
    // When
    app.textfields["Send something..."].tap()
    app.textfields["Send something..."].type("Hello")
    app.buttons["Send"].tap()
    // Then
    XCTAssert(app.labels["result"].text == "How are you?")
}
```

### UI elements: Access and interaction
When you record a tap inside a textfield using the *recording feature* on Xcode, you will end up with something like this:
```swift
app.textfields["Send something..."].tap()
```
Xcode identifies the textfield by its placeholder (*Send something*). If the placeholder changes in a later code update the test will stop working.
To avoid this kind of scenarios, a valid solution is using **accessibility identifiers**.

> **Accessibility identifiers** helps disabled people using apps identifying elements to be used by *Voice Over*

For UI testing, they are interesting as they allow us to identify elements in tests.
In the viewController class related to the test, some identifiers can be defined this way:
```swift
class ViewController: UIViewController {

    func viewDidLoad() {
    }
    
    @IBOutlet weak var inputTextField: UITextField! {
        didSet {
            inputTextField.accessibilityIdentifier = "input_text_field"
        }
    }
    @IBOutlet weak var sendButton: UIButton! {
        didSet {
            sendButton.accessibilityIdentifier = "send_button"
        }
    }
    @IBOutlet weak var resultLabel: UILabel! {
        didSet {
            resultLabel.accessibilityIdentifier = "result_label"
        }
    }
}
```

On XCUITests, an element on the UI can be accessed by its identifier from an array that includes every UI element of a kind on the current visible screen of the app: app.buttons, app.textfields, app.labels, etc.:
```swift
func test_sendMessage_hello() {
    ...
    // When
    app.textfields["input_text_field"].tap()
    app.textfields["input_text_field"].type("Hello")
    app.buttons["send_button"].tap()
    // Then
    XCTAssert(app.labels["result_label"].text == "How are you?")
}
```

### Reusing code

### - Reusing variables

For reusing UI element variables through different test cases, they can be initialized on the XCTestCase class setUp method:
```swift
class UITests: XCTestCase {
    var app: XCUIApplication!
    var inputTextField: XCUIElement!
    var sendButton: XCUIElement!
    var resultLabel: XCUIElement!
    override func setUp() {
        app = XCUIApplication()
        inputTextField = app.textfields["input_text_field"]
        sendButton = app.buttons["send_button"]
        resultLabel = app.labels["result_label"]
    }

    func test_sendMessage_hello() {
        // Given
        inputTextField.text = ""
        let messageToSend = "Hello"
        // When
        inputTextField.tap()
        inputTextField.type("Hello")
        sendButton.tap()
        // Then
        XCTAssert(resultLabel)
        XCTAssert(resultLabel.text == "How are you?")
    }
}

```

### - Reusing methods
Sometimes we need to check different cases for a single method.
If we replicate the same instructions in every test case, the code become duplicated.
For reusing some code between test cases, we can use helper methods. For example, for the next test:

```swift
func test_sendMessage_hello() {
    ...
    // When
    inputTextField.tap()
    inputTextField.type("Hello")
    sendButton.tap()
    ...
}
```

We can create the next helper methods, allowing reusing *typing* and *tapping* features:

```swift
func typeTextOnInputBar(_ text: String) {
    inputTextView.tap()
    inputTextView.typeText(text)
}
func tapSendButton() {
    sendButton.tap()
}
```
For specific cases, we can join these two methods into one:
```swift
func sendMessage(_ message: String) {
    typeTextOnInputBar(message)
    tapSendButton()
}
```
So the test case will become:
```swift
func test_sendMessage_Hello() {
    ...
    // When
    sendMessage("Hello")
    ...
}
```
> Remember to avoid helper methods names starting by `test_` as they will be considered a test case by Xcode

### Better assertions
When checking whether everything in the test case went as expected, we will use `XCTAssert`.

> XCTAssert statements indicate whether a condition is OK or it is not.

An XCTAssert statement is composed like this:
```swift
<XCTAssert>(<condition>, <error_message>)
```
Where:
```
- XCAssert: Kind of assertion: eg. XCAssertTrue -> If the condition is true, it is ok.
- condition: The condition we want to check in a string format.
- error_message: The message that is shown if the condition is not fulfilled. This is important, so we get a quick understanding about why the test fails.
```

In the used test case, we will check if the result label text is *How are you?*:
```swift
func test_sendMessage_Hello() {
    ...
    // Then
    XCTAssertTrue(labelResult.text == "How are you?", "Invalid text result")
}
```

## Conclusion

UI Testing can be a very productive way to test your interface but making the tests understandable and maintainable can be hard if we don't hold to best practices.
There are many ways to improve the test cases, I showed you some. Happy coding!
