## Notification Center

> Notification center is not a notification process as it is thought.

> By generating a notification within the application, you can access this notification from anywhere.

> Notification Center is the sending part and the observer is the receiving part.

- Notification Center: If it is a radio station
- Observer: Radio


![mm1](https://user-images.githubusercontent.com/49749125/142889530-6c7bd0bc-a6b0-41c0-835f-a14a20060620.png)


### Creating Notification Name with Extension


```swift
extension Notification.Name{
    static let notificationName = Notification.Name("notificationX")
}
```

> We use the key value pair structure when sending notifications.

> With userInfo, we can send data in the form of key value pairs in the array.

> But I want to show that we can create a class and use an object as follows.

```swift
class Persons{
    var personName : String?
    var personAge : Int?
    
    init(){
        
    }
    
    init(personName : String , personAge : Int){
        self.personName = personName
        self.personAge = personAge
    } 
}

```

### Notification Submission

```swift
   let person = Persons(personName: "Alican", personAge: 26)
   NotificationCenter.default.post(name: .notificationName, object: nil, userInfo: ["message" : "hello" , "todaysDate" : Date(), "data" : person]) 
```
> Data is sent in dictionary format with userInfo.

### Capture and Listen to Notification

```swift
   // We will handle broadcast (comed from Notification Center)
   NotificationCenter.default.addObserver(self, selector: #selector(makeSomething(notification:)), name: .notificationName, object: nil)
```

```swift
    @objc func makeSomething(notification : Notification){
      let comedMessage = notification.userInfo?["message"]
      let comedDate = notification.userInfo?["todaysDate"]
      let comedData = notification.userInfo?["data"] as! Persons
      dataLabel.text = "\(comedData.personName!) \(comedData.personAge!)"
    }
```

## Notifications

> App permission can be asked to the user within the app.

> If the application permission is taken, about the application in the settings
panel is allowed.

> Once permission is obtained, there is no need to obtain permission again.

> AppDelegate didFinishLaunchingWithOptions() method
permission may be obtained.

### Getting permission for the app

```swift
    import UIKit
    import UserNotifications
    
    class ViewController : UIViewController{
    
        var permissionControl : Bool = false
    
        override func viewDidLoad(){
            super.viewDidLoad()
            
            UNUserNotificationCenter.current().requestAuthorization(options: [.alert , .sound , .badge]) { granted, error in
                self.permissionControl = granted
                if granted{
                    print("Permission was confirmed.")
                }else{
                    print("Permission was not confirmed.")
                }
            }
        }
    
   }
```
### Creating Notification Content

```swift
   import UIKit
   import UserNotifications
   
   @IBAction func createNotification(_ sender: UIButton) {
        
     if permissionControl{
         let content = UNMutableNotificationContent()
         content.title = "Title"
         content.subtitle = "SubTitle"
         content.body = "Message"
         content.badge = 1
         content.sound = UNNotificationSound.default
            
         let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 10, repeats: false)
         let notificationRequest = UNNotificationRequest(identifier: "XNotification", content: content, trigger: trigger)
            
         // we use withCompletionHandler to take another action when the notification is clicked.
         UNUserNotificationCenter.current().add(notificationRequest, withCompletionHandler: nil)
        }
    }
    
```
![mm2](https://user-images.githubusercontent.com/49749125/142908404-2c8f35fe-1214-4bea-859b-089dbfaee336.png)

### Timely Triggering of Notification

```swift
   import UIKit
   import UserNotifications
   
   let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 10, repeats: false) // timeInterval : after how many seconds , repeats : would you repeat
    
```
### Creating Request for Notification

```swift
   import UIKit
   import UserNotifications
   
   let notificationRequest = UNNotificationRequest(identifier: "XNotification", content: content, trigger: trigger)
    
```

### Running Notification

```swift
   import UIKit
   import UserNotifications
   
   // we use withCompletionHandler to take another action when the notification is clicked.
   UNUserNotificationCenter.current().add(notificationRequest, withCompletionHandler: nil)
    
```

### Notification Working While the Application is in the Foreground

> It only works in the background with the initial version of the notification encoding.

> The following coding should be done to make it work in the foreground.

```swift
   import UIKit
   import UserNotifications
   
   // This extension using for foreground notifications
   extension ViewController : UNUserNotificationCenterDelegate{
    func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler: @escaping        (UNNotificationPresentationOptions) -> Void) {
        completionHandler([.banner , .sound , .badge])
    }
}
    
```
> And below code should be run in viewDidLoad.

```swift
   import UIKit
   import UserNotifications
   
   // This extension using for foreground notifications
   extension ViewController : UNUserNotificationCenterDelegate{
    func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler: @escaping        (UNNotificationPresentationOptions) -> Void) {
        completionHandler([.banner , .sound , .badge])
    }
 }
    
```

> And below code should be run in viewDidLoad.

```swift
   import UIKit
   import UserNotifications
   
   override func viewDidLoad() {
      super.viewDidLoad()
        
      UNUserNotificationCenter.current().delegate = self // --> This code added because necessary for our extensions

      UNUserNotificationCenter.current().requestAuthorization(options: [.alert , .sound , .badge]) { granted, error in
          self.permissionControl = granted
          if granted{
             print("Permission was confirmed.")
          }else{
             print("Permission was not confirmed.")
          }
      }
  }
    
```

### Notification Click and Badge Reset

```swift
   import UIKit
   import UserNotifications
   
   // This extension using for foreground notifications
extension ViewController : UNUserNotificationCenterDelegate{
    func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
        completionHandler([.banner , .sound , .badge])
    }
    
    // This method using for Badge Reset When Notification Clicked.
    func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
        let app = UIApplication.shared
        
        if app.applicationState == .active{
            print("clicked when on foregorund.")
            app.applicationIconBadgeNumber = 0
        }
        
        if app.applicationState == .inactive{
            print("clicked when on background.")
            app.applicationIconBadgeNumber = 0
        }
        
        completionHandler()
    }
  }  
}
    
```
### Repeated Notification

> Repeated statements must be a minimum of 60 seconds.

#### Temporal Repetitions (with Seconds Type)

> It generates the first notification 60 seconds after it is run and at 60 second intervals. Starts to repeat.

```swift
   import UIKit
   import UserNotifications
   
   let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 60, repeats: false)
   
```
