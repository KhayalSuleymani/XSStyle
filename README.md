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

### SDK connection architecture
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

### Coordinator architecture
Clean  **Coordinator** connects Wallet App coordinator [WalletApp](https://apps.apple.com/az/app/wallet-budget-money-manager/id1032467659).

```swift

// MARK: Wallet Coordinator

public class Wallet: Coordinator<AppRoute> {                            // ---------------
    @discardableResult                                                  //                |
    override public func move(by route: AppRoute) -> Self {             //                |         
        switch route {                                                  //                |
        case .authorize(let route):                                     //                |
            let c = Authorize(route)                                    // C --------     |
            show(v: c)                                                  //            |   |
        case .dashboard(let route):                                     //            |
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

// MARK: MODULE UNIT TESTS
final class WalletTests: TestCase {    
    override func test () {
        let c = Wallet()
        c
            .move(by: .authorize(.view1(.mock)))                          // will open authorize module first view.
            .move(by: .dashboard(.tab1(.products(.mock))))                // will open the products module under tab1.
            .move(by: .dashboard(.tab2(.operations(.mock))))              // will open the operations module under tab2.
            .move(by: .dashboard(.tab3(.payments(.view1(.mock)))))        // will open the payments module under tab3.
            .move(by: .dashboard(.tab4(.products(.mock))))                // will open the products module under tab4.
    }
}

```

Lets deep dive into the **Products** app coordinator to see what is inside, all micro apps working with clean MVVM-C pattern.

```swift


// MARK: Products app Coordinator contains too many coordinators..

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

// MARK: MODULE UNIT TESTS
final class ProductsTests: TestCase {    
    override func test () {
        let c = Products()
        c
            app.move(to: .cards(.view1(data)))         // will open cards module first view.
            app.move(to: .accounts(.view2(data)))      // will open accounts module second view.
            app.move(to: .deposits(.view3(data)))      // will open deposits module third view.
            app.move(to: .loans(.view3(data)))         // will open loans module third view.
}


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

// MARK: CARDS MODULE UNIT TESTS
final class CardsTests: TestCase {    
    override func test () {
        let c = Cards()
        c
            app.move(to: .view1(data))     
            app.move(to: .view2(data))
            app.move(to: .view3(data)) 
}



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

// MARK: ACCOUNTS MODULE UNIT TESTS
final class AccountsTests: TestCase {    
    override func test () {
        let c = Accounts()
        c
            app.move(to: .view1(data))     
            app.move(to: .view2(data))
            app.move(to: .view3(data)) 
}

```

### MVVM architecture

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
typealias View1 = View<Model1> // no need viewcontroller anymore


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

**App Code Coverage written via XSStyle** 

![alt text](https://github.com/KhayalSuleymani/XSStyle/blob/master/Documents/UnitTests/unit_tests_coverage.png)

**nice :)** 

### Components architecture

![alt text](https://github.com/KhayalSuleymani/XSStyle/blob/master/Documents/Components/components_hierarchy.png)


Style models **Inheritance** hierarchies. let's suppose 

**X**  --> Label component <br />
**Y**  --> Button component <br />
**Z**  --> Image component <br />

examples should be...

**XY** --> LabelButtonComponent <br />
**YX** --> ButtonLabelComponent <br />

**XYZ** --> LabelButtonImageComponent <br />
**ZXY** --> ImageLabelButtonComponent <br />

Lets show, how component inheritance works in real example.


// first inheritance

```swift

class ImageComponent: Component<ImageComponentStyle> {

    weak var v1: ImageView!
    
    @discardableResult
    override func configure(_ s: ImageComponentStyle) -> Self {
        v1.configure(s.s1)
        super.configure(s)                                     
    }
}

class ImageLabelComponent: Component<ImageLabelComponentStyle> {
    
    weak var v1: ImageView!
    weak var v2: Label!
    
    @discardableResult
    override func configure(_ s: ImageLabelComponentStyle) -> Self {
        v1.configure(s.s1)
        v2.configure(s.s2)
        super.configure(s)
    }
}

class ImageLabelLabelComponent: Component<ImageLabelLabelComponentStyle> {

    weak var v1: ImageView!
    weak var v2: Label!
    weak var v3: Label!
    
    @discardableResult
    override func configure(_ s: ImageLabelLabelComponentStyle) -> Self {
        v1.configure(s.s1)
        v2.configure(s.s2)
        v3.configure(s.s3)
        super.configure(s)
    }
}

class ImageLabelLabelButtonComponent: Component<ImageLabelLabelComponentStyle> {

    weak var v1: ImageView!
    weak var v2: Label!
    weak var v3: Label!
    weak var v4: Button!

    @discardableResult
    override func configure(_ s: ImageLabelLabelButtonComponent) -> Self {
        v1.configure(s.s1)
        v2.configure(s.s2)
        v3.configure(s.s3)
        v4.configure(s.s4)
        super.configure(s)
    }
}

```

// second inheritance

```swift

open class ImageComponentStyle: ComponentStyle {
    
    let s1: ImageStyle
    
    public init(s1: ImageStyle) {
        self.s1 = s1
    }
}


open class ImageLabelComponentStyle: ImageComponentStyle {
    
    let s2: LabelStyle
    
    public init(s1: ImageStyle,
                s2: LabelStyle) {
        self.s2 = s2
        super.init(s1: s1)
    }
}


open class ImageLabelLabelComponentStyle: ImageLabelComponentStyle {
    
    let s3: LabelStyle
    
    public init(s1: ImageStyle,
                s2: LabelStyle,
                s3: LabelStyle) {
        self.s3 = s3
        super.init(s1: s1,
                   s2: s2)
    }
}

open class ImageLabelLabelButtonComponentStyle: ImageLabelLabelComponentStyle {
    
    let s4: ButtonStyle
    
    public init(s1: ImageStyle,
                s2: LabelStyle,
                s3: LabelStyle,
                s4: ButtonStyle) {
        self.s4 = s4
        super.init(s1: s1,
                   s2: s2,
                   s3: s3)
    }
}


```

// let's suppose we have ImageLabelLabelButtonComponent but different styles, your style model inheritance shoud be like |

```swift

open class ImageLabelLabelButtonComponentStyle1: ImageLabelLabelButtonComponentStyle { }
open class ImageLabelLabelButtonComponentStyle2: ImageLabelLabelButtonComponentStyle { }
open class ImageLabelLabelButtonComponentStyle3: ImageLabelLabelButtonComponentStyle { }
open class ImageLabelLabelButtonComponentStyle4: ImageLabelLabelButtonComponentStyle { }

```

// you wil add all your components to the tableview-collection datasource just one time... 

```swift

public class View<Style:ViewStyle>: ViewController, Configurable {

    lazy var dataSource = DataSource.shared

    @discardableResult
    public func configure (state: ViewState<Style>) -> Self {
       switch state {
         case .loaded (let style): 
          dataSource
               .set(tableView)
               .set(style.sectionsStyle)
          indicator.loaded()
         case .loading (let style):
            indicator.loading()
            configure(state: .loaded(style))
         case _:
           break
       }
       return self
    } 
}

extension DataSource {
   // register all your components here with via key-value (model-view) principle just one time...
   static var shared: DataSource {
     .init(components: [
         Descriptor<ImageComponentStyle1, ImageComponent1>(),
         Descriptor<ImageComponentStyle2, ImageComponent2>(),
         Descriptor<ImageComponentStyle3, ImageComponent3>(),
         Descriptor<LabelComponentStyle1, LabelComponent1>(),
         Descriptor<LabelComponentStyle2, LabelComponent2>(),
         Descriptor<LabelComponentStyle3, LabelComponent3>(),
         Descriptor<ButtonComponentStyle1, ButtonComponent1>(),
         Descriptor<ButtonComponentStyle2, ButtonComponent2>(),
         Descriptor<ButtonComponentStyle2, ButtonComponent3>(),
         Descriptor<ImageLabelComponentStyle1, ImageLabelComponent1>(),
         Descriptor<ImageLabelComponentStyle2, ImageLabelComponent2>(),
         Descriptor<ImageLabelComponentStyle3, ImageLabelComponent3>(),
         Descriptor<ImageButtonComponentStyle1, ImageButtonComponent1>(),
         Descriptor<ImageButtonComponentStyle2, ImageButtonComponent2>(),
         Descriptor<ImageButtonComponentStyle3, ImageButtonComponent3>(),
         Descriptor<ImageButtonComponentStyle4, ImageButtonComponent4>(),
      ])
   }
}

```


