# XSStyle

### XSStyle framework focuses on reducing the need for manually creating views in a UI, and instead, it allows you to create views by using style models. 

Lets start from begining. 

Clean **AppDelegate** .

```swift

@main class App: Responder {
    
    override var coordinator: Coordinator {
        Wallet()
    }

    override var connector: Connector {
        Connector(connections: [
            FirebaseConnector(), // Will connect Firebase SDK
            FacebookConnector(), // will connect Facebook SDK
            GoogleConnector(),   // Will Connect Google SDK
        ])
    }
}

```
**Connector** working with simple Composite [pattern](https://refactoring.guru/design-patterns/composite).
```swift

class FirebaseConnector: Connector {
    convenience init() {
        self.init(connections: [
            FirebaseCrashlyticsConnector(),
            FirebaseAnalyticsConnector(),
            FirebaseDatabaseConnector(),
        ])
    }
}

class GoogleConnector: Connector {
    convenience init() {
        self.init(connections: [
            GoogleCrashlyticsConnector(),
            GoogleAnalyticsConnector(),
            GoogleDatabaseConnector(),
        ])
    }
}

class FacebookConnector: Connector {
    convenience init() {
        self.init(connections: [
            FacebookCrashlyticsConnector(),
            FacebookAnalyticsConnector(),
            FacebookDatabaseConnector(),
        ])
    }
}

```





Style models **Inheritance** hierarchies.
