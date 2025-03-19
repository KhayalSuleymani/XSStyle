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
            GoogleConnector(),   // Will Connect Google SDK
            FacebookConnector(), // will connect Facebook SDK
        ])
    }
}

```
**Coordinator** connects WalletApp coordinator [WalletApp](https://apps.apple.com/az/app/wallet-budget-money-manager/id1032467659).

```swift

class Wallet: Coordinator<AppRoute> {
    override func prepareTransition(for route: AppRoute) -> Transition {
        switch route {
        case .authorize(let route):
            let app = Authorize(route)
            return .set([app])
        case .tabbar:
            let app = TabBar(tabs: [
                Home(),
                Planning(),
                Statistics(),
                More(),
            ])
            return .push(app)
        }
    }
}

```

**Connector** working with simple Composite [pattern](https://refactoring.guru/design-patterns/composite) and connects sub connections which is working for proper SDK.

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
