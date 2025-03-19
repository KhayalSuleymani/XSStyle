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
            FacebookConnector(), // will connect Facebook SDK
            FirebaseConnector(), // Will connect Firebase SDK
            GoogleConnector(),   // Will Connect Google SDK
        ])
    }
}
```


Style models **Inheritance** hierarchies.
