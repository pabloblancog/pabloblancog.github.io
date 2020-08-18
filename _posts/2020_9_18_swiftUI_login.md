---
published: false
---


# Basic login with SwiftUI for beginners

`SwiftUI 1.0` `Swift 5.2`

## Goal

Build a login screen using `SwiftUI` with the next requirements:
- Proposed layout
- Field validation: User & password textfields need to be filled and the toogle for Accept Terms and Conditions enabled
- If enabled, the button will display a message when pressed.

## Topics related

- Creating a SwiftUI layout
- Declaring UI elements: `Text`, `Textfield`, `Toogle` and `Button`
- Setting and validating input data from these UI elements
- Wrapping the whole view for navigation

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

    @State private var email: String = ""
    @State private var password: String = ""
    @State private var agreedToTerms: Bool = false
    
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
                    print("Logged in!")
                }){
                    Text("Sign in")
                }
                
                // 7. Bottom spacer
                Spacer()
            }
    }
}
```

> Don't worry about the `@State private var` variables declared on the top, we will go through them later.

**//1. Vertical stack: `VStack(alignment: .leading, spacing: 10)`**

In SwiftUI there are many ways to distribute our content on the screen: Horizontal, Vertical, in layers...

One of them is **VStack**, that means a vertical stack of elements.
Every element declared inside the container `VStack` will be included in a stack, following the declaration order.
In this case, our stack will be integrated by a text, two textfields, a toogle and a button (//2 to //6).

**//2. Text**: `Text("Introduce your credentials")` insert a text on the screen, in this case our screen description.

**//3. Email field**: `TextField("Email", text: $email)` A textfield allows users to add input data to the app. In this case, it will be used for capturing the user's email.
The text `Email` corresponds to the textfield placeholder.

We will talk later about the `text: $email` declaration.

**//4. Password field**: `SecureField("Password", text: $password)`.

It is a type of textfield that allows data input in a secure way, hiding the text with bullets. The string `Password` is its placeholder.

We will talk later about the `text: $password` declaration.

**//5. Toogle to agree T&C**: `Toogle(isOn: ...) {}`
Toogle allows users to choose a state associated to a property. In our case, we decide if the user accepts the Terms and conditions or he/she doesn't.

The toogle is visually composed horizontally by two elements:
- Left: Any view. In this case is the text `Text("Agree to terms and conditions")`that we include inside the brackets.
```
Toggle(isOn: $agreedToTerms) {
    Text("Agree to terms and conditions")
}
```
- Right: The toogle control. It is enabled/disabled depending on the variable included after the word `isOn` on `Toggle(isOn: ...)`

We will talk later about the `$agreedToTerms` declaration.

**//6. Sign in button**
Defined as: 
```
Button(action: {
    print("Logged in!")
}){ 
   Text("Sign in")
       .frame(minWidth: 0, maxWidth: .infinity)
}
```
The button has two parts:
- Action: Indicates the action taken when the button is pressed.
```
action: {
   print("Logged in!")
}
```

- Body of the button: The view that the button will display. In this case, we are displaying the text `Sign in`.
```
{
    Text("Sign in")
}
```

**//7. Bottom spacer** (more info on another episode...)

Once we have filled the vertical stack, how does the compiler know the stack vertical position? Should it be next to the top? Center? Bottom?

The element `Spacer()` tells the compiler that a space must be created on the defined position, so the other elements must be readapted to fit the layout to the screen.

In this case we want the elements to be aligned to the top, so we are saying with `Spacer()` that the bottom includes a free space, so the elements will go up on the screen.

> If no `Spacer()` is provided, so the position of the elements cannot be properly calculated, the compiler set by default the stack vertically centered.

> Play with `Spacer()` by placing it between elements or in the first place on the stack. Run the app to check how the elements are reorganized on the screen.


## 2. Input data saving `@State variables`

If we remember, in the textfields and the toogle previously declared, we had three variables: `$email`, `$password`, `$agreedToTerms`.

```
TextField("Email", text: $email)
SecureField("Password", text: $password)
Toggle(isOn: $agreedToTerms)
```
These three variables are declared in the class, before the `body View` and save the input state of the elements:

```
@State private var email: String = ""
@State private var password: String = ""
@State private var agreedToTerms: Bool = false
```
    
`$email` is a string that contains the input data for the email textfield
`$password` is a string that contains the input data for the password textfield
`$agreedToTerms` is a boolean value contains with the input data for the toogle

When the user interacts with the elements, these variables automatically get their content updated.

In the other hand, if these variables are modified, the UI element will be updated too. For example, the toogle needs a initial state, so we include the `false` value to `agreedToTerms`.

## 3. Input data validation

In this example, the `Sign in` button will be enabled when:
- The User textfield and the Password textfield are not empty (`$email` and `$password`)
- The toogle is enabled (`$agreedToTerms`)

These variables are referred to the ones stored in the class (explained in the previous point)
A way to validate whether they are valid could be:
```
let isEnabled = !email.isEmpty && !password.isEmpty && agreedToTerms == true
```
or if they are invalid:
```
let isDisabled = email.isEmpty || password.isEmpty || agreedToTerms == false
```

## 4. Button update - `View modifier`

Although the input data is validated, the button needs to be updated in order to enable/disable the login.
```
// 6. Sign in button
Button(action: {
    print("Logged in!")
}){
    Text("Sign in")
}
```

SwiftUI allows the developer to modify an element appearance or logic by using `View modifiers`. These modifiers are declared after the element declaration.

```
// 6. Sign in button
Button(action: {
    print("Logged in!")
}){
    Text("Sign in")
}
.disabled(email.isEmpty || password.isEmpty || !agreedToTerms)

```

In this case, the line `.disabled(email.isEmpty || password.isEmpty || !agreedToTerms)` indicates that the button will be disabled when the condition inside is fulfilled.

When the button is enabled and is pressed, the message `Logged in` in the console will be displayed.

## 5. Title and layout fixes

Once we know about view modifiers, we can resolve two problems related to the layout:

### Elements are too close to the screen bounds
Let's add the modifier `.padding(16)` after the `VStack` container for adding a 16pt extra padding the whole stack view.

### No title provided

To fix this, we will add a `navigation view`. That means that if we add new independent views and we connect them, we can navigate through them.

To achieve this, we will wrap the entire `VStack` container into a `NavigationView` container:

var body: some View {

    // Navigation container view
    NavigationView {

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
                print("Logged in!")
            }){
                Text("Sign in")
            }

            // 7. Bottom spacer
            Spacer()
        }
        .padding(16)
    }
}

Now we have navigation, we can add a navigation title to the screen, adding the view modifier `navigationBarTitle("Log in")` for including the title.


## Conclusion, where to go?

SwiftUI uses a declarative way of creating layouts. It is very comprehensive and easy to use.

- Custom UI elements -> Custom log in
- Field validation? More complicated forms? -> Sign up form