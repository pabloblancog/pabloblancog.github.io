---
published: false
---


# Basic login with SwiftUI for beginners

`SwiftUI 1.0` `Swift 5.2`

## Goal

Create a simple login:
- Requires user/password to be filled
- Requires a switch to be turned on
- An alert will be displayed when succeed.

## Things you will learn

- Creating a basic SwiftUI layout
- Using visual elements like `Text`, `Textfield`, `Toogle` and `Button`
- Validating non-empty fields
- Displaying an alert

## 1. Layout

The layout to create consists of:

> Layout

We start creating a new SwiftUI view class called `LoginView`:
```
struct LoginView: View {
    var body: some View {

    }
}
```

So, following the layout, we will add the elements:

```swift
struct LoginView: View {

    @State private var email = ""
    @State private var password = ""
    @State private var agreedToTerms = false
    
    var body: some View {

            // 1. Vertical stack
            VStack() {
            
                // 2. Text
                Text("Introduce your credentials")

                // 3. Email field
                TextField("Email", text: $email)

                // 4. Password field
                SecureField("Password", text: $password)

                // 5. Toogle to agree T&C
                Toggle(isOn: $agreedToTerms) {
                    Text("Agree to terms and conditions")
                }

                // 6. Sign in button
                Button(action: {
                
                }){
                    Text("Sign in")
                }
                
                // 7. Bottom spacer
                Spacer()
            }
    }
}
```

**//1. Vertical stack: `VStack(alignment: .leading, spacing: 10)`**

In SwiftUI there are many ways to distribute our content on the screen. One of them is `VStack`, that means a vertical stack of elements. Every element declared inside the container `VStack` will be included in a stack, following the declaration order.
In this case our stack will be composed by a text, two textfields, a toogle and a button (2-6).

**//2. Text**: `Text("Introduce your credentials")` allows to insert a text on the screen, in this case our screen description.

**//3. Email field**: `TextField("Email", text: $email)` This declaration allows include an input element so the user can add info to the app. In this case, this textfield will be used for capture the user's email. The text `Email` corresponds to the placeholder that the textfield will show.
We will talk later about the `text: $email` declaration.

**//4. Password field**: `SecureField("Password", text: $password)`.
It is a type of textfield that allows data input in a secure way, hiding the text with bullets. The string `Password` is its placeholder.
We will talk later about the `text: $password` declaration.

**//5. Toogle to agree T&C**: `Toogle(isOn: ...) {}`
Toogle allows us to choose a state associated to a property. In our case, we decide whether we accept the Terms and conditions or we don't.

The toogle is composed by two horizontal elements: 
- The element on the left: It can be any view you can imagine. In this case is the text `Text("Agree to terms and conditions")`that we include inside the brackets.
```
Toggle(isOn: $agreedToTerms) {
    Text("Agree to terms and conditions")
}
```
- The element on the right: the toogle. It is enabled/disabled depending on the variable included after the word `isOn` on `Toggle(isOn: ...)`

We will talk later about the `$agreedToTerms` declaration.

**//6. Sign in button**
Defined as: 
```
Button(action: {
}){ 
   Text("Sign in")
       .frame(minWidth: 0, maxWidth: .infinity)
}
```
The button has two parts:
Action: Indicates the action taken when the button is pressed (Not filled yet)
```
action: {
   ...
}
```

Body of the button: The view that the button will display. In this case, we are displaying the text `Sign in`.
```
{
    Text("Sign in")
}
```

**//7. Bottom spacer** (more info on another episode...)

Once we have filled the vertical stack, how the compiler knows where is the position of it in the vertical axis. Should it be next to the top? Center? Bottom?
The element `Spacer()` tells the compiler that a space must be created on the defined position, so the other elements must be readapted to fit the layout to the screen.
In this case we want the elements to be aligned to the top, so we are saying with `Spacer()` that the bottom includes a free space, so the elements need to go up on the screen.

> If no Spacer() is provided, so the position of the elements cannot be properly calculated, the compiler set by default the stack on the vertical center.

> Play with `Spacer()` moving it between elements or placing it in the first place on the stack.


## 3. Validation


## 4. Success, alert display

## Conclusion, where to go?
- Custom UI elements -> Custom log in
- Field validation? More complicated forms? -> Sign up form