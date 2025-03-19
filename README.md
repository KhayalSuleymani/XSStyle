# XSStyle

### XSStyle framework focuses on reducing the need for manually creating views in a UI, and instead, it allows you to create views by using style models. 

Lets start from begining. 

Clean **AppDelegate** .

```swift

@main class App: Responder {
    
    override var coordinator: Coordinatable {
        Wallet()
    }

    override var connector: Connectable {
        Connector(connections: [
            FirebaseConnector(), // Will connect Firebase SDK
            GoogleConnector(),   // Will Connect Google SDK
            FacebookConnector(), // will connect Facebook SDK
        ])
    }
}

```
Clean  **Coordinator** connects Wallet App coordinator [WalletApp](https://apps.apple.com/az/app/wallet-budget-money-manager/id1032467659).

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

Lets deep dive to the **Authorize & Payments** apps coordinator to see what is inside, all micro apps working with clean MVVM-C pattern.

```swift

open class Authorize: Coordinator<AuthorizeRoute> {
    open override func move(to route: AuthorizeRoute) -> Transition {
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

open class Payments: Coordinator<PaymentsRoute> {
    open override func move(to route: PaymentsRoute) -> Transition {
        switch route {
        case .payments(let d):
            let m = PaymentsViewStyle(d)
            let v = PaymentsView()
            let _ = PaymentsViewModel(m)
                .subscribe(v: v)
                .move(by: self)
            return .push(v)
        case .payment(let d):
            let m = PaymentViewStyle(d)
            let v = PaymentView()
            let _ = PaymentViewModel(m)
                .subscribe(v: v)
                .move(by: self)
            return .push(v)
        case .merchant(let d):
            let m = MerchantViewStyle(d)
            let v = MerchantView()
            let _ = MerchantViewModel(m)
                .subscribe(v: v)
                .move(by: self)
            return .push(v)
        case .approve(let d):
            let m = ApproveViewStyle(d)
            let v = ApproveView()
            let _ = ApproveViewModel(m)
                .subscribe(v: v)
                .move(by: self)
            return .push(v)
        case .success(let d):
            let m = SuccessViewStyle(d)
            let v = SuccessView()
            let _ = SuccessViewModel(m)
                .subscribe(v: v)
                .move(by: self)
            return .present(v)
        }
    }
}

```

**Authorize App manual Deeplinking**. Will open proper screen under Authorize app.

```swift

let app = Authorize()            let app = Payments()
app.move(to: .signIn)            app.move(to: .payments(data))
app.move(to: .login)             app.move(to: .payment(data))
app.move(to: .signUp)            app.move(to: .merchant(data)) 
app.move(to: .biometric)         app.move(to: .approve(data))
                                 app.move(to: .success(data))


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
