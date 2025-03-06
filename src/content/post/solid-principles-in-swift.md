---
layout: ../../layouts/post.astro
title: SOLID Principles in Swift
description: Building Robust iOS Applications
dateFormatted: Oct 15, 2023
---

![SOLID Principles in Swift](/assets/images/posts/solid-principles.webp)

As iOS developers, we often focus on what our code does rather than how it's structured. However, as your applications grow, well-architected code becomes crucial for maintainability, testability, and extensibility. The SOLID principles, introduced by Robert C. Martin, provide a framework for creating clean, maintainable Swift code that stands the test of time.

In this comprehensive guide, we'll explore each SOLID principle with practical Swift examples relevant to iOS development, understand their benefits, and learn implementation strategies for real-world applications.

## S: Single Responsibility Principle (SRP)

**The Single Responsibility Principle states that a class should have only one reason to change or should be responsible for only one aspect of your application's functionality.**

### Violating SRP

Let's examine a common pattern in iOS applications where we might violate SRP:

```swift
class PersonDataHandler {
    let person: Person
    
    init(person: Person) {
        self.person = person
    }
    
    func isValid() -> Bool {
        return !person.name.isEmpty && person.age >= 18
    }
    
    func save() {
        // Save to UserDefaults
        let defaults = UserDefaults.standard
        defaults.set(person.name, forKey: "personName")
        defaults.set(person.age, forKey: "personAge")
        defaults.synchronize()
        
        // Update UI
        NotificationCenter.default.post(name: Notification.Name("PersonSaved"), object: nil)
        
        // Analytics tracking
        AnalyticsManager.shared.trackEvent("person_saved", properties: ["name": person.name])
    }
    
    func formatForDisplay() -> String {
        return "Name: \(person.name), Age: \(person.age)"
    }
}
```

This `PersonDataHandler` class violates SRP because it's handling multiple responsibilities:
- Data validation 
- Data persistence
- UI updates
- Analytics tracking
- Formatting for display

### Applying SRP

Let's refactor this code to follow SRP:

```swift
// Data model
struct Person {
    let name: String
    let age: Int
}

// Validation responsibility
struct PersonValidator {
    func isValid(person: Person) -> Bool {
        return !person.name.isEmpty && person.age >= 18
    }
}

// Persistence responsibility
struct PersonStorage {
    func save(person: Person) {
        let defaults = UserDefaults.standard
        defaults.set(person.name, forKey: "personName")
        defaults.set(person.age, forKey: "personAge")
        defaults.synchronize()
    }
    
    func fetch() -> Person? {
        let defaults = UserDefaults.standard
        guard let name = defaults.string(forKey: "personName") else { return nil }
        let age = defaults.integer(forKey: "personAge")
        return Person(name: name, age: age)
    }
}

// UI notification responsibility
struct PersonNotifier {
    func notifySaved() {
        NotificationCenter.default.post(name: Notification.Name("PersonSaved"), object: nil)
    }
}

// Analytics responsibility
struct PersonAnalytics {
    func trackSave(person: Person) {
        AnalyticsManager.shared.trackEvent("person_saved", properties: ["name": person.name])
    }
}

// Formatting responsibility
struct PersonFormatter {
    func formatForDisplay(person: Person) -> String {
        return "Name: \(person.name), Age: \(person.age)"
    }
}
```

Now we have separate structs, each with a single responsibility. This approach offers several benefits:

- **Higher cohesion**: Each component focuses on one specific job
- **Better testability**: You can test each responsibility in isolation
- **Easier maintenance**: Changes to one aspect don't impact others
- **Improved reusability**: Components can be used independently in different contexts

### Few Tips for SRP

- In iOS development, consider separating UI-related code from business logic using patterns like MVVM or Coordinator
- Use extensions to organize code within a class by functionality
- Create dedicated manager classes for system services (like notifications, analytics, networking)
- Consider using value types (structs) for simple responsibilities instead of classes

## O: Open-Closed Principle (OCP)

**The Open-Closed Principle states that software entities should be open for extension but closed for modification.**

In simpler terms, you should be able to extend a class's behavior without modifying its source code.

### Violating OCP

Let's consider a payment processing example in an e-commerce iOS app:

```swift
class PaymentProcessor {
    func processPayment(amount: Double, method: String) {
        switch method {
        case "creditCard":
            print("Processing credit card payment of $\(amount)")
            // Credit card processing logic
            
        case "applePay":
            print("Processing Apple Pay payment of $\(amount)")
            // Apple Pay processing logic
            
        default:
            fatalError("Unsupported payment method")
        }
    }
}
```

This violates OCP because whenever we want to add a new payment method (like PayPal or crypto), we have to modify the existing `PaymentProcessor` class, which has been tested and is working correctly in production.

### Applying OCP

Let's refactor this using protocols and extensions:

```swift
// Abstract protocol that defines payment behavior
protocol PaymentMethod {
    func processPayment(amount: Double)
}

// Extension for existing payment methods
class CreditCardPayment: PaymentMethod {
    func processPayment(amount: Double) {
        print("Processing credit card payment of $\(amount)")
        // Credit card processing logic
    }
}

class ApplePayPayment: PaymentMethod {
    func processPayment(amount: Double) {
        print("Processing Apple Pay payment of $\(amount)")
        // Apple Pay processing logic
    }
}

// Updated payment processor that depends on abstraction
class PaymentProcessor {
    func processPayment(amount: Double, method: PaymentMethod) {
        method.processPayment(amount: amount)
    }
}

// Later, we can add new payment methods without modifying existing code
class PayPalPayment: PaymentMethod {
    func processPayment(amount: Double) {
        print("Processing PayPal payment of $\(amount)")
        // PayPal processing logic
    }
}
```

Now we can add new payment methods without modifying the original `PaymentProcessor` class. The class is closed for modification but open for extension.

### Few Tips for OCP

- Use Swift protocols and protocol extensions to define behavior that can be extended
- Leverage UIKit's delegation pattern, which allows behavior extension without modification
- Consider using the Strategy pattern for swappable algorithms and behaviors
- SwiftUI's view modifiers follow OCP by allowing you to extend view behavior without modifying the original views

## L: Liskov Substitution Principle (LSP)

**The Liskov Substitution Principle states that objects of a superclass should be replaceable with objects of a subclass without affecting the correctness of the program.**

This means that a subclass should override the parent class methods in a way that doesn't break functionality from a client's perspective.

### Violating LSP

Let's look at a classic example with birds:

```swift
class Bird {
    func fly() {
        print("Flying high")
    }
}

class Duck: Bird {
    override func fly() {
        print("Duck flying")
    }
}

class Penguin: Bird {
    override func fly() {
        // Penguins can't fly!
        fatalError("Penguins can't fly")
    }
}

// Usage
func makeBirdFly(bird: Bird) {
    // This should work with ANY bird
    bird.fly()
}

// Works fine
let duck = Duck()
makeBirdFly(bird: duck)

// Runtime error!
let penguin = Penguin()
makeBirdFly(bird: penguin)  // This will crash
```

This violates LSP because we can't substitute a `Penguin` for a `Bird` without breaking the program. A function expecting a `Bird` can't use a `Penguin` reliably.

### Applying LSP

Let's refactor the code to respect LSP:

```swift
// Base class with common behavior
class Bird {
    func eat() {
        print("Eating")
    }
    
    func sleep() {
        print("Sleeping")
    }
}

// Protocol for flying birds
protocol FlyingBird {
    func fly()
}

// Duck can fly
class Duck: Bird, FlyingBird {
    func fly() {
        print("Duck flying")
    }
}

// Penguin can't fly, so it doesn't conform to FlyingBird
class Penguin: Bird {
    func swim() {
        print("Swimming")
    }
}

// Usage
func makeItFly(bird: FlyingBird) {
    bird.fly()
}

// Works fine
let duck = Duck()
makeItFly(bird: duck)

// Won't compile - type checking prevents the error
let penguin = Penguin()
// makeItFly(bird: penguin)  // Compiler error - Penguin doesn't conform to FlyingBird
```

Now our code respects LSP. We only pass birds that can fly to functions requiring flying capability.

### Few Tips for LSP

- Consider UIKit's view hierarchy: any `UIView` should be substitutable where a `UIView` is expected
- When subclassing UIKit components, honor the superclass's behavior expectations
- Use protocols instead of inheritance when appropriate to define behaviors
- Apple's frameworks are designed with LSP in mind - for instance, you can substitute any `UIViewController` type in a navigation controller

## I: Interface Segregation Principle (ISP)

**The Interface Segregation Principle states that no client should be forced to depend on methods it does not use.**

In Swift terms, it means we should favor smaller, more focused protocols over large, monolithic ones.

### Violating ISP

Consider an app with different document types:

```swift
protocol Document {
    func open()
    func save()
    func print()
    func edit()
    func sign()
    func encrypt()
    func scan()
}

// Now every document type must implement ALL methods
class PDFDocument: Document {
    func open() { /* Implementation */ }
    func save() { /* Implementation */ }
    func print() { /* Implementation */ }
    func edit() { /* Implementation */ }
    func sign() { /* Implementation */ }
    func encrypt() { /* Implementation */ }
    func scan() { fatalError("Can't scan a PDF") }  // Doesn't make sense
}

class PhotoDocument: Document {
    func open() { /* Implementation */ }
    func save() { /* Implementation */ }
    func print() { /* Implementation */ }
    func edit() { /* Implementation */ }
    func sign() { fatalError("Can't sign a photo") }  // Doesn't make sense
    func encrypt() { /* Implementation */ }
    func scan() { fatalError("Can't scan a photo") }  // Doesn't make sense
}
```

This violates ISP because both `PDFDocument` and `PhotoDocument` are forced to implement methods they don't need or can't sensibly support.

### Applying ISP

Let's refactor with more focused protocols:

```swift
// Base protocol with common functionality
protocol Document {
    func open()
    func save()
}

// Optional functionality as separate protocols
protocol Printable {
    func print()
}

protocol Editable {
    func edit()
}

protocol Signable {
    func sign()
}

protocol Encryptable {
    func encrypt()
}

protocol Scannable {
    func scan()
}

// Now documents only implement what they need
class PDFDocument: Document, Printable, Editable, Signable, Encryptable {
    func open() { /* Implementation */ }
    func save() { /* Implementation */ }
    func print() { /* Implementation */ }
    func edit() { /* Implementation */ }
    func sign() { /* Implementation */ }
    func encrypt() { /* Implementation */ }
}

class PhotoDocument: Document, Printable, Editable, Encryptable {
    func open() { /* Implementation */ }
    func save() { /* Implementation */ }
    func print() { /* Implementation */ }
    func edit() { /* Implementation */ }
    func encrypt() { /* Implementation */ }
}

// Usage for specific capabilities
func printDocument(doc: Printable) {
    doc.print()
}

func signDocument(doc: Signable) {
    doc.sign()
}
```

Now each document type only needs to implement the protocols that match its capabilities.

### Few Tips for ISP

- UIKit uses this principle extensively with protocols like `UITableViewDataSource` and `UITableViewDelegate`
- In SwiftUI, the modifier pattern follows ISP by letting you opt-in to specific behaviors
- Consider using protocol composition (with `&` operator) for required combinations of behaviors
- Swift's protocol extensions allow adding default implementations to reduce boilerplate

## D: Dependency Inversion Principle (DIP)

**The Dependency Inversion Principle states that high-level modules should not depend on low-level modules; both should depend on abstractions.**

Furthermore, abstractions should not depend on details; details should depend on abstractions.

### Violating DIP

Consider a networking layer in an iOS app:

```swift
class UserService {
    private let urlSession = URLSession.shared
    
    func fetchUsers(completion: @escaping ([User]?, Error?) -> Void) {
        let url = URL(string: "https://api.example.com/users")!
        
        urlSession.dataTask(with: url) { data, response, error in
            if let error = error {
                completion(nil, error)
                return
            }
            
            guard let data = data else {
                completion(nil, NSError(domain: "No data", code: 0, userInfo: nil))
                return
            }
            
            do {
                let users = try JSONDecoder().decode([User].self, from: data)
                completion(users, nil)
            } catch {
                completion(nil, error)
            }
        }.resume()
    }
}

// Usage
class UserViewModel {
    private let userService = UserService()
    
    func loadUsers(completion: @escaping ([User]) -> Void) {
        userService.fetchUsers { users, error in
            if let users = users {
                completion(users)
            } else {
                // Handle error
                print(error?.localizedDescription ?? "Unknown error")
                completion([])
            }
        }
    }
}
```

This violates DIP because `UserViewModel` (high-level module) directly depends on `UserService` (low-level module), making it difficult to swap out the implementation or test the code.

### Applying DIP

Let's refactor using protocol abstractions:

```swift
// Abstract protocol for any user data source
protocol UserDataSource {
    func fetchUsers(completion: @escaping ([User]?, Error?) -> Void)
}

// Implementation that depends on the abstraction
class APIUserService: UserDataSource {
    private let urlSession: URLSession
    
    init(urlSession: URLSession = .shared) {
        self.urlSession = urlSession
    }
    
    func fetchUsers(completion: @escaping ([User]?, Error?) -> Void) {
        let url = URL(string: "https://api.example.com/users")!
        
        urlSession.dataTask(with: url) { data, response, error in
            if let error = error {
                completion(nil, error)
                return
            }
            
            guard let data = data else {
                completion(nil, NSError(domain: "No data", code: 0, userInfo: nil))
                return
            }
            
            do {
                let users = try JSONDecoder().decode([User].self, from: data)
                completion(users, nil)
            } catch {
                completion(nil, error)
            }
        }.resume()
    }
}

// Mock implementation for testing
class MockUserService: UserDataSource {
    private let mockUsers: [User]
    private let shouldFail: Bool
    
    init(mockUsers: [User] = [], shouldFail: Bool = false) {
        self.mockUsers = mockUsers
        self.shouldFail = shouldFail
    }
    
    func fetchUsers(completion: @escaping ([User]?, Error?) -> Void) {
        if shouldFail {
            completion(nil, NSError(domain: "Mock error", code: 0, userInfo: nil))
        } else {
            completion(mockUsers, nil)
        }
    }
}

// View model now depends on abstraction
class UserViewModel {
    private let userDataSource: UserDataSource
    
    // Dependency injection
    init(userDataSource: UserDataSource) {
        self.userDataSource = userDataSource
    }
    
    func loadUsers(completion: @escaping ([User]) -> Void) {
        userDataSource.fetchUsers { users, error in
            if let users = users {
                completion(users)
            } else {
                // Handle error
                print(error?.localizedDescription ?? "Unknown error")
                completion([])
            }
        }
    }
}

// Usage
let apiService = APIUserService()
let viewModel = UserViewModel(userDataSource: apiService)

// For testing
let mockService = MockUserService(mockUsers: [User(id: 1, name: "Test User")])
let testViewModel = UserViewModel(userDataSource: mockService)
```

Now both the `UserViewModel` (high-level) and `APIUserService` (low-level) depend on the `UserDataSource` abstraction, not on each other.

### Few Tips for DIP

- Use dependency injection to provide implementations to classes that depend on abstractions
- Consider using a dependency injection container for complex apps
- Swift's protocol-oriented programming is perfect for implementing DIP
- UIKit patterns like delegation already implement DIP (e.g., `UITableViewDataSource`)
- Combine framework uses publishers and subscribers that depend on abstractions, not concrete implementations

## Implementing SOLID in Existing iOS Projects

Transforming an existing iOS project to follow SOLID principles can be challenging. Here are some practical approaches:

### Incremental Refactoring Strategy

1. **Start with the most problematic areas**: Identify classes with the most responsibilities or those that change frequently
2. **Apply SRP first**: Break large classes into smaller, focused components
3. **Introduce abstractions gradually**: Add protocols for key components to support OCP and DIP
4. **Use Swift extensions**: Organize code logically without dramatic restructuring
5. **Add tests as you refactor**: Ensure your changes don't break existing functionality

### Example Refactoring Path

Consider a typical iOS view controller with too many responsibilities:

```swift
class ProductViewController: UIViewController {
    // UI elements
    private let tableView = UITableView()
    
    // Data
    private var products: [Product] = []
    
    // Network
    private let urlSession = URLSession.shared
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        fetchProducts()
    }
    
    private func setupUI() {
        // Complex UI setup code
    }
    
    private func fetchProducts() {
        let url = URL(string: "https://api.example.com/products")!
        
        urlSession.dataTask(with: url) { [weak self] data, response, error in
            // Data parsing and error handling
            DispatchQueue.main.async {
                self?.tableView.reloadData()
            }
        }.resume()
    }
    
    // UITableViewDataSource methods
    
    // UITableViewDelegate methods
    
    // Analytics tracking
    
    // Navigation logic
}
```

#### Step 1: Apply SRP with extensions (minimal change)

```swift
// Main class
class ProductViewController: UIViewController {
    // UI elements
    private let tableView = UITableView()
    
    // Data
    private var products: [Product] = []
    
    // Network - still violating DIP
    private let urlSession = URLSession.shared
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        fetchProducts()
    }
    
    private func fetchProducts() {
        let url = URL(string: "https://api.example.com/products")!
        
        urlSession.dataTask(with: url) { [weak self] data, response, error in
            // Data parsing and error handling
            DispatchQueue.main.async {
                self?.tableView.reloadData()
            }
        }.resume()
    }
}

// MARK: - UI Setup
extension ProductViewController {
    private func setupUI() {
        // Complex UI setup code
    }
}

// MARK: - UITableViewDataSource
extension ProductViewController: UITableViewDataSource {
    // TableView data source methods
}

// MARK: - UITableViewDelegate
extension ProductViewController: UITableViewDelegate {
    // TableView delegate methods
}

// MARK: - Analytics
extension ProductViewController {
    func trackScreenView() {
        // Analytics logic
    }
}
```

#### Step 2: Extract Service Layer (apply SRP and DIP)

```swift
// Abstract protocol
protocol ProductService {
    func fetchProducts(completion: @escaping (Result<[Product], Error>) -> Void)
}

// Concrete implementation
class APIProductService: ProductService {
    private let urlSession: URLSession
    
    init(urlSession: URLSession = .shared) {
        self.urlSession = urlSession
    }
    
    func fetchProducts(completion: @escaping (Result<[Product], Error>) -> Void) {
        let url = URL(string: "https://api.example.com/products")!
        
        urlSession.dataTask(with: url) { data, response, error in
            // Implementation
        }.resume()
    }
}

// Updated ViewController
class ProductViewController: UIViewController {
    // UI elements
    private let tableView = UITableView()
    
    // Data
    private var products: [Product] = []
    
    // Dependency
    private let productService: ProductService
    
    // Dependency injection
    init(productService: ProductService) {
        self.productService = productService
        super.init(nibName: nil, bundle: nil)
    }
    
    required init?(coder: NSCoder) {
        self.productService = APIProductService()
        super.init(coder: coder)
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        fetchProducts()
    }
    
    private func fetchProducts() {
        productService.fetchProducts { [weak self] result in
            // Handle result
            DispatchQueue.main.async {
                self?.tableView.reloadData()
            }
        }
    }
}
```

#### Step 3: Extract UI components (continue applying SRP)

```swift
class ProductCell: UITableViewCell {
    // Cell implementation
}

class ProductsView: UIView {
    private let tableView = UITableView()
    
    // UI implementation
    
    func update(with products: [Product]) {
        // Update UI
    }
}

// Further simplified ViewController
class ProductViewController: UIViewController {
    private let productsView = ProductsView()
    private let productService: ProductService
    
    init(productService: ProductService) {
        self.productService = productService
        super.init(nibName: nil, bundle: nil)
    }
    
    // ...
}
```

## Benefits of SOLID in iOS Development

Applying SOLID principles to your iOS projects brings numerous advantages:

1. **Increased testability**: Smaller, focused components are easier to test
2. **Improved maintainability**: Changes to one part of the system have minimal impact on others
3. **Better code organization**: Clear separation of concerns makes code navigation easier
4. **Reduced bugs**: Well-defined interfaces and responsibilities lead to fewer unexpected behaviors
5. **Easier onboarding**: New team members can understand the codebase faster
6. **Future-proofing**: Code is more resilient to changing requirements
7. **Performance optimization**: Focused components allow targeted optimization

## Conclusion

SOLID principles provide a robust foundation for building maintainable, flexible iOS applications. While implementing them requires some upfront investment, the long-term benefits to your codebase and development process are substantial.

Remember that SOLID isn't about rigid rules but about principles that guide your design decisions. Apply them pragmatically, focusing on the areas where they bring the most value to your specific project.

As your Swift and iOS skills grow, these principles will become second nature, helping you create better software that stands the test of time.