# XSStyle Architecture 

### XSStyle framework is open for micro-frontend approach and focuses on reducing the need for manually creating views in a UI, and instead, it allows you to create views by using style models. 

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
            SDK_2(),                    // google    A (adapter)
            SDK_3(),                    // facebook
            SDK_4(),                    // tracker   |
        ])                              // ----------
    }
}

```


**Connector** working with simple Composite [pattern](https://refactoring.guru/design-patterns/composite) and connects sub connections which is working for proper SDK.

```swift

class SDK_1: Connector {                            // ---------------
    convenience init() {                            //                |
        self.init(connections: [                    //                |
            FirebaseCrashlyticsConnector(),         // mod1 ----               
            FirebaseAnalyticsConnector(),           // mod2 ---- |- SDK
            FirebaseDatabaseConnector(),            // mod3 ----               
        ])                                          //                |
    }                                               //                |
}                                                   // ---------------

class SDK_2: Connector {                            // ---------------
    convenience init() {                            //                |
        self.init(connections: [                    //                |
            GoogleCrashlyticsConnector(),           // mod1 ----               
            GoogleAnalyticsConnector(),             // mod2 ---- |- SDK
            GoogleDatabaseConnector(),              // mod3 ----               
        ])                                          //                |
    }                                               //                |
}                                                   // ---------------


class SDK_3: Connector {                            // ---------------
    convenience init() {                            //                |
        self.init(connections: [                    //                |
            FacebookCrashlyticsConnector(),         // mod1 ----               
            FacebookAnalyticsConnector(),           // mod2 ---- |- SDK
            FacebookDatabaseConnector(),            // mod3 ----               
        ])                                          //                |
    }                                               //                |
}                                                   // ---------------


```


Clean  **Coordinator** connects Wallet App coordinator [WalletApp](https://apps.apple.com/az/app/wallet-budget-money-manager/id1032467659).

```swift

// MARK: Wallet Coordinator

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

let app = Wallet()

app.move(to: .authorize(.view1(data)))         // will open authorize module first view.
app.move(to: .authorize(.view2(data)))         // will open authorize module second view.
app.move(to: .authorize(.view3(data)))         // will open authorize module third view.

app.move(to: .dashboard(.tab1(.mod1(data))))   // will open the first module under first tab.
app.move(to: .dashboard(.tab2(.mod1(data))))   // will open the first module under second tab.
app.move(to: .dashboard(.tab3(.mod2(data))))   // will open the second module under third tab.
app.move(to: .dashboard(.tab4(.mod3(data))))   // will open the third module under fourth tab.



```

Lets deep dive into the **Products** app coordinator to see what is inside, all micro apps working with clean MVVM-C pattern.

```swift


// MARK: Products app Coordinator

public class Products: Coordinator<ProductsRoute> {                     // ---------------
    @discardableResult                                                  //                |
    override public func move(by route: ProductsRoute) -> Self {        //                |
        switch route {                                                  //                |
        case .cards(let route):                                         //                |
            let c = Cards(route)                                        // C ---------    |
            show(v: c)                                                  //            |   |
        case .accounts(let route):                                      //            |   |
            let c = Accounts(route)                                     // C ---------|
            show(v: c)                                                  //            |-- C
        case .deposits(let route):                                      //            |
            let c = Deposits(route)                                     // C ---------|   |
            show(v: c)                                                  //            |   |
        case .loans(let route):                                         //            |   |
            let c = Loans(route)                                        // C ---------    |
            show(v: c)                                                  //                |
        }                                                               //                |
        return self                                                     //                |
    }                                                                   //                |
}                                                                       // ---------------

let app = Products()
app.move(to: .cards(.view1(data)))         // will open cards module first view.
app.move(to: .accounts(.view2(data)))      // will open accounts module second view.
app.move(to: .deposits(.view3(data)))      // will open deposits module third view.
app.move(to: .loans(.view3(data)))         // will open loans module third view.


```



Lets deep dive to the **Authorize & Payments** apps coordinator to see what is inside, all micro apps working with clean MVVM-C pattern.

```swift

// MARK: Cards (Model View ViewModel Coordinator) - MVVM-C

public class Cards: Coordinator<CardsRoute> {                           // ---------------
    @discardableResult                                                  //                |
    override public func move(by route: CardsRoute) -> Self {           //                |
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

// MARK: Accounts (Model View ViewModel Coordinator) - MVVM-C

public class Accounts: Coordinator<AccountsRoute> {                     // ---------------
    @discardableResult                                                  //                |
    override public func move(by route: AccountsRoute) -> Self {        //                |
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


let app = Cards()                 let app = Accounts()
app.move(to: .view1(data))        app.move(to: .view1(data))
app.move(to: .view2(data))        app.move(to: .view2(data))
app.move(to: .view3(data))        app.move(to: .view3(data)) 


```

**SINGLE MVVM LAYER** 

```swift


// MARK: **Model** (m)

class Model1: ViewStyle<Transaction> {
        
    override var sectionsStyle: [SectionStyle] {
        
        // fill section manual indirect...
        let s1 = SectionStyle()
        
        // component1
        let s1_c1 = ImageLabelButtonComponentStyle(
            s1: .style(.s1(data?.amount)),
            s2: .style(.s2("Hello")),
            s3: .style(.s3(.s1("https://"), event: event(_:))))
            .didSelect { _ in
                self.move(by: .view(data))
            }.willMove { _ in
                self.move(by: .view(data))
            }.didMove { _ in
                self.move(by: .view(data))
            }.isHidden {
                data.amount.isEmpty
            }.isShimmering {
                data.isEmpty
            }
        
        // component2
        let s1_c2 = ImageLabelLabelComponentStyle2(
            s1: .style(.s1("https://via.placeholder.com/150")),
            s2: .style(.s1("Hello...")),
            s3: .style(.s2("Hello...")))
            .isHidden {
                data.amount.isEmpty
            }.isShimmering {
                data.isEmpty
            }
        
        s1.add(s1_c1)
        s1.add(s1_c2)
        
        return [ s1 ]
    }
}


// MARK: **View** (v)
typealias View1 = View<Model1>


// MARK: **View Model** (vm)
class ViewModel1: ViewModel<Model1> {
    @discardableResult
    override func move(by c: Coordinator<Route>) -> Self {
        NetworkService.shared
            .requestTransaction(model?.id) {
                $0.configure(.init($1))
        })
        return self
    }
}


```

// 
// 

![alt text](https://github.com/KhayalSuleymani/XSStyle/blob/master/Documents/UnitTests/unit_tests_coverage.png)

// 
// 

Style models **Inheritance** hierarchies.


