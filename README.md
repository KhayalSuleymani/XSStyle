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
**Coordinator** connects Wallet App coordinator [WalletApp](https://apps.apple.com/az/app/wallet-budget-money-manager/id1032467659).

```swift

class Wallet: Coordinator<AppRoute> {
    override func move(to route: AppRoute) -> Transition {
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

**Wallet App manual Deeplinking**. Will open proper screen under Authorize and Tabbar apps. 

```swift

let app = Wallet()

app.move(to: .authorize(.signIn))
app.move(to: .authorize(.login))
app.move(to: .authorize(.signUp))
app.move(to: .authorize(.biometric))

app.move(to: .tabbar(.dashboard(.home)))
app.move(to: .tabbar(.plannings(.plannings)))
app.move(to: .tabbar(.statistics(.statistics)))
app.move(to: .tabbar(.more(.more)))

```

Lets deep dive to the **Authorize** app coordinator working with clean MVVM-C pattern.

```swift

open class Authorize: Coordinator<AuthorizeRoute> {
    open override func prepareTransition(for route: AuthorizeRoute) -> NavigationTransition {
        switch route {
        case .login:
            let m = LoginViewStyle()
            let v = LoginView()
            let _ = LoginViewModel(m)
                .subscribe(v: v)
                .move(by: self)
            return .set([v])
        case .signUp:
            let m = SignUpViewStyle()
            let v = SignUpView()
            let _ = SignUpViewModel(m)
                .subscribe(v: v)
                .move(by: self)
            return .push(v)
        case .signIn:
            let m = SignInViewStyle()
            let v = SignInView()
            let _ = SignInViewModel(m)
                .subscribe(v: v)
                .move(by: self)
            return .push(v)
        case .biometric:
            let m = BiometricViewStyle()
            let v = BiometricView()
            let _ = BiometricViewModel(m)
                .subscribe(v: v)
                .move(by: self)
            return .push(v)
        }
    }
}

```

**Authorize App manual Deeplinking**. Will open proper screen under Authorize app.

```swift

let app = Authorize()
app.move(to: .signIn)
app.move(to: .login)
app.move(to: .signUp)
app.move(to: .biometric)

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
