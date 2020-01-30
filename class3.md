# iOS Development: Table-based Apps

1. [ Introduction ](#1)
2. [ Getting Started ](#2)
3. [ Table Views ](#3)
4. [ Protocols ](#4)
5. [ The Delegation Pattern ](#5)
6. [ Adding Data to Our Table ](#6)
7. [ Lifecycle Methods ](#7)
8. [ Model View Controller in Cocoa ](#8)
9. [ Multiple Screens ](#9)
10. [ Passing Data to Another View Controller ](#10)
11. [ Building out the RecipeDetailViewController ](#11)
12. [ Assignment ](#12)

<a name="1"></a>
## 1) Introduction

In order to explore building table-based apps, we're building a recipe app called CookIt.

<a name="2"></a>
## 2) Getting Started

Open XCode and create a new Single View App project called "CookIt".

We'll be adding another view later.

<a name="3"></a>
## 3) Table Views

Let's add a table view to our view controller in interface builder. Use the object library to search for table view. Drag one onto your view controller and change the size to be the same as the view.

![](https://i.imgur.com/bT1jsem.png)

In the object library, search for table view cell. Drag one onto your table view.

![](https://i.imgur.com/XB8YUdr.png)

In the Document Outline panel, select your table view cell. In the Attributes Inspector, add a reuse identifier: `recipeCell`. This is important for identifying our cell later when we add data to our table.

![](https://i.imgur.com/ZepLXTO.png)

If you run our app, you'll see the default table view.

<img src="https://i.imgur.com/4LDwmyz.png" width="250" />

Let's connect our tableView to our ViewController. Open the assistant editor, and control-drag your table view to create an outlet called `tableView`. You don't need to do this for the prototype cell.

<a name="4"></a>
## 4) Protocols

Swift has single inheritance, so any names that you see following the first class are protocols.

```swift
class SomeClass: Superclass, Protocol1, Protocol2, ... {
  ...
}
```

A protocol is like a contract that says what you need to do to conform to that protocol (or play by that protocol's rules).

```swift
protocol CanFireMissiles {
  var target: Location { get set }
  var missileType: MissileType { get set }
  
  func fire()
}

// PartyCannon conforms to the CanFireMissiles protocol
struct PartyCannon: CanFireMissiles {
  var target: Location = Location(lat: 123.123, long: 456.456)
  var missileType: MissileType = MissileType.partyExplosion
  
  func fire() {
    MissileLauncher.fireMissiles(location: location, type: missileType)
  }
  
  ...
}
```

<a name="5"></a>
## 5) The Delegation Pattern

Delegation is a design pattern that enables a class or struct to delegate some of its responsibilities to an instance of another type. This pattern is implemented by defining a protocol that expresses the responsibilities of being a delegate, such that a conforming type is guaranteed to provide the functionality that has been delegated.

When our view controller sets itself to the tableView's delegate, it's saying that it will conform to the rules denoted by the UITableViewDelegate protocol.

The UITableView is a closed source class created by apple. It provides a delegate property, that is optional in that it can be nil. When it's set `tableView.delegate = self` where self is our view controller, our view controller is saying that it will recieve whatever tableView data is available to the delegate. When the delegate is nil, the data goes nowhere.

<a name="6"></a>
## 6) Adding Data to Our Table

Let's set our view controller up to conform to the UITableViewDelegate and UITableViewDataSource protocols, and set our view controller as the delegate and dataSource for each.

Paste the following code to your ViewController.swift

```swift
// We've added the UITableViewDelegate and UITableViewDataSource prototypes
// This means our ViewController class needs to conform to these prototypes
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {

  @IBOutlet weak var tableView: UITableView!
  
  // Dummy data
  let recipes = ["Best Brownies", "Banana Bread", "Chocolate Chip Cookies"]

  override func viewDidLoad() {
    super.viewDidLoad()

    // We are telling the tableView that our ViewController will act as its delegate and dataSource
    tableView.delegate = self
    tableView.dataSource = self
  }
  
  // required method to conform to UITableViewDataSource
  func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    // We are setting the number of
    return recipes.count
  }
  
  // required method to conform to UITableViewDataSource
  func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    // We are getting the cell from the queue of cells
    // We recycle cells for better performance
    let cell = tableView.dequeueReusableCell(withIdentifier: "recipeCell", for: indexPath)
    
    // We set the cell's text label to be our recipe string for that index
    cell.textLabel?.text = recipes[indexPath.row]
    
    return cell
  }

}
```

Run your app to see this in action!

<img src="https://i.imgur.com/a875xfN.png" width="250" />


<a name="7"></a>
## 7) Lifecycle Methods

You might've noticed that we set the delegate and dataSource from inside the `viewDidLoad` method.

`viewDidLoad()` is a view controller life cycle method. It's called once, right after view controller is created and the view has loaded into memory, but before the screen has rendered. It's great for instantiating instance variables or building out subviews that need to exist for the entire lifecycle of the view controller.

Another useful lifecycle method is `viewDidAppear()`. This method is called after the view has become visible and has loaded into the view hierarchy.

There are a number of view controller life cycle methods. You can read more about them in the [Apple Documentation for UIViewController](https://developer.apple.com/documentation/uikit/uiviewcontroller)

<a name="8"></a>
## 8) Model View Controller in Cocoa

Cocoa applies a standard MVC pattern, where controllers act as intermediaries between the view and the model.

![](https://i.imgur.com/oowzGp6.png)

Image from the [Apple documentation on MVC](https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/MVC.html)

We've had some exposure to view controllers and views. Let's look more closely at models.

### Adding a Model

Let's add a Recipe model. Our recipe has 2 properties, a title, and an array of steps.

Create a new Swift file and call it Recipe.swift.

![](https://i.imgur.com/sOJ8Mze.png)

![](https://i.imgur.com/lY1uvn7.png)

Add the following code to your Recipe.swift file.

```swift
class Recipe {
  var title: String
  var steps: [String]
  
  init(title: String, steps: [String]) {
    self.title = title
    self.steps = steps
  }
}
```

Now that we have a Recipe model. Let's add some recipes to our view controller and have our table consume the data.

in our ViewController.swift file, change our recipes property to the following:

```swift
let recipes = [
  Recipe(title: "Best Brownies", steps: ["step1", "step2"]),
  Recipe(title: "Banana Bread", steps: ["step1", "step2"]),
  Recipe(title: "Chocolate Chip Cookies", steps: ["step1", "step2"])
]
```

### Exercise

In the `tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)` method, change the cell text label to show each Recipe object's title. (5 mins)

<a name="9"></a>
## 9) Multiple Screens

Now that we have a table that shows a list of our recipes, let's add another view controller. This one will manage the view for each individual recipe - we call this a detail view.

Navigate to interface builder. In the object library, search for View Controller. Drag one onto your story board.

![](https://i.imgur.com/MmhyvNf.png)

Let's add a new code file called RecipeDetailViewController.swift.

Create a new file.
<img src="https://i.imgur.com/hRBYQEp.png" width="250" />

Select Cocoa Touch Class as the source.
![](https://i.imgur.com/VSflVjR.png)

Name your file RecipeDetailViewController, which is a subclass of UIViewController.

![](https://i.imgur.com/yFsfIHl.png)


### The Identity Inspector

Head back over to interface builder. In the Document Outline panel select our new view controller. In the right-hand panel, select the Identity Inspector. In the class field, enter RecipeDetailViewController.

![](https://i.imgur.com/Vs10WYv.png)

### Segues

Segues are objects that manage the transitions between screens. We can add a segue from interface builder by selecting prototype cell, holding control and dragging to our destination view controller (RecipeDetailViewController).

![](https://i.imgur.com/OnCnUhT.jpg)

There are a few different types of segues. We're using a show segue, which just slides the destination view on top of the previous view.

![](https://i.imgur.com/tapzpXj.png)

Once your segue is created, you will see an arrow between your view controllers in interface builder.

![](https://i.imgur.com/878wDlk.png)

### Navigation Controller

A useful practice is to embed related views inside a navigation controller. This provides us with some useful features for navigating between our views.

You can do this by selecting the root view controller (the controller where our app starts), selecting Editor > Embed In > Navigation Controller from the top level menu.

![](https://i.imgur.com/PtEjwec.jpg)

After adding the navigation controller, you can see a menu bar along the top of our screens. This provides us with easy access to our navigation stack. Let's run our app and try it out.

![](https://i.imgur.com/en1tQzG.png)

<a name="10"></a>
## 10) Passing Data to Another View Controller

For our RecipeDetailViewController to do anything useful, it will need to have access to a recipe object.

Let's add that to our code.

```swift
class RecipeDetailViewController: UIViewController {
  
  var recipe: Recipe?

  ...

}
```

When someone taps on one of our recipe cells, and the app prepares to segue to the detail screen, we need to set the recipe object on our RecipeDetailViewController.

Add the following method to your ViewController.swift file.

```swift
override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
  if let recipeDetailViewController = segue.destination as? RecipeDetailViewController,
     let index = tableView.indexPathForSelectedRow?.row {
    recipeDetailViewController.recipe = recipes[index]
  }
}
```

Let's set the recipe's title on its detail view. Change the viewDidLoad method in your RecipeDetailViewController to the following:

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    self.title = recipe?.title
  }
```

<img src="https://i.imgur.com/aViQjhz.png" width="250" />

<a name="11"></a>
## 11) Building out the RecipeDetailViewController

Let's add some more interesting information to our recipes.

Change our Recipe model to have an imageURL property:

```swift
class Recipe {
  var title: String
  var steps: [String]
  var imageURL: String
  
  init(title: String, steps: [String], imageURL: String) {
    self.title = title
    self.steps = steps
    self.imageURL = imageURL
  }
}
```

Change our recipe data in ViewController.swift to the following:

```swift
let recipes = [
  Recipe(title: "Best Brownies", steps: ["step1", "step2"], imageURL: "https://images.pexels.com/photos/45202/brownie-dessert-cake-sweet-45202.jpeg"),
  Recipe(title: "Banana Bread", steps: ["step1", "step2"], imageURL: "https://images.pexels.com/photos/830894/pexels-photo-830894.jpeg"),
  Recipe(title: "Chocolate Chip Cookies", steps: ["step1", "step2"], imageURL: "https://images.pexels.com/photos/230325/pexels-photo-230325.jpeg")
]
```

Add the following to `tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)`: 
*Note: you will want to store recipe as a new constant for re-use in assigning both the cell.textLabel?.text and the cell.imageView?.image*

```swift
// get image from url
// note we are retrieving the image synchronously for simplicity, this is not a best practice
if let url = URL(string: recipe.imageURL), let data = try? Data(contentsOf: url) { //make sure your image in this url does exist, otherwise unwrap in a if let check / try-catch
  cell.imageView?.image = UIImage(data: data)
}
```

<img src="https://i.imgur.com/1gEpNWX.png" width="250" />

Let's build out the recipe detail view.

### Exercise

In interface builder, add an image view, steps label and table view with prototype cell to your RecipeDetailViewController. (5 mins)

Your view may look something like this:

<img src="https://i.imgur.com/jfkJGtw.png" width="250" />

### Exercise

Set the recipe image in the code.

Make the RecipeDetailViewController conform to the UITableViewDelegate and UITableViewDataSource protocols. Don't forget to set our view controller as the delegate and dataSource for the tableView in the viewDidLoad method. Set the text label for each cell as the corresponding step. (15 mins)

Your solution will look end up looking something like this:

<img src="https://i.imgur.com/GSJAgyT.png" width="250" />

<a name="12"></a>
## 12) Assignment

Create a Book Tracker app. This app shows a listing of books you've read, and shows a detail view of each book and your thoughts about it.

Your app should have the following:

* A Book model with the following properties
    * Title
    * ISBN (10 or 13)
    * Author
    * Cover Image URL (use Amazon to get cover image urls)
    * Your Rating out of 5
    * Your Notes (as a String Optional)
* A root view controller that manages a listing (table) of your books. You can hard code the book data (for now until we learn more about persistent storage).
* A detail view controller that manages a detail view of each book. The detail view should show the book cover image, title, isbn, author, your rating and your notes.
