---
layout: post
title: Handling environments in Swift
published: true
---

In real-world apps, developers deal with many different backend environments. Every environment has its properties and configuration so it is important to have a proper structure to manage them.

In this article, I explain how to create a great structure for environment configuration in Swift, that can easily evolve but rely stable on.

## 1. Endpoint structure
We start by creating an `Endpoint` struct declared like this:

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

## 2. App endpoints
Our app has two different features:
- *getUsers*: Get the users from a city
- *addUser*: Add a new user

So, let's declare an enumeration with two different cases:

```swift
enum EndpointCases {
    case getUsers(city: String)
    case addUser(name: String, surname: String, city: String)
}
```

## 3. Environment configuration
To have a clear understanding of the environments that are managed, we create an environment configuration manager, that holds the environment used and the possibilities to select:

```swift
struct EnvironmentConfiguration {
    
    enum Environment {
        case production
        case development
    }
    
    var environment: Environment
}
```

> In this case, we have defined two different environments: **production** and **development**. Any other environments can be added as needed.

## 4. Configuring environment properties
It's time to configure every development defined on `EndpointCases`.
We create a `struct` per environment that provides the needed endpoint properties, as defined on the `Endpoint` protocol.
- In this case, we create the `ProductionEndpoint` struct, that implements the `Endpoint` protocol for the *production* environment.
- Inside this structure, `var endpointCase: EndpointCases` indicates the endpoint case needed.
- Within every field, we declare a `Switch` to distinguish every endpoint case.

```swift
struct ProductionEndpoint: Endpoint {
    
    var endpointCase: EndpointCases

    var httpMethod: String {
        switch endpointCase {
        case .getUsers:
            return "GET"
        case .addUser:
            return "POST"
        }
    }
    
    var baseURLString: String {
        switch endpointCase {
        case .getUsers:
            return "https://pabloblan.co/api/"
        case .addUser:
            return "https://pabloblan.co/api/"
        }
    }
    
    var path: String {
        switch endpointCase {
        case .getUsers(let city):
            return "v1/users/\(city)"
        case .addUser:
            return "v1/users/create"
        }
    }
    
    var headers: [String: Any]? {
        switch endpointCase {
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
        switch endpointCase {
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

## 5. Retrieving endpoint properties
The *production* environment properties are now ready to use, so let's create a function in our `EnvironmentConfiguration` to get the `Endpoint` properties.

```swift
struct EnvironmentConfiguration {
    
    enum Environment {
        case production
        case development
    }
    
    var environment: Environment
    
	func getEndpoint(endpointCase: EndpointCases) -> Endpoint {
		switch environment {
		case .production:
			return ProductionEndpoint(endpointCase: endpointCase)
		case .development:
			// Development struct to be filled
		}
	}
}
```

## 6. Select the environment for the worker
We now need the `Worker` class to implement the request call to the API.
- The `Worker` class needs to know the environment selected, so we include the environment configuration into it.
- The data request logic is developed on the `func request()`.
- To perform the requests needed for every case, we will access the `func request()` method from the different cases methods: `getUsers()` and `addUser()`.

```swift
class Worker {
    
    var configuration: EnvironmentConfiguration
    
    init(configuration: EnvironmentConfiguration) {
        self.configuration = configuration
    }

    func getUsers(from city: String, completion: @escaping (URLResponse?, Error?) -> Void) {
        // Request
    }

    func addUser(name: String, surname: String, city: String, completion: @escaping (URLResponse?, Error?) -> Void) {
        // Request
    }
    
    private func request(endpoint: Endpoint, completion: @escaping (URLResponse?, Error?) -> Void) {
                
	// Endpoint URL
        let session = URLSession.shared
        let url = URL(string: endpoint.url)!
        var urlRequest = URLRequest(url: url)
        
	// HTTP method
        urlRequest.httpMethod = endpoint.httpMethod
		
	// HTTP headers
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

## 7. Get the endpoint for the specified environment
Inside the `Worker` functions created for our app cases, we get the `Endpoint` needed by using our environment configuration.

```swift
func getUsers(from city: String, completion: @escaping (URLResponse?, Error?) -> Void) {
    let usersEndpoint: Endpoint = configuration.getEndpoint(endpointCase: .getUsers(city: city))
    request(endpoint: usersEndpoint) { response, error in
        completion(response, error)
    }
}

func addUser(name: String, surname: String, city: String, completion: @escaping (URLResponse?, Error?) -> Void) {
    let addUserEndpoint: Endpoint = configuration.getEndpoint(endpointCase: .addUser(name: name, surname: surname, city: city))
    request(endpoint: addUserEndpoint) { response, error in
        completion(response, error)
    }
}
```
Finally, depending on the environment you want to use, we can create and use the `Worker` class, like this:
```swift
let productionConfig = EnvironmentConfiguration(environment: .production)
let productionWorker = Worker(configuration: productionConfig)
productionWorker.getUsers(from: "Madrid") { response, error in }
productionWorker.addUser(name: "Pablo", surname: "Blanco", city: "Barcelona") { response, error in }

let developmentConfig = EnvironmentConfiguration(environment: .development)
let developmentWorker = Worker(configuration: developmentConfig)
developmentWorker.getUsers(from: "Madrid") { response, error in }
developmentWorker.addUser(name: "Pablo", surname: "Blanco", city: "Barcelona") { response, error in }
```

## Conclusion
Once again, enumerations and protocol-oriented programming allow Swift developers to create clearly defined structure to handle different scenarios, or environments, like in this case.
Every environment has its own properties and configuration, so using approaches like the one explained above provides us the needed abstraction to handle every situation as required.
## Where to go next
In this article, I covered how we can structure the way of getting endpoint properties when using different environments. But how do you set the environment to use depending on the app scheme? This will be covered in the next article.
