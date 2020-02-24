# iOS Development: Building a Multi-View App with Cloud Storage

1. [ Introduction ](#1)
2. [ Getting Started ](#2)
3. [ CocoaPods ](#3)
4. [ Cloud Storage with Firebase ](#4)
5. [ The Chatee Interface ](#5)
6. [ Auto Layout ](#6)
7. [ Multiple Screens ](#7)
8. [ Authenticating Users with Firebase ](#8)
9. [ Performing a Segue Programmatically ](#9)
10. [ Setting up the ChatViewController ](#10)
11. [ Animating the Keyboard ](#11)
12. [ Gesture Recognizers ](#12)
13. [ Assignment ](#13)

<a name="1"></a>
## 1) Introduction

During this lesson we'll be building a realtime chat app called Chatee. We're going to use Firebase for user authentication and as a realtime cloud database for our app's storage. We'll also explore using the Cocoapods dependency manager, and using auto layout to position elements so they look great on different devices.

<a name="2"></a>
## 2) Getting Started

Create a new XCode Single View project (it will become multi view later). Name your project Chatee.

<a name="3"></a>
## 3) CocoaPods

Follow the install instructions at [cocoapods.ord](https://cocoapods.org/)

Open terminal and execute the following:

```bash
sudo gem install cocoapods
```

To setup your cocoapods environment:

```bash
pod setup
```

While that runs let's head over to Firebase and set up our project.

<a name="4"></a>
## 4) Cloud Storage with Firebase

We're going to use a realtime cloud database in order to store our user authentication data and our messages.

Head over to [firebase.google.com](https://firebase.google.com/). Log in with any google email address.

Create a new Firebase project called Chatee. Change the country to Canada.

![](https://i.imgur.com/2trVI8a.png)

From the Chatee Firebase dashboard, select the iOS icon to set up the project for iOS.

![](https://i.imgur.com/reAYLNO.png)

For the bundle identifier, use the same one as in your Chatee XCode project.

![](https://i.imgur.com/aBAnRf5.png)

Download the `GoogleService-info.plist` file and add it to your XCode project.

![](https://i.imgur.com/Cm3Rxkw.png)

![](https://i.imgur.com/HHAmKtE.png)

### Using CocoaPods

Continuing with the Firebase project setup steps, now we need to add the Firebase pods to our project in order to have access to the code we need to connect to Firebase.

#### Creating a Podfile

Open terminal and navigate to your Chatee XCode project directory root.

```bash
pod init
```

This command creates an empty Podfile. Open your Podfile in XCode.

```bash
open Podfile -a XCode
```

#### Adding Pods

Let's add some pods to our Podfile.

```
# Uncomment the next line to define a global platform for your project
platform :ios, '9.0'

target 'Chatee' do
  # Comment the next line if you're not using Swift and don't want to use dynamic frameworks
  use_frameworks!

  # Pods for Chatee
  pod 'Firebase'
  pod 'Firebase/Auth'
  pod 'Firebase/Database'
end
```

Go back to your terminal, in your project root.

```bash
pod install
```

Use the Chatee.xcworkspace app to open your project from now on.

```bash
open Chatee.xcworkspace -a XCode
```

### Configuration in AppDelegate

Add the following line to your AppDelegate.swift file in the `application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?)` method.

```swift
FirebaseApp.configure()
```

It will end up looking like this:

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    // Override point for customization after application launch.
  FirebaseApp.configure()
  return true
}
```

You will also need to `import Firebase` at the top of the file.

**Important!**
Complete the Firebase project setup by running your app to test the connection to your newly created Firebase project.

### Enable Authentication in Firebase

![](https://i.imgur.com/UZlkmfM.png)

### Create the Realtime Database
![](https://i.imgur.com/oKJdO2i.png)


Be sure to start in test mode:
![](https://i.imgur.com/nlTpytg.png)

### Test your Database Connection
Test your database by overwritting your `ViewController.swift` viewDidLoad function:
```swift
override func viewDidLoad() {
    super.viewDidLoad()
    // Do any additional setup after loading the view.
    var ref: DatabaseReference!
    ref = Database.database().reference()
    ref.setValue("Test")
  }
```
Run your application in a simulator and check your database as the value Test should have been written to the main node of your Realtime Database.

<a name="5"></a>
## 5) The Chatee Interface

Our Chatee app is going to need 4 screens; a welcome screen, a login screen, a registration screen, and the screen where a user can send and receive messages.

Let's head over to interface builder and add a welcome screen to get started.

Our welcome screen is going to need two buttons; "Login" and "Register".

![](https://i.imgur.com/xEfO6pB.png)

Now that we have our buttons, we want them to actually do something. In order to set this up, we should create a view controller class that will be specific to this view controller in interface builder.

Let's call our new view controller `WelcomeViewController`.

Create a file called WelcomeViewController.swift (sublcass of UIViewController).

Don't forget to set your view controller's class to WelcomeViewController in the identity inspector. You can delete the default ViewController.swift class as we won't be using it.

<a name="6"></a>
## 6) Auto Layout

Here are our buttons on the iPhone SE sized screen.

<img src="https://i.imgur.com/nWnHs5j.png" width="250" />

Recall from our Slot Master app that when we viewed our app in any screen size different from iPhone 8, it appeared skewed.

We're going to apply auto layout constraints to our view elements in order to make them look great on any screen size and orientation.

There are 2 different approaches when applying auto layout:

### Pinning

![](https://i.imgur.com/8iUWE8W.png)

<img src="https://i.imgur.com/voaYxUb.png" width="300" />


#### Pinning with Explicit Width and Height Constraints

![](https://i.imgur.com/YPMKNFA.png)

<img src="https://i.imgur.com/ORvlcMi.png" width="300" />


### Alignment

Our element will have width and height constraints, as well as alignment constraints (centered vertically and horizontally for instance)

![](https://i.imgur.com/Z0gM77n.png)

![](https://i.imgur.com/PuXCvgy.png)

<img src="https://i.imgur.com/A33iFVK.png" width="300" />


### Adding Constraints to our Buttons

We start by adding a new view, and dragging our buttons inside it.

<img src="https://i.imgur.com/NRGIr8j.png" width="250" />

We are going to add width and height constraints, and vertical and horizontal centering constraints to our container.

Our Log In button will be pinned to the top, left and right of the container. Register is pinned to the bottom, left and right of the container. We also add a constraint to give the buttons equal heights. We do this by selecting one button, holding control and dragging to the other button.

![](https://i.imgur.com/oq3aBpS.png)

To make our buttons look nicer we can add some content insets via the Size Inspector to one of our buttons. The other one will end up looking the same because we have "same heights" constraint.

![](https://i.imgur.com/2dRTk7Q.png)

You can view the constraints for any element in the Size Inspector.

![](https://i.imgur.com/4poZq27.png)

<a name="7"></a>
## 7) Multiple Screens

When a user clicks one of our buttons, we want to show the correct screen. Let's start by adding a new View Controller to our storyboard by using the Object Library.

![](https://i.imgur.com/AW9upwe.png)

### Exercise

Add a new code file called LoginViewController. Assign this class to our new view controller. Check out the notes on adding WelcomeViewController if you're stuck. (5 mins)

### Exercise

In your LoginViewController on your storyboard, add 2 text fields and a log in button. Use auto layout constraints to center these. (10 mins)

Your solution might look something like this:

<img src="https://i.imgur.com/7ZcvyyH.png" width="250" />

### Segues

Add a show segue from your log in button to your the LoginViewController.

![](https://i.imgur.com/L1KPZCm.png)

![](https://i.imgur.com/N0scsps.png)

![](https://i.imgur.com/NlL9clg.png)

### Navigation Controller

Recall from last lesson that we can embed our root view controller in a navigation controller in order to give us some nice navigation features out of the the box. Go ahead and embed our WelcomeViewController in a NavigationController.

![](https://i.imgur.com/PiCsXrP.png)

![](https://i.imgur.com/aJ2Lwiq.png)

### Exercise

Add a `RegistrationViewController` and use a segue to transition to our registration view when the "Register" button is clicked from our welcome screen. You will need to add the view controller in interface builder and add the code file. Your registration screen should look very similar to the login screen. It should have 2 text fields (email and password) and a "Register" button. (10 - 15 mins)

Your solution might look something like this:

![](https://i.imgur.com/LEl3NLm.png)

<a name="8"></a>
## 8) Authenticating Users with Firebase

Let's actually start authenticating users using Firebase and storing them in our cloud db.

### Exercise
Open the assistant editor and navigate to the `RegistrationViewController`. Add outlets for our email and password text fields, and an action for our Register button called `registerButtonTapped`. (5 mins)

In our `registerButtonTapped` action, let's use the Firebase Auth methods to create a new user.

```swift
import UIKit
import Firebase

class RegistrationViewController: UIViewController {

  @IBOutlet weak var emailTextField: UITextField!
  @IBOutlet weak var passwordTextField: UITextField!
  
  @IBAction func registerButtonTapped(_ sender: UIButton) {
    if let email = emailTextField.text, let password = passwordTextField.text {
      Auth.auth().createUser(withEmail: email, password: password) { (user, error) in
        if let err = error {
          // there was an error, print it
          print(err)
        } else {
          // successfully created user
          print("successfully created user " + email)
        }
      }
    }
  }
  
  ...
```

If we take a look at the `createUser` method declaration, it looks like this:

```swift
createUser(withEmail: String, password: String, completion: AuthDataResultCallback?) {
  ...
}
```

Note that the third parameter is called "completion"

### Closures

Closures are blocks of functionality (anaonymous functions) that can be passed around and used in your code. they store references to any constants and variables from the context in which they are defined. Closures are often called lambdas in other programming languages.

Closures take the following form:

```swift
{ (parameters) -> ReturnType in
  statements
}
```

Let's look at the `createUser` function call again. When a function in Swift passes a function as it's last parameter, we can change the call to the following instead of placing the function call inside the `completion` parameter.

```swift
Auth.auth().createUser(withEmail: email, password: password) { (user, error) in
  ...
}
```

Let's try running our app and registering a user.

![](https://i.imgur.com/HoIwB3O.png)

Note that our password field is showing the password instead of hiding it. You can change this by selecting the password text field element in interface builder, going to the Attributes Inspector and checking the "Secure Text Entry" checkbox. Do this for the login password field as well.

![](https://i.imgur.com/bcWjYLV.png)

The console tells us we were successful!

![](https://i.imgur.com/YFfxahq.png)


In Firebase under Authentication we can see our new user!

![](https://i.imgur.com/0ywo4Xm.png)


Ideally, once a user has successfully registered or logged in, we want to send them to an actual messaging screen so they can chat.

<a name="9"></a>
## 9) Performing a Segue Programmatically

Let's add a ChatViewController and segue from the RegistrationViewController.

![](https://i.imgur.com/NcERm8K.png)

We want to trigger this segue when a user has successfully registered or logged in. In order to access our segue in code, we need give it an identifier: `showChat`.

![](https://i.imgur.com/MVfTDRn.png)

Add the following line in your RegistrationViewController after a user has been successfully registered.

```swift
self.performSegue(withIdentifier: "showChat", sender: self)
```

### Exercise

Complete the LoginViewController.

* Add outlets for the text fields
* Add an action that is triggered when the Log In button is tapped
* Add a segue to the ChatViewController
* Give your segue an identifier
* Use `Auth.auth().signIn(withEmail: email, password: password: completion: ...)` Firebase method to sign the user in
* Trigger your segue programmatically if login is successful

(20 mins)

<a name="10"></a>
## 10) Setting up the ChatViewController

### Exercise

Add a table view (and table view cell) - this is for our chat messages, and a view with a text field (to send a message). (5 mins)

Your chat scene in InterfaceBuilder will look something like this:

<img src="https://i.imgur.com/S3WH4KM.png" width="250" />

### Adding a Custom Cell View

Let's add some new files. Create a new file called MessageCell. Be sure to check the "Also create XIB file" options.

![](https://i.imgur.com/DajDyxc.png)


Our cell will need to be flexible for different sizes of text.

#### Organizing our Project

Now that our project is becoming more complex, we should get organized. Click on the Chatee folder and add 3 new "groups": "Controller", "Model" and "View".

Move your files into their designated categories.

<img src="https://i.imgur.com/coiAUJm.png" width="250" />

### Back to our Custom Cell

Open the MessageCell.xib file as source code.

![](https://i.imgur.com/225DHoL.png)

So we can get the cell up and running, paste the following xml into the file.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<document type="com.apple.InterfaceBuilder3.CocoaTouch.XIB" version="3.0" toolsVersion="14313.18" targetRuntime="iOS.CocoaTouch" propertyAccessControl="none" useAutolayout="YES" useTraitCollections="YES" useSafeAreas="YES" colorMatched="YES">
    <device id="retina4_7" orientation="portrait">
        <adaptation id="fullscreen"/>
    </device>
    <dependencies>
        <plugIn identifier="com.apple.InterfaceBuilder.IBCocoaTouchPlugin" version="14283.14"/>
        <capability name="Safe area layout guides" minToolsVersion="9.0"/>
        <capability name="documents saved in the Xcode 8 format" minToolsVersion="8.0"/>
    </dependencies>
    <objects>
        <placeholder placeholderIdentifier="IBFilesOwner" id="-1" userLabel="File's Owner"/>
        <placeholder placeholderIdentifier="IBFirstResponder" id="-2" customClass="UIResponder"/>
        <tableViewCell contentMode="scaleToFill" selectionStyle="default" indentationWidth="10" reuseIdentifier="messageCell" rowHeight="105" id="KGk-i7-Jjw" customClass="MessageCell" customModule="Chatee" customModuleProvider="target">
            <rect key="frame" x="0.0" y="0.0" width="306" height="105"/>
            <autoresizingMask key="autoresizingMask" flexibleMaxX="YES" flexibleMaxY="YES"/>
            <tableViewCellContentView key="contentView" opaque="NO" clipsSubviews="YES" multipleTouchEnabled="YES" contentMode="center" tableViewCell="KGk-i7-Jjw" id="H2p-sc-9uM">
                <rect key="frame" x="0.0" y="0.0" width="306" height="104.5"/>
                <autoresizingMask key="autoresizingMask"/>
                <subviews>
                    <imageView userInteractionEnabled="NO" contentMode="scaleToFill" horizontalHuggingPriority="251" verticalHuggingPriority="251" translatesAutoresizingMaskIntoConstraints="NO" id="iav-5L-f2t">
                        <rect key="frame" x="15" y="10" width="60" height="60"/>
                        <constraints>
                            <constraint firstAttribute="height" constant="60" id="SvK-e5-AqV"/>
                            <constraint firstAttribute="width" constant="60" id="anT-XA-5Pe"/>
                        </constraints>
                    </imageView>
                    <view contentMode="scaleToFill" translatesAutoresizingMaskIntoConstraints="NO" id="Yxy-xX-iOb">
                        <rect key="frame" x="90" y="10" width="201" height="84.5"/>
                        <subviews>
                            <label opaque="NO" userInteractionEnabled="NO" contentMode="left" horizontalHuggingPriority="251" verticalHuggingPriority="252" verticalCompressionResistancePriority="751" text="Sender" textAlignment="natural" lineBreakMode="tailTruncation" baselineAdjustment="alignBaselines" adjustsFontSizeToFit="NO" translatesAutoresizingMaskIntoConstraints="NO" id="VZ7-Wf-KOM">
                                <rect key="frame" x="10" y="5" width="181" height="13.5"/>
                                <fontDescription key="fontDescription" type="system" pointSize="11"/>
                                <nil key="textColor"/>
                                <nil key="highlightedColor"/>
                            </label>
                            <label opaque="NO" userInteractionEnabled="NO" contentMode="left" horizontalHuggingPriority="251" verticalHuggingPriority="251" text="Label" textAlignment="natural" lineBreakMode="tailTruncation" numberOfLines="0" baselineAdjustment="alignBaselines" adjustsFontSizeToFit="NO" translatesAutoresizingMaskIntoConstraints="NO" id="k7w-Hz-nV0">
                                <rect key="frame" x="10" y="20.5" width="181" height="54"/>
                                <fontDescription key="fontDescription" type="system" pointSize="17"/>
                                <nil key="textColor"/>
                                <nil key="highlightedColor"/>
                            </label>
                        </subviews>
                        <color key="backgroundColor" red="0.78580353989999996" green="0.55147496250000005" blue="0.92153996230000002" alpha="1" colorSpace="custom" customColorSpace="displayP3"/>
                        <constraints>
                            <constraint firstItem="k7w-Hz-nV0" firstAttribute="top" secondItem="VZ7-Wf-KOM" secondAttribute="bottom" constant="2" id="GJg-y9-yQH"/>
                            <constraint firstAttribute="bottom" secondItem="k7w-Hz-nV0" secondAttribute="bottom" constant="10" id="LSD-pC-0kz"/>
                            <constraint firstItem="k7w-Hz-nV0" firstAttribute="leading" secondItem="Yxy-xX-iOb" secondAttribute="leading" constant="10" id="U4b-sr-bU9"/>
                            <constraint firstAttribute="trailing" secondItem="k7w-Hz-nV0" secondAttribute="trailing" constant="10" id="Y1Y-NI-2T3"/>
                            <constraint firstItem="VZ7-Wf-KOM" firstAttribute="leading" secondItem="Yxy-xX-iOb" secondAttribute="leading" constant="10" id="g6a-v8-QvQ"/>
                            <constraint firstAttribute="trailing" secondItem="VZ7-Wf-KOM" secondAttribute="trailing" constant="10" id="yw2-eW-xvt"/>
                            <constraint firstItem="VZ7-Wf-KOM" firstAttribute="top" secondItem="Yxy-xX-iOb" secondAttribute="top" constant="5" id="zUw-6F-OIT"/>
                        </constraints>
                        <userDefinedRuntimeAttributes>
                            <userDefinedRuntimeAttribute type="number" keyPath="layer.cornerRadius">
                                <integer key="value" value="15"/>
                            </userDefinedRuntimeAttribute>
                            <userDefinedRuntimeAttribute type="boolean" keyPath="layer.masksToBounds" value="YES"/>
                        </userDefinedRuntimeAttributes>
                    </view>
                </subviews>
                <constraints>
                    <constraint firstItem="Yxy-xX-iOb" firstAttribute="leading" secondItem="iav-5L-f2t" secondAttribute="trailing" constant="15" id="AIy-oM-HRQ"/>
                    <constraint firstItem="iav-5L-f2t" firstAttribute="top" secondItem="H2p-sc-9uM" secondAttribute="top" constant="10" id="AxN-kB-0Fn"/>
                    <constraint firstItem="iav-5L-f2t" firstAttribute="leading" secondItem="H2p-sc-9uM" secondAttribute="leading" constant="15" id="Rmc-iy-9eL"/>
                    <constraint firstAttribute="trailing" secondItem="Yxy-xX-iOb" secondAttribute="trailing" constant="15" id="aJo-1U-c6L"/>
                    <constraint firstAttribute="bottom" secondItem="Yxy-xX-iOb" secondAttribute="bottom" constant="10" id="hDJ-V6-HUW"/>
                    <constraint firstItem="Yxy-xX-iOb" firstAttribute="top" secondItem="H2p-sc-9uM" secondAttribute="top" constant="10" id="xhV-0z-sbu"/>
                </constraints>
            </tableViewCellContentView>
            <viewLayoutGuide key="safeArea" id="njF-e1-oar"/>
            <connections>
                <outlet property="messageBodyBackground" destination="Yxy-xX-iOb" id="arn-43-SIH"/>
                <outlet property="messageBodyLabel" destination="k7w-Hz-nV0" id="U5Q-Si-91i"/>
                <outlet property="senderImageView" destination="iav-5L-f2t" id="Ti2-j3-Eii"/>
                <outlet property="senderLabel" destination="VZ7-Wf-KOM" id="DWU-SW-KuD"/>
            </connections>
            <point key="canvasLocation" x="41.600000000000001" y="75.112443778110944"/>
        </tableViewCell>
    </objects>
</document>

```

When you open your MessageCell.xib file in Interface Builder, you should see something like this:

<img src="https://i.imgur.com/0VY1v29.png" width="250" />

Open your MessageCell.swift file in the assistant editor, and create outlets for the following elements: the senderImageView, senderLabel, messageBodyImageView, messageBodyBackground.

```swift
class MessageCell: UITableViewCell {

  @IBOutlet weak var messageBodyBackground: UIView!
  @IBOutlet weak var senderImageView: UIImageView!
  @IBOutlet weak var messageBodyLabel: UILabel!
  @IBOutlet weak var senderLabel: UILabel!

  override func awakeFromNib() {
        super.awakeFromNib()
        // Initialization code
    }

    override func setSelected(_ selected: Bool, animated: Bool) {
        super.setSelected(selected, animated: animated)

        // Configure the view for the selected state
    }

}
```

### Exercise

Add a Message model with a sender property and a message property, both are strings. (5 mins)

```swift
class Message {
  var sender: String
  var messageBody: String
  
  init(sender: String, messageBody: String) {
    self.sender = sender
    self.messageBody = messageBody
  }
}
```

Go back to Interface Builder and open the Assistant Editor for the ChatViewController. Create outlets for your table view (to display the messages) and text field (to send new messages).
```swift
@IBOutlet weak var tableView: UITableView!
@IBOutlet weak var textField: UITextField!
```

Now add an action method to send messages.
```swift
@IBAction func sendButtonTapped(_ sender: UIButton) {
    if let text = textField.text {
      let messagesDB = Database.database().reference().child("Messages")
      
      let messageDict = [
        "Sender": Auth.auth().currentUser?.email,
        "MessageBody": text
      ]
      
      // set value in firebase db
      messagesDB.childByAutoId().setValue(messageDict) { (error, ref) in
        if let err = error {
          print(err)
        } else {
          print("Message successfully saved")
        }
        
        // clear the textField after sending
        self.textField.text = ""
      }
    }
  }
```
Finally, add the logic to populate the messages in the table using our custom cell.
```swift
/*
    You will need to use import Firebase, and add reference to the UITableViewDelegate, UITableViewDataSource protocols.
*/

  var messages: [Message] = []
  
  override func viewDidLoad() {
    super.viewDidLoad()
    
    // set up this view controller as the tableView's delegate and dataSource
    tableView.delegate = self
    tableView.dataSource = self
    
    // register our custom cell xib
    tableView.register(UINib(nibName: "MessageCell", bundle: nil) , forCellReuseIdentifier: "messageCell")

    getMessages()
  }
  
  override func viewDidAppear(_ animated: Bool) {
    scrollToBottom()
  }
  
  func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    return messages.count
  }
  
  func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: "messageCell", for: indexPath) as! MessageCell
    let message = messages[indexPath.row]

    cell.messageBodyLabel.text = message.messageBody
    cell.senderLabel.text = message.sender
    
    // if the cell is showing our message make it a different colour than other user messages
    if let email = Auth.auth().currentUser?.email, email == message.sender {
      cell.messageBodyBackground.backgroundColor = UIColor.green
      cell.senderImageView.image = UIImage(named: "stars")
    } else {
      cell.messageBodyBackground.backgroundColor = UIColor.orange
      cell.senderImageView.image = UIImage(named: "smile")
    }
    
    return cell
  }
  
  // scrolls the table view to the bottom so we can see the most recent messages
  func scrollToBottom() {
    guard tableView.numberOfRows(inSection: 0) > 0 else { return }

    let indexPath = IndexPath(row: messages.count - 1, section: 0)
    tableView.scrollToRow(at: indexPath, at: .bottom, animated: false)
  }
  
  func getMessages() {
    // get all the messages in our firebase Messages db
    let messagesDB = Database.database().reference().child("Messages")

    // observe for changes in firebase Messages db
    messagesDB.observe(.childAdded) { (snapShot) in
      // we originally stored our data as a dictionary, ie ["Sender": "an email address", "MessageBody": "blah blah"], so we need to cast the value we're getting from firebase back into a dictionary
      if let value = snapShot.value as? Dictionary<String, String>,
         let messageBody = value["MessageBody"],
         let sender = value["Sender"] {
        // create a Message object from the data we get back from firebase
        let message = Message(sender: sender, messageBody: messageBody)
        // add our new messages to our messages array
        self.messages.append(message)
        
        // reload our table
        self.tableView.reloadData()
        
        // ensure our table is scrolled to the bottom where the most recent message is
        self.scrollToBottom()
      }
    }
  }

```

![](https://i.imgur.com/ShSMVw7.png)

Go ahead and run your app on multiple simulators and try it out!

<a name="11"></a>
## 11) Animating the Keyboard

If we leave our app as it is now, the keyboard will obstruct our textfield view and button. We need to move these up when the keyboard opens.

### Exercise

Add the `UITextFieldDelegate` to the `ChatViewController`. Set the `ChatViewController` as the delegate in `viewDidLoad`. (5 mins)

Add an`@IBOutlet weak var heightConstraint: NSLayoutConstraint!` from the height constraint on the text field and send button's container View to the `ChatViewController`. If you did not set a height constraint on the View, add one with a value of 100 points.

![](https://i.imgur.com/QqhoeZH.png)


Add the following methods to the `ChatViewController`

```swift
func textFieldDidBeginEditing(_ textField: UITextField) {
  UIView.animate(withDuration: 0.5) {
    self.heightConstraint.constant = 360
    self.view.layoutIfNeeded()
    self.scrollToBottom()
  }
}

func textFieldDidEndEditing(_ textField: UITextField) {
  UIView.animate(withDuration: 0.5) {
    self.heightConstraint.constant = 100
    self.view.layoutIfNeeded()
    self.scrollToBottom()
  }
}
```

<img src="https://i.imgur.com/38cnw3y.png" width="250" />
Now our keyboard pushes our textfield up so we can still access the Send button. The only problem now is that we can't close the keyboard. To do this, we'll need to use a gesture recognizer.

<a name="12"></a>
## 12) Gesture Recognizers

Gesture Recognizers allow you to watch for gestures on designated views in order to do something in response.

Add the following code to your viewDidLoad method in ChatViewController.swift.

```swift
// gesture recognizers
let tapGesture = UITapGestureRecognizer(target: self, action: #selector (tableViewTapped))
tableView.addGestureRecognizer(tapGesture)
```

Now add the following method at the end of the ChatViewController.

```swift
@objc func tableViewTapped() {
  textField.endEditing(true)
}
```
Note: the `@objc` makes this method available to objective c APIs

<a name="13"></a>
## 13) Assignment

Try running through the notes again on your own and rebuild your own Chatee app. Thoroughly read through and understand all the code in the ChatViewController.


















