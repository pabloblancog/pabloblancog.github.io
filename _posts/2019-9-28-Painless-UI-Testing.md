---
layout: post
title: Painless UI Testing
---

UI Testing is a great thing to keep apps maintainable and reliable during its whole lifecycle, but sometimes could become a big pain.
Here we will transform a bad test case into a better one, applying some code refactors.

> Note: it's assumed that the reader knows some basic usage of UI testing.

We have obtained the next test using the Recording feature from Xcode:
```swift
func test1() {
    // 1. The user taps in a textfield
    app.textfields["Send something..."].tap()
    
    // 2. The user types the text "Hello"
    app.textfields["Send something..."].type("Hello")
    
    // 3. The user taps send button
    app.buttons["Send"].tap()
    
    // 4. If the text in the label is "How are you?", the text will pass.
    XCTAssert(app.labels["result"].text == "How are you?")
}
```

This code looks understandable and easy to follow, but …

> Test plans become difficult to maintain when they grow

UI Elements can change during development, making tests stop working and needed for refactoring.
Also, test cases like the one above could provoque code duplication when some new cases using the same UI elements were required.

Some improvements can be made in order to avoid code refactors after every test case creation. Let's see some:

### 1. Test case naming
Test cases names must be intuitive and understandable. A test case name should follow a syntax like this:
```swift
func test_<what_to_test>_<conditions>() {
    ...
}
```
Where:
```swift
test_: Indicates that this is an Xcode test case method. 
Note: In case you need a helper function, you must not include this prefix.
<what_to_test>: Indicates the purpose of your test.
<conditions>: Indicates the conditions set for the test case.
```

So, in the previous test case will become this:
```swift
func test_sendMessage_hello() {
    app.textfields["Send something..."].tap()
    app.textfields["Send something..."].type("Hello")
    app.buttons["Send"].tap()
    XCTAssert(app.labels["result"].text == "How are you?")
}
```
  
### 2. Test case structure
Test cases should follow a fixed structure, so the test case can be easily followed step by step. The next structure, based on Gherkin syntax, is:
```swift
Given the scenario X, when I make the action X, then I got the result X
```
A title on each step should be included to clarify the structure. In the test case, also an initial configuration is made fo textField and the message to send has been included.
```swift
func test_sendMessage_hello() {
    // Given
    textField.text = ""
    let messageToSend = "Hello"
    // When
    app.textfields["Send something..."].tap()
    app.textfields["Send something..."].type("Hello")
    // Then
    app.buttons["Send"].tap()
    XCTAssert(app.labels["result"].text == "How are you?")
}
```

### 3. UI elements: Access and interaction
When you record a tap on the textfield directly from the simulator or the device, you will end up with something like this:
```swift
app.textfields["Send something..."].tap()
```
Xcode detected you tapped on a textfield that has the placeholder *Send something*. If the placeholder is changed in a later code update, the test will stop working, and it will have to be refactored.
To avoid this scenario, let's use **accessibility identifiers**.
Accessibility identifiers helps disabled people using apps. Applied to UI testing, they allow to identify elements.
In the viewController class related to the test, some identifiers can be defined:
```swift
class ViewController: UIViewController {
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

UI Testing includes some useful methods that can be used to init and reset our test cases shared data.
```swift
setUp(): Sets up a test case before it is launched.
tearDown(): Clears data after the test case finishes.
```

On XCUITests, an element on the UI can be accessed by its identifier from an array that includes every UI element of a kind on the current visible screen of the app: app.buttons, app.textfields, app.labels, etc.
For the test case, some variables for the UI elements will be initialized on the setUp method:
```swift
class UITests: XCTestCase {
    var app: XCUIApplication!
    var inputTextField: XCUIElement!
    var sendButton: XCUIElement!
    var resultLabel: XCUIElement!
    override func setUp() {
        app = XCUIApplication()
        sendButton = app.buttons["send_button"]
        inputTextField = app.textfields["input_text_field"]
        resultLabel = app.labels["result_label"]
   }
}
```

Based on these new variables, a little refactor on the test case can be made. The elements keep identified no matter what their value are (button title, label text, textfield placeholder, etc.).
```swift
func test_sendMessage_Hello() {
    // Given
    textField.text = ""
    let messageToSend = "Hello"
    // When
    inputTextField.tap()
    inputTextField.type("Hello")
    sendButton.tap()
    // Then
    XCTAssert(resultLabel)
    XCTAssert(resultLabel.text == "How are you?")
}
```

### 4. Reusing methods
Let's imagine we have some different responses depending on what text we send. New test cases will become very repetitive, and they will duplicate a lot of code.
For reusing some code between test cases, let's create some helper methods with the most repeated actions in the test cases.
```swift
func typeTextOnInputBar(_ text: String) {
    inputTextView.tap()
    inputTextView.typeText(text)
}
func tapSendButton() {
    sendButton.tap()
}
```
Joining this two methods, we get:
```swift
func sendMessage(_ message: String) {
    typeTextOnInputBar(message)
    tapSendButton()
}
```
And the test case will be like this:
```swift
func test_sendMessage_Hello() {
    // Given
    textField.text = ""
    let messageToSend = "Hello"
    // When
    sendMessage("Hello")
    // Then
    XCTAssert(labelResult)
    XCTAssert(labelResult.text == "How are you?")
}
```

### 5. Better assertions
It's time to check if everything in the test case went as expected. For this purpose, we will use XCTAssert.
> XCTAssert statements indicate whether a condition is OK or it is not.
An XCTAssert statement is composed like this:
```swift
<XCTAssert>(<condition>, <error_message>)
```
Where:
```
XCAssert: Kind of assertion: eg. XCAssertTrue -> If the condition is true, it is ok.
condition: The condition we want to check in a string format.
error_message: The message that is shown if the condition is not fulfilled. This is important, so we get a quick understanding about why the test fails.
```

Let's update our test case:
```swift
func test_sendMessage_Hello() {
    // Given
    textField.text = ""
    let messageToSend = "Hello"
    // When
    sendMessage("Hello")
    // Then
    XCTAssertTrue(labelResult.text == "How are you?", "Invalid text result")
}
```
Conclusion
UI testing is an amazing way to test an app, and following the steps explained will make the test cases will be more understandable and maintainable.
Happy coding!
