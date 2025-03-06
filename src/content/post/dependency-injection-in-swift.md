---
layout: ../../layouts/post.astro
title: Dependency Injection in Swift
description: A Comprehensive Guide
dateFormatted: Oct 22, 2023
---

![Dependency Injection In Swift](/assets/images/posts/dependency-injection.webp)

Dependency Injection (DI) may sound like an intimidating technical term, but as James Shore famously put it: 
> Dependency Injection is a 25-dollar term for a 5-cent concept.

This article aims to demystify this powerful design pattern and demonstrate how it can dramatically improve your Swift applications.

## What is Dependency Injection?

At its core, dependency injection is a design pattern where a class receives its dependencies from external sources rather than creating them internally. Think of it as "providing what a class needs from the outside" instead of having the class create or find what it needs on its own.

### The Problem DI Solves

Let's start with a common scenario in iOS development:

```swift
class UserProfileViewController: UIViewController {
    // Creating the dependency internally
    private let userService = UserService()
    
    func loadUserProfile() {
        let userId = getCurrentUserId()
        userService.fetchUserProfile(userId: userId) { [weak self] result in
            // Handle result
        }
    }
}
```

In this example, `UserProfileViewController` creates its own instance of `UserService`. This tight coupling introduces several issues:

1. **Testing becomes difficult** - You can't easily substitute a mock service
2. **Reusability is limited** - The class is tied to a specific implementation
3. **Flexibility is reduced** - Changing the service implementation affects all using classes

## Types of Dependency Injection in Swift

Let's explore the three primary methods of implementing dependency injection in Swift:

### 1. Initializer-based Dependency Injection

This is often considered the most straightforward and safest approach:

```swift
class UserProfileViewController: UIViewController {
    private let userService: UserServiceProtocol
    
    // Dependencies provided through initializer
    init(userService: UserServiceProtocol) {
        self.userService = userService
        super.init(nibName: nil, bundle: nil)
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    func loadUserProfile() {
        let userId = getCurrentUserId()
        userService.fetchUserProfile(userId: userId) { [weak self] result in
            // Handle result
        }
    }
}
```

By using a protocol `UserServiceProtocol`, we've made our class more flexible and testable. The protocol might look like:

```swift
protocol UserServiceProtocol {
    func fetchUserProfile(userId: String, completion: @escaping (Result<UserProfile, Error>) -> Void)
}
```

This approach ensures dependencies are provided at initialization time and can't be changed later, making the object's state more predictable.

### 2. Property-based Dependency Injection

This approach allows dependencies to be set after initialization:

```swift
class UserProfileViewController: UIViewController {
    // Dependency can be set after initialization
    var userService: UserServiceProtocol!
    
    func loadUserProfile() {
        guard userService != nil else {
            fatalError("UserService not set")
        }
        
        let userId = getCurrentUserId()
        userService.fetchUserProfile(userId: userId) { [weak self] result in
            // Handle result
        }
    }
}
```

While more flexible, this approach comes with risks - the dependency might not be set when needed, requiring additional validation.

### 3. Method-based Dependency Injection

With this approach, dependencies are provided directly to the methods that need them:

```swift
class UserProfileViewController: UIViewController {
    
    func loadUserProfile(using userService: UserServiceProtocol) {
        let userId = getCurrentUserId()
        userService.fetchUserProfile(userId: userId) { [weak self] result in
            // Handle result
        }
    }
}
```

This is useful when a dependency is only needed for specific operations.

## Real-World Example: Building a Weather App

Let's see how dependency injection shines in a real-world scenario:

```swift
// Define our protocol
protocol WeatherServiceProtocol {
    func fetchWeather(for location: String, completion: @escaping (Result<WeatherData, Error>) -> Void)
}

// Concrete implementation
class OpenWeatherMapService: WeatherServiceProtocol {
    func fetchWeather(for location: String, completion: @escaping (Result<WeatherData, Error>) -> Void) {
        // Actual implementation to fetch from OpenWeatherMap API
    }
}

// Test implementation
class MockWeatherService: WeatherServiceProtocol {
    var weatherToReturn: WeatherData?
    var errorToReturn: Error?
    
    func fetchWeather(for location: String, completion: @escaping (Result<WeatherData, Error>) -> Void) {
        if let error = errorToReturn {
            completion(.failure(error))
        } else if let weather = weatherToReturn {
            completion(.success(weather))
        }
    }
}

// Our view controller with DI
class WeatherViewController: UIViewController {
    private let weatherService: WeatherServiceProtocol
    private let location: String
    
    init(weatherService: WeatherServiceProtocol, location: String) {
        self.weatherService = weatherService
        self.location = location
        super.init(nibName: nil, bundle: nil)
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        loadWeatherData()
    }
    
    private func loadWeatherData() {
        weatherService.fetchWeather(for: location) { [weak self] result in
            // Update UI based on result
            switch result {
            case .success(let weatherData):
                self?.updateUI(with: weatherData)
            case .failure(let error):
                self?.showError(error)
            }
        }
    }
}
```

In this example, our `WeatherViewController` can work with any service that conforms to `WeatherServiceProtocol`. This provides several key benefits:

1. **Testing is easy** - We can inject a `MockWeatherService` for tests
2. **We can switch API providers** - Want to use a different weather API? Create a new implementation of the protocol
3. **The class is focused** - It doesn't need to know how weather data is fetched, only how to display it

## Advanced Patterns with Dependency Injection

### Using DI Containers

For larger applications, manually wiring up dependencies can become cumbersome. DI containers can help:

```swift
class AppContainer {
    // Shared singleton for simplicity
    static let shared = AppContainer()
    
    // Services
    lazy var networkService: NetworkServiceProtocol = {
        return URLSessionNetworkService()
    }()
    
    lazy var weatherService: WeatherServiceProtocol = {
        return OpenWeatherMapService(networkService: self.networkService)
    }()
    
    // Create view controllers with injected dependencies
    func makeWeatherViewController(for location: String) -> WeatherViewController {
        return WeatherViewController(weatherService: weatherService, location: location)
    }
}
```

Now creating a view controller is simple:

```swift
let weatherVC = AppContainer.shared.makeWeatherViewController(for: "London")
```

### Property Wrappers for DI (Swift 5.1+)

With property wrappers, we can make dependency injection even more elegant:

```swift
@propertyWrapper
struct Inject<T> {
    let wrappedValue: T
    
    init() {
        self.wrappedValue = Resolver.shared.resolve()
    }
}

// Usage
class WeatherViewController: UIViewController {
    @Inject private var weatherService: WeatherServiceProtocol
    
    // Rest of the class...
}
```

This approach requires a resolver class to handle the dependency resolution, but it makes the DI pattern very clean and concise in your class implementations.

## Best Practices

1. **Use protocols** - They provide the abstraction needed for effective DI
2. **Prefer initializer injection** - It creates immutable dependencies and makes object state more predictable
3. **Keep dependencies explicit** - Don't hide dependencies in a container unless needed for simplicity
4. **Be mindful of object lifecycles** - Ensure injected objects are retained properly
5. **Don't over-inject** - Only inject what a class truly needs from the outside

## Common Pitfalls to Avoid

1. **Creating circular dependencies** - When two classes depend on each other
2. **Injecting too many dependencies** - May indicate a class with too many responsibilities
3. **Not validating optional dependencies** - Particularly with property injection
4. **Relying too heavily on singletons** - Even with DI, singletons can limit testability

## Conclusion

Dependency injection is a powerful pattern that promotes loose coupling, testability, and maintainability in your Swift applications. While it may seem like an extra step at first, the benefits quickly become apparent as your application grows in complexity.

By making your dependencies explicit and external, you'll not only write more testable code but also gain flexibility to adapt to changing requirements. Start simple with initializer injection and evolve your DI approach as your application's needs grow.

Remember, dependency injection isn't just about writing testable code—it's about creating a flexible, maintainable architecture that can evolve with your application.