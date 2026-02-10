# XSStyle

### XSStyle framework focuses on reducing the need for manually creating views in a UI, and instead, it allows you to create views by using style models. 

Lets start from begining. 

Clean **AppDelegate** .

```swift

// MARK: App Delegate

@main class Main: App {
    
    /// app coordinator connection
    override var controller: Wallet {
        Wallet()                        // -------- C
    }
    
    /// app third parties connection
    override var connector: Connector { // ----------
        .init(connections: [            //           |
            SDK_1(),                    // firebase
            SDK_2(),                    // evam      A (adapter)
            SDK_3(),                    // adjust
            SDK_4(),                    // tracker   |
        ])                              // ----------
    }
}


```
Clean  **Coordinator** connects Wallet App coordinator [WalletApp](https://apps.apple.com/az/app/wallet-budget-money-manager/id1032467659).

```swift

// MARK: Test Coordinator

public class Wallet: Coordinator<AppRoute> {                            // ---------------
    @discardableResult                                                  //                |
    override public func move(by route: AppRoute) -> Self {             //                |
        switch route {                                                  //                |
        case .authorize(let r):                                         //                |
            let c = Authorize(r)                                        // C --------     |
            show(v: c)                                                  //            |   |
        case .dashboard(_):                                             //            |
            let c = Tab(tabs: [                                         //            |-- C
                Tab1(),                                                 //            |
                Tab2(),                                                 //            |   |
                Tab3(),                                                 // C --------     |
                Tab4(),                                                 //                |
            ])                                                          //                |
            overFull(v: c)                                              //                |
        }                                                               //                |
        return self                                                     //                |
    }                                                                   //                |
}                                                                       // ---------------


```

**Wallet App manual Deeplinking**. Will open proper screen under Authorize and Tabbar apps. 

```swift

let app = Wallet()

app.move(to: .authorize(.signIn))
app.move(to: .authorize(.login))
app.move(to: .authorize(.signUp))
app.move(to: .authorize(.biometric))

app.move(to: .dashboard(.home(data)))
app.move(to: .dashboard(.plannings(data)))
app.move(to: .dashboard(.statistics(data)))
app.move(to: .dashboard(.more(data)))

```

Lets deep dive to the **Authorize & Payments** apps coordinator to see what is inside, all micro apps working with clean MVVM-C pattern.

```swift

// MARK: Transactions (Model View ViewModel Coordinator) - MVVM-C

public class Authorize: Coordinator<AuthorizeRoute> {                   // ---------------
    @discardableResult                                                  //                |
    override public func move(by route: AuthorizeRoute) -> Self {       //                |
        switch route {                                                  //                |
        case .view1(let d):                                             //                |
            let m = Model1(d)                                           // m              |
            let v = View1()                                             // v  --------    |
            let _ = ViewModel1(m)                                       // vm         |   |
                .subscribe(v)                                           //            |   |
                .move(by: self)                                         //            |   |
            push(v: v)                                                  //            |   |
        case .view2(let d):                                             //            |   |
            let m = Model2(d)                                           // m          |
            let v = View2()                                             // v  --------|-- C
            let _ = ViewModel2(m)                                       // vm         |
                .subscribe(v)                                           //            |   |
                .move(by: self)                                         //            |   |
            push(v: v)                                                  //            |   |
        case .view3(let d):                                             //            |   |
            let m = Model3(d)                                           // m          |   |
            let v = View3()                                             // v  --------    |
            let _ = ViewModel3(m)                                       // vm             |
                .subscribe(v)                                           //                |
                .move(by: self)                                         //                |
            present(v: v)                                               //                |
        }                                                               //                |
        return self                                                     //                |
    }                                                                   //                |
}                                                                       // ---------------


public class Payments: Coordinator<PaymentsRoute> {                     // ---------------
    @discardableResult                                                  //                |
    override public func move(by route: PaymentsRoute) -> Self {        //                |
        switch route {                                                  //                |
        case .view1(let d):                                             //                |
            let m = Model1(d)                                           // m              |
            let v = View1()                                             // v  --------    |
            let _ = ViewModel1(m)                                       // vm         |   |
                .subscribe(v)                                           //            |   |
                .move(by: self)                                         //            |   |
            push(v: v)                                                  //            |   |
        case .view2(let d):                                             //            |   |
            let m = Model2(d)                                           // m          |
            let v = View2()                                             // v  --------|-- C
            let _ = ViewModel2(m)                                       // vm         |
                .subscribe(v)                                           //            |   |
                .move(by: self)                                         //            |   |
            push(v: v)                                                  //            |   |
        case .view3(let d):                                             //            |   |
            let m = Model3(d)                                           // m          |   |
            let v = View3()                                             // v  --------    |
            let _ = ViewModel3(m)                                       // vm             |
                .subscribe(v)                                           //                |
                .move(by: self)                                         //                |
            present(v: v)                                               //                |
        }                                                               //                |
        return self                                                     //                |
    }                                                                   //                |
}                                                                       // ---------------

```

**Authorize App manual Deeplinking**. Will open proper screen under Authorize app.

```swift

let app = Authorize()            let app = Payments()
app.move(to: view1(data))        app.move(to: view1(data))
app.move(to: view2(data))        app.move(to: view2(data))
app.move(to: view3(data))        app.move(to: view3(data)) 

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
