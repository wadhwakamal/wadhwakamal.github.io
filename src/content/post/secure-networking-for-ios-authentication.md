---
layout: ../../layouts/post.astro
title: Implementing Secure Networking to Complement Your iOS Authentication System
description: Enhance your iOS authentication system by implementing secure networking
dateFormatted: Sep 08, 2024
---

Hey there, fellow iOS developers! Today we're diving into one of those topics that might seem intimidating at first but is absolutely crucial for building professional-grade apps: implementing secure networking to complement your authentication system.

If you've already built an authentication flow in your app (you know, the whole sign-up/login dance), you're only halfway there. Without proper secure networking practices, you might as well be shouting your users' data across a crowded room! Let's fix that and make your app fortress-level secure.

## Understanding the Security Challenge

Before we jump into code, let's wrap our heads around what we're trying to solve. When your app communicates with a server:

1. Data could be intercepted (man-in-the-middle attacks)
2. Servers could be impersonated (DNS spoofing)
3. Data could be tampered with in transit
4. Authentication tokens could be stolen

Each of these scenarios is a nightmare waiting to happen. But don't worry - iOS gives us some powerful tools to prevent them!

## 1. Enforcing HTTPS with App Transport Security

First things first - if you're not using HTTPS in 2025, we need to have a serious talk! Apple actually enforces this through App Transport Security (ATS), which blocks insecure connections by default.

```swift
// This is in your Info.plist file
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <false/>
</dict>
```

The cool thing about ATS is that it's not just checking for HTTPS - it's verifying that you're using modern, secure TLS protocols and cipher suites. It's like having a security expert constantly looking over your shoulder!

But wait, what if you absolutely need to connect to a legacy server that doesn't support the latest security standards? You can configure exceptions:

```swift
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSExceptionDomains</key>
    <dict>
        <key>legacy-api.example.com</key>
        <dict>
            <key>NSExceptionAllowsInsecureHTTPLoads</key>
            <false/>
            <key>NSExceptionMinimumTLSVersion</key>
            <string>TLSv1.2</string>
            <key>NSExceptionRequiresForwardSecrecy</key>
            <true/>
        </dict>
    </dict>
</dict>
```

Think of these exceptions like leaving a single window open in your security system - sometimes necessary, but always a calculated risk!

## 2. Certificate Pinning: Trust No One

HTTPS is great, but what if someone manages to compromise a certificate authority or performs a sophisticated man-in-the-middle attack? This is where certificate pinning comes in - it's like giving your app a photo ID of your server and saying "only trust this exact face!"

Here's how to implement it using URLSession:

```swift
class NetworkManager: NSObject, URLSessionDelegate {

    private let trustedCertificates: [Data] = {
        let certificateURLs = Bundle.main.urls(forResourcesWithExtension: "cer", subdirectory: nil) ?? []
        return certificateURLs.compactMap { try? Data(contentsOf: $0) }
    }()

    lazy var session: URLSession = {
        let configuration = URLSessionConfiguration.default
        return URLSession(configuration: configuration, delegate: self, delegateQueue: nil)
    }()

    func urlSession(_ session: URLSession, didReceive challenge: URLAuthenticationChallenge,
                   completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void) {

        guard let serverTrust = challenge.protectionSpace.serverTrust,
              let certificate = SecTrustGetCertificateAtIndex(serverTrust, 0) else {
            completionHandler(.cancelAuthenticationChallenge, nil)
            return
        }

        // Get server certificate data
        let serverCertificateData = SecCertificateCopyData(certificate) as Data

        // Check if certificate is trusted
        if trustedCertificates.contains(serverCertificateData) {
            completionHandler(.useCredential, URLCredential(trust: serverTrust))
        } else {
            completionHandler(.cancelAuthenticationChallenge, nil)
        }
    }
}
```

This might look intimidating, but here's what it's doing:

1. Loading trusted certificates bundled with your app
2. Checking if the server's certificate matches one of these trusted certificates
3. If it matches, proceed; if not, cancel the connection

For a real-world analogy, imagine you're meeting someone from an online dating app. Instead of trusting that they are who they say they are just because they showed up, you've asked for a specific identifying mark or object that only the real person would have!

## 3. Token Management: Keeping Your Keys Safe

Now that we've secured the communication channel, let's talk about how to handle those precious authentication tokens.

### Secure Storage with the Keychain

First rule of token club: never store tokens in UserDefaults! Instead, use the iOS Keychain:

```swift
class KeychainManager {
    enum KeychainError: Error {
        case itemAlreadyExists
        case itemNotFound
        case unexpectedStatus(OSStatus)
    }

    static func save(token: String, forKey key: String) throws {
        let tokenData = token.data(using: .utf8)!

        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: key,
            kSecValueData as String: tokenData,
            kSecAttrAccessible as String: kSecAttrAccessibleWhenUnlockedThisDeviceOnly
        ]

        let status = SecItemAdd(query as CFDictionary, nil)

        if status == errSecDuplicateItem {
            // Item already exists, update it
            let updateQuery: [String: Any] = [
                kSecClass as String: kSecClassGenericPassword,
                kSecAttrAccount as String: key
            ]

            let attributesToUpdate: [String: Any] = [
                kSecValueData as String: tokenData
            ]

            let updateStatus = SecItemUpdate(updateQuery as CFDictionary, attributesToUpdate as CFDictionary)
            guard updateStatus == errSecSuccess else {
                throw KeychainError.unexpectedStatus(updateStatus)
            }
        } else if status != errSecSuccess {
            throw KeychainError.unexpectedStatus(status)
        }
    }

    static func retrieveToken(forKey key: String) throws -> String {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: key,
            kSecReturnData as String: true,
            kSecMatchLimit as String: kSecMatchLimitOne
        ]

        var item: CFTypeRef?
        let status = SecItemCopyMatching(query as CFDictionary, &item)

        guard status == errSecSuccess else {
            throw KeychainError.unexpectedStatus(status)
        }

        guard let tokenData = item as? Data,
              let token = String(data: tokenData, encoding: .utf8) else {
            throw KeychainError.itemNotFound
        }

        return token
    }

    static func deleteToken(forKey key: String) throws {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: key
        ]

        let status = SecItemDelete(query as CFDictionary)
        guard status == errSecSuccess || status == errSecItemNotFound else {
            throw KeychainError.unexpectedStatus(status)
        }
    }
}
```

The Keychain is iOS's secure vault - it's encrypted at rest and protected by the device's security systems. Think of it as the difference between hiding your house key under the doormat (UserDefaults) versus putting it in a locked safe (Keychain).

### Token Refresh Strategy

Tokens should expire - it's a security best practice. Here's how to handle refreshing them:

```swift
class AuthNetworkManager {
    private var accessToken: String?
    private var refreshToken: String?
    private var tokenExpirationDate: Date?

    private let networkManager = NetworkManager()

    func authenticatedRequest<T: Decodable>(request: URLRequest) async throws -> T {
        var clientRequest = request
        // Check if token is expired or about to expire
        if needsTokenRefresh() {
            try await refreshAccessToken()
        }

        guard let token = accessToken else {
            throw AuthError.notAuthenticated
        }

        clientRequest.addValue("Bearer \(token)", forHTTPHeaderField: "Authorization")

        let (data, response) = try await networkManager.session.data(for: request)

        // Check if we got a 401 Unauthorized (token might have been invalidated)
        if let httpResponse = response as? HTTPURLResponse, httpResponse.statusCode == 401 {
            // Try to refresh token and retry
            try await refreshAccessToken()
            return try await authenticatedRequest(request: request)
        }

        return try JSONDecoder().decode(T.self, from: data)
    }

    private func needsTokenRefresh() -> Bool {
        guard let expirationDate = tokenExpirationDate else {
            return true
        }

        // Refresh if token expires in less than 5 minutes
        return expirationDate.timeIntervalSinceNow < 300
    }

    private func refreshAccessToken() async throws {
        guard let refreshToken = refreshToken else {
            throw AuthError.notAuthenticated
        }

        var request = URLRequest(url: URL(string: "https://api.example.com/refresh")!)
        request.httpMethod = "POST"
        request.addValue("application/json", forHTTPHeaderField: "Content-Type")

        let refreshBody = ["refresh_token": refreshToken]
        request.httpBody = try JSONEncoder().encode(refreshBody)

        let (data, _) = try await networkManager.session.data(for: request)
        let tokenResponse = try JSONDecoder().decode(TokenResponse.self, from: data)

        // Update tokens
        self.accessToken = tokenResponse.accessToken
        if let newRefreshToken = tokenResponse.refreshToken {
            self.refreshToken = newRefreshToken
            try KeychainManager.save(token: newRefreshToken, forKey: "refreshToken")
        }
        self.tokenExpirationDate = Date().addingTimeInterval(tokenResponse.expiresIn)

        try KeychainManager.save(token: tokenResponse.accessToken, forKey: "accessToken")
    }

    enum AuthError: Error {
        case notAuthenticated
        case refreshFailed
    }

    struct TokenResponse: Decodable {
        let accessToken: String
        let refreshToken: String?
        let expiresIn: TimeInterval

        enum CodingKeys: String, CodingKey {
            case accessToken = "access_token"
            case refreshToken = "refresh_token"
            case expiresIn = "expires_in"
        }
    }
}
```

This approach is like having a smart key system for your house - the key automatically renews itself before it expires, and if someone tries to revoke your key remotely, you've got a backup way to get a new one.

## 4. Implementing a Secure API Client

Now let's tie everything together with a clean, secure API client:

```swift
/// A secure API client for handling network requests with authentication
class SecureAPIClient {
    // MARK: - Properties

    private let baseURL: URL
    private let networkManager: NetworkManager
    private let authManager: AuthNetworkManager

    // MARK: - Initialization

    init(baseURL: URL) {
        self.baseURL = baseURL
        self.networkManager = NetworkManager()
        self.authManager = AuthNetworkManager()
    }

    // MARK: - Public Methods

    /// Performs an authenticated request to the API
    /// - Parameters:
    ///   - endpoint: The API endpoint path (will be appended to baseURL)
    ///   - method: HTTP method (GET, POST, PUT, DELETE, etc.)
    ///   - parameters: Optional parameters to include in the request
    ///   - requiresAuth: Whether this endpoint requires authentication
    /// - Returns: Decoded response of the specified type
    func request<T: Decodable>(
        endpoint: String,
        method: HTTPMethod,
        parameters: [String: String]? = nil,
        requiresAuth: Bool = true
    ) async throws -> T {

        let url = baseURL.appendingPathComponent(endpoint)
        var request = URLRequest(url: url)
        request.httpMethod = method.rawValue
        request.addValue("application/json", forHTTPHeaderField: "Content-Type")

        // Add parameters based on method type
        if let parameters = parameters {
            switch method {
            case .get:
                // For GET requests, encode parameters as URL query items
                if var components = URLComponents(url: url, resolvingAgainstBaseURL: true) {
                    let encoder = JSONEncoder()
                    let data = try encoder.encode(parameters)
                    if let dict = try JSONSerialization.jsonObject(with: data) as? [String: Any] {
                        components.queryItems = dict.map { URLQueryItem(name: $0.key, value: "\($0.value)") }
                        if let newURL = components.url {
                            request.url = newURL
                        }
                    }
                }
            default:
                // For non-GET requests, encode parameters in the body
                let encoder = JSONEncoder()
                request.httpBody = try encoder.encode(parameters)
            }
        }

        if requiresAuth {
            // Use the auth manager for authenticated requests
            return try await authManager.authenticatedRequest(request: request)
        } else {
            // Use the network manager for non-authenticated requests
            let (data, response) = try await networkManager.session.data(for: request)

            // Validate response
            try validateResponse(data: data, response: response)

            // Decode the response
            let decoder = JSONDecoder()
            decoder.keyDecodingStrategy = .convertFromSnakeCase
            return try decoder.decode(T.self, from: data)
        }
    }

    // MARK: - Private Methods

    private func validateResponse(data: Data, response: URLResponse) throws {
        guard let httpResponse = response as? HTTPURLResponse else {
            throw APIError.invalidResponse
        }

        switch httpResponse.statusCode {
        case 200...299:
            // Success! No action needed
            return
        case 401:
            throw APIError.unauthorized
        case 403:
            throw APIError.forbidden
        case 404:
            throw APIError.notFound
        case 500...599:
            throw APIError.serverError(httpResponse.statusCode)
        default:
            throw APIError.unexpectedStatusCode(httpResponse.statusCode)
        }
    }

    // MARK: - Nested Types

    enum HTTPMethod: String {
        case get = "GET"
        case post = "POST"
        case put = "PUT"
        case patch = "PATCH"
        case delete = "DELETE"
    }

    enum APIError: Error, LocalizedError {
        case invalidResponse
        case unauthorized
        case forbidden
        case notFound
        case serverError(Int)
        case unexpectedStatusCode(Int)

        var errorDescription: String? {
            switch self {
            case .invalidResponse:
                return "The server returned an invalid response."
            case .unauthorized:
                return "You are not authorized to perform this action."
            case .forbidden:
                return "Access to this resource is forbidden."
            case .notFound:
                return "The requested resource was not found."
            case .serverError(let code):
                return "A server error occurred: \(code)"
            case .unexpectedStatusCode(let code):
                return "An unexpected error occurred: \(code)"
            }
        }
    }
}

// MARK: - Usage Examples

// Example models
struct UserProfile: Codable {
    let id: String
    let name: String
    let email: String
    let profileImageUrl: String?
}

struct LoginParameters: Codable {
    let email: String
    let password: String
}

// Example API calls
extension SecureAPIClient {
    func login(email: String, password: String) async throws -> Bool {
        let parameters = ["email": email, "password": password]
        let response: AuthResponse = try await request(
            endpoint: "auth/login",
            method: .post,
            parameters: parameters,
            requiresAuth: false
        )

        // Store tokens
        try KeychainManager.save(token: response.accessToken, forKey: "accessToken")
        try KeychainManager.save(token: response.refreshToken, forKey: "refreshToken")

        return true
    }

    func fetchUserProfile() async throws -> UserProfile {
        return try await request(
            endpoint: "user/profile",
            method: .get,
            parameters: nil
        )
    }

    func updateUserProfile(name: String) async throws -> UserProfile {
        struct Parameters: Codable {
            let name: String
        }

        return try await request(
            endpoint: "user/profile",
            method: .patch,
            parameters: ["name": name]
        )
    }

    struct AuthResponse: Decodable {
        let accessToken: String
        let refreshToken: String
        let expiresIn: Int
    }
}
```

Our `SecureAPIClient` is a beautifully structured class that handles all the security concerns we've discussed while providing a clean, easy-to-use interface for the rest of your app.

## 5. Testing Your Security Implementation

Security isn't just about writing the code - it's about verifying it works as expected. Here are some ways to test your implementation:

### Charles Proxy Testing

Use Charles Proxy to inspect network traffic and verify:

- Your app uses HTTPS for all connections
- Your certificate pinning blocks invalid certificates
- Your authentication headers are correctly sent

### Penetration Testing

Consider hiring a security specialist to perform penetration testing on your app. They'll try to:

- Intercept network traffic
- Extract tokens from the device
- Bypass authentication mechanisms

## 6. Best Practices and Common Pitfalls

Let's wrap up with some battle-tested wisdom:

### Do:

- Implement certificate pinning for high-security apps
- Use the Keychain for storing sensitive data
- Implement token refresh strategies
- Log users out automatically after periods of inactivity
- Sanitize all user input before sending to your server

### Don't:

- Store tokens or passwords in UserDefaults or plain text files
- Hardcode API keys or secrets in your source code
- Trust data received from the server without validation
- Log sensitive information (even in debug builds)
- Forget to update your security measures as new vulnerabilities are discovered

## Conclusion

Building a secure networking layer isn't just a checkbox for your app - it's an ongoing commitment to protecting your users' data. The techniques we've covered are the foundation of a robust security strategy that complements your authentication system.

Remember, the most secure code is the one that anticipates attacks and prepares for them before they happen. It's like the famous quote from Sun Tzu’s The Art of War:

> Victorious warriors win first and then go to war, while defeated warriors go to war first and then seek to win.

In security, we need to think about where the vulnerabilities might be, not just the ones we already know about.

If you're looking to level up your app's security even further, consider exploring topics like:

- End-to-end encryption for messaging apps
- Secure offline storage with CoreData and encryption
- Advanced certificate pinning techniques with trust evaluation

Keep coding securely! 🔐👩‍💻👨‍💻
