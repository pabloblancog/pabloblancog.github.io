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

- Create a basic SwiftUI layout
- Create custom elements: Texts, Textfields, switchs and buttons
- Create a basic field validation
- Display alert from a button

## 1. Layout

We start creating a new SwiftUI view class called LoginView:
```
struct LoginView: View {
    var body: some View {

    }
}
```

The layout to create consists of:


So, following the layout, we will add the elements:

```swift
struct LoginView: View {

    @State private var email = ""
    @State private var password = ""
    @State private var agreedToTerms = false
    
    var body: some View {

            // 1. Vertical stack
            VStack(alignment: .leading, spacing: 10) {
            
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
                        .frame(minWidth: 0, maxWidth: .infinity)
                }
                
                // 7. Bottom spacer
                Spacer()
            }
    }
}
```

**1. Vertical stack: `VStack(alignment: .leading, spacing: 10)`**

In SwiftUI there are many ways to distribute our content on the screen. One of them is `VStack`, that means a vertical stack of elements. Every element declared inside the container `VStack` will be included in a stack, following the declaration order.
In this case our stack will be composed by a text, two textfields, a toogle and a button (2-6).

`VStack(alignment: .leading, spacing: 10)` means that the elements in the stack will be aligned to the leading (left side), and each element will have a 10 points spacing to the next one.

**2. Text**: `Text("Introduce your credentials")` allows to insert a text on the screen, in this case our screen description.

**3. Email field**: `TextField("Email", text: $email)` This declaration allows include an input element so the user can add info to the app. In this case, this textfield will be used for capture the user's email. The text `Email` corresponds to the placeholder that the textfield will show.
We will talk later about the `text: $email` declaration.

**4. Password field**: `SecureField("Password", text: $password`. Similar to the previous textfield, this element allows introduce characters in a secure way, hiding the text. As the textfield, `Password` stands for the placeholder. 
We will talk later about the `text: $password` declaration.

**5. Toogle**: 
**6. Sign in button**
**7. Bottom spacer** (more info on another episode...)


## 3. Validation

## 4. Success, alert display

## Conclusion, where to go?
- Custom UI elements -> Custom log in
- Field validation? More complicated forms? -> Sign up form