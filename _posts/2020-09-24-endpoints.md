---
layout: post
title: Better REST API management in Swift
published: true
---

Spending time developing new features is great. Spending unnecessary time updating your REST API endpoints is not. But… how to avoid it?


A non-unified approach to managing endpoints can provoke many issues if API updates imply a lot of similar, repetitive code changes.

In this article, you will understand how to structure your REST API endpoint configuration in a better way, by using some swifty techniques.


Let’s review this piece of code. It creates and launches a request to get some user data from a REST API:

```swift
class Worker {

    func getUsers(city: String, completion: @escaping (URLResponse?, Error?) -> Void) {

        let session = URLSession.shared
        let url = URL(string: "https://pabloblan.co/api/v1/users/" + city)!
        var urlRequest = URLRequest(url: url)
        
        urlRequest.httpMethod = "GET"
        
        urlRequest.setValue("application/json", forHTTPHeaderField: "Content-Type")
        urlRequest.setValue("application/json", forHTTPHeaderField: "Accept")
        urlRequest.setValue("1a2s3d4f5g6h7j", forHTTPHeaderField: "Authorization")
        urlRequest.setValue("123456789", forHTTPHeaderField: "API-KEY")
        urlRequest.setValue("PROJECT1", forHTTPHeaderField: "PROJECT")
        urlRequest.setValue("IOS", forHTTPHeaderField: "CHANNEL")
        
        let task = session.dataTask(with: urlRequest) { data, response, error in
            completion(response, error)
        }

        task.resume()
    }
}
```

Ok, the code works. But this implementation has some underlying issues:
- *What if you need to add a new endpoint?* Duplicate this function adding a lot of new, similar code.
- *What if you have many of these endpoints and need to upgrade your API version?* Find and update every declared URL.
- *What if you need to include some new authentication fields?* Update header fields on every endpoint declaration.

These problems can be resolved by using a good **REST API endpoint configuration**.

```swift
protocol Endpoint {
    var httpMethod: HTTPMethod { get }
    var baseURLString: String { get }
    var path: String { get }
    var headers: [String: Any]? { get }
    var body: [String: Any]? { get }
}

extension Endpoint {
    // a default extension that creates the full URL
    var url: String {
        return baseURLString + path
    }
}
```

## 1. Endpoint definition
This is all about endpoints, so it makes sense to create an `Endpoint` protocol that defines all its content.
> This `Endpoint` declaration can be extended to add further details and needs, p.eg. security configuration.

## 2. Endpoint cases
The app needs to get users from a concrete city, so I need to define only one case:

```swift
enum EndpointCases {
    case getUsers(city: String)
}
```

## 3. Endpoint data filling
To collect this endpoint details, you will implement `Endpoint` protocol in our `EndpointCases` struct.
Every endpoint required field is filled including a `switch` statement to get the right data for the needed case. By now, we have a single case: `getUsers`.

```swift
enum EndpointCases: Endpoint {
    
    case getUsers(city: String)
    
    var httpMethod: String {
        switch self {
        case .getUsers:
            return "GET"
        }
    }
    
    var baseURLString: String {
        switch self {
        case .getUsers:
            return "https://pabloblan.co/api/"
        }
    }
    
    var path: String {
        switch self {
        case .getUsers(let city):
            return "v1/users/\(city)/"
        }
    }
    
    var headers: [String: Any]? {
        switch self {
        case .getUsers:
            return ["Content-Type": "application/json",
                    "Accept": "application/json",
                    "Authorization": "1a2s3d4f5g6h7j",
                    "API-KEY": "123456789",
                    "PROJECT": "PROJECT1",
                    "CHANNEL": "IOS"]
        }
    }
    
    var body: [String : Any]? {
        switch self {
        case .getUsers:
            return [:]
        }
    }
}
```

## 4. Endpoint usage
It’s time to use our endpoint. We need to update our `Worker` class to connect properly with this new `Endpoint` implementation:

```swift
class Worker {
	
    func getUsers(endpoint: Endpoint, completion: @escaping (URLResponse?, Error?) -> Void) {
                
	let session = URLSession.shared

	// URL
	let url = URL(string: endpoint.url)!
	var urlRequest = URLRequest(url: url)

	// HTTP Method
	urlRequest.httpMethod = endpoint.httpMethod

	// HTTP Headers
	endpoint.headers?.forEach({ header in
	    urlRequest.setValue(header.value as? String, forHTTPHeaderField: header.key)
	})
        
        let task = session.dataTask(with: urlRequest) { data, response, error in
            completion(response, error)
        }

        task.resume()
    }
}
```

Some things to point out:
- **Data is not hardcoded anymore**: We get the URL directly from the endpoint, and also the httpMethod and the header fields.
- **Better header fields configuration**: As we joined all the headers in a variable, we can now loop over them and set as many headers as declared.

Now we just need to call our modified function where needed, like this:

```swift
let usersEndpoint = EndpointCases.getUsers(city: "Madrid")
Worker().getUsers(endpoint: usersEndpoint) { response, error in
    print(response)
}
```

## 5. Add a new endpoint
Everything is working nicely. Now it’s time to add new features to our app. In this case, we want to add users to the database. Let’s add the new case to the endpoints enumeration.

```swift
enum EndpointCases {
    case getUsers(city: String)
    case addUser(name: String, surname: String, city: String)
}
```

The compiler warns us to fill the required data in our `EndpointCases` struct. Our new endpoint is a `POST` endpoint, so we also include some `body` data.

```swift
enum EndpointCases: Endpoint {
    
    case getUsers(city: String)
    case addUser(name: String, surname: String, city: String)
    
    var httpMethod: String {
        switch self {
        case .getUsers:
            return "GET"
        case .addUser:
            return "POST"
        }
    }
    
    var baseURLString: String {
        switch self {
        case .getUsers:
            return "https://pabloblan.co/api/"
        case .addUser:
            return "https://pabloblan.co/api/"
        }
    }
    
    var path: String {
        switch self {
        case .getUsers(let city):
            return "v1/users/\(city)"
        case .addUser:
            return "v1/users/add"
        }
    }
    
    var headers: [String: Any]? {
        switch self {
        case .getUsers:
            return ["Content-Type": "application/json",
                    "Accept": "application/json",
                    "Authorization": "1a2s3d4f5g6h7j",
                    "API-KEY": "123456789",
                    "PROJECT": "PROJECT1",
                    "CHANNEL": "IOS"]
        case .addUser:
            return ["Content-Type": "application/json",
                    "Accept": "application/json",
                    "Authorization": "1a2s3d4f5g6h7j",
                    "API-KEY": "123456789",
                    "PROJECT": "PROJECT1",
                    "CHANNEL": "IOS"]
        }
    }
    
    var body: [String : Any]? {
        switch self {
        case .getUsers:
            return [:]
        case .addUser(let name, let surname, let city):
            return ["name": name,
                    "surname": surname,
                    "city": city]
        }
    }
}
```

And the call to Worker’s `addUser` function will be:

```swift
Worker().addUser(name: "Pablo", surname: "Blanco", city: "Barcelona") { response, error in
    print(response)
}
```
## One more thing…

We can make some improvements to our `Worker` class:

- Reusing the generic request function to be used by every endpoint, making the function calls clearer and more understandable.
- `func request()` is now private so the only point to access the `Worker` right now is the designated methods `getUsers()` and `addUser()`.

```swift
class Worker {
	
	func getUsers(from city: String, completion: @escaping (URLResponse?, Error?) -> Void) {
		let usersEndpoint = EndpointCases.getUsers(city: city)
		request(endpoint: usersEndpoint) { response, error in
			completion(response, error)
		}
	}

	func addUser(name: String, surname: String, city: String, completion: @escaping (URLResponse?, Error?) -> Void) {
		let addUserEndpoint = EndpointCases.addUser(name: name, surname: surname, city: city)
		request(endpoint: addUserEndpoint) { response, error in
			completion(response, error)
		}
	}

	private func request(endpoint: Endpoint, completion: @escaping (URLResponse?, Error?) -> Void) {

		let session = URLSession.shared

		// URL
		let url = URL(string: endpoint.url)!
		var urlRequest = URLRequest(url: url)

		// HTTP Method
		urlRequest.httpMethod = endpoint.httpMethod

		// Header fields
		endpoint.headers?.forEach({ header in
			urlRequest.setValue(header.value as? String, forHTTPHeaderField: header.key)
		})

		let task = session.dataTask(with: urlRequest) { data, response, error in
			completion(response, error)
		}

		task.resume()
	}
}
```

Now the calls to the `Worker` class will be:

```swift
Worker().getUsers(from: "Madrid") { response, error in
    print(response)
}

Worker().addUser(name: "Pablo", surname: "Blanco", city: "Barcelona") { response, error in
    print(response)
}
```

## Conclusion
Swift provides some good techniques that make API configurations a little bit cleaner and understandable.
Enums and protocols are powerful when mixed, and we can take advantage of both to create a structure that keeps our endpoints organized and easy to maintain and configure.

## Where to go next?
Some related functionalities to this API endpoint configuration approach may be covered in the next articles:
- *Decoding*: How to properly decode given data from JSON files in a generic way.
- *Environments*: How to use this approach to support many different API environments and their different API configurations.
