# iOS Development: Core Data Apps

1. [ Introduction ](#1)
2. [ Getting Started ](#2)
3. [ Core Data ](#3)
4. [ Entities ](#4)
5. [ Retrieving our Context ](#5)
6. [ Creating Items ](#6)
7. [ Presenting a Controller Programmatically ](#7)
8. [ Reading Items ](#8)
9. [ Updating Items ](#9)
10. [ Deleting Items and SwipeCellKit ](#10)
11. [ Organizing Our Files ](#11)
12. [ Extensions ](#12)
13. [ Search ](#13)
14. [ Categories ](#14)
15. [ Entity Relationships ](#15)
16. [ Loading Associations ](#16)
17. [ Compound Predicates ](#17)
18. [ Assignment ](#18)


<a name="1"></a>
## 1) Introduction

We will be building a todo app called ToDude. It will keep a list of categories and each category will have an associated list of items. We are going to be using Core Data APIs for persisting our Item and Category data. If our app is shut down and becomes inactive, either by us or the operating system, we can retrieve our saved data the next time we open the app.

<a name="2"></a>
## 2) Getting Started

Create a new single view todo app called ToDude.

In order to create a project with Core Data stubs, select "Use Core Data" when creating your project. We can add everything manually, but it's faster to do it this way. 

![](https://i.imgur.com/wNL0b7X.png)

### Subclassing UITableViewController

When we subclass UITableViewController, we get much of delegate and dataSource set up for free. This is an alternative to adding the tableView and cell ourselves now that we know how everything gets wired up.

Remove the default view controller, and replace it with a table view controller. Be sure to set this as the root view controller in the attributes inspector.

<img src="https://i.imgur.com/SATMDWf.png" width="250" />

<img src="https://i.imgur.com/8pzgitV.png" width="250" />

Embed your View Controller in a Navigation Controller.

![](https://i.imgur.com/DHqT75u.png)

Create a new file called ItemListViewController.swift. Be sure to create it as a subclass of UITableViewController.

![](https://i.imgur.com/Cv2pZCL.png)


Set ItemListViewController as the class for your table view controller on the main storyboard. 

![](https://i.imgur.com/YiJNNXE.png)

Give your cell and identifier of "itemCell".

<img src="https://i.imgur.com/qOv0PmQ.png" width="250" />

<a name="3"></a>
## 3) Core Data

Core Data is a framework that can be used in your app to manage the model layer objects, including persisting data locally on your device. 

Take a look at AppDelegate.swift and note that now we have a `persistentContainer` variable that we can access in order to retrieve our Core Data context.

We also now have a `saveContext` method which we will use to save our data.


<a name="4"></a>
## 4) Entities

Let's create our Item entity

Open your `ToDudeCD.xcdatamodeld` file.

Add a new Entity and name it `Item`. Add two attributes, title (String) and completed (Boolean).

![](https://i.imgur.com/T3lY62S.png)

<img src="https://i.imgur.com/eCAMVRI.png" width="300" />

![](https://i.imgur.com/s1LYca8.png)

At the top of ItemListViewController.swift, add the following property:

```swift
// import CoreData
var items = [Item]()
```

If our todo item is "completed" (the completed attribute is true), we want to show a checkmark on the cell. We can do this pretty easily using the cell accessoryType.

After the viewDidLoad method replace the methods that follow 
```swift
// MARK: - Table view data source
```
with the following:
```swift
    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return items.count
    }

  
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
      let cell = tableView.dequeueReusableCell(withIdentifier: "itemCell", for: indexPath)
      
      let item = items[indexPath.row]
      
      cell.textLabel?.text = item.title
      cell.accessoryType = item.completed ? .checkmark : .none
      
      return cell
    }
  
    override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
      let item = items[indexPath.row]
    
      // toggle completed
      item.completed = !item.completed
    }
```


<a name="5"></a>
## 5) Retrieving our Context

At the top of ItemListViewController, add a property called `context`. We get the `persistentContainer` via our AppDelegate, and use this to retrieve our context.

```swift
let context = (UIApplication.shared.delegate as! AppDelegate).persistentContainer.viewContext
```

<a name="6"></a>
## 6) Creating Items

We want our users to be able to create new Items. Let's add a button and action method to enable this.

Open the Object Library and add a Bar Button Item to the Navigation Bar of your Table View Controller.

![](https://i.imgur.com/Uqohskc.png)

Change the System Item attribute to "Add".

![](https://i.imgur.com/lHvZf9e.png)

Open the Assistant Editor, and control drag from your bar button item to create an action called "addItemButtonTapped".

<a name="7"></a>
## 7) Presenting a Controller Programmatically

We are going to show an alert box when a user taps the + button.

```swift
@IBAction func addItemButtonTapped(_ sender: UIBarButtonItem) {
  // we need this in order to access the text field data outside of the 'addTextField' scope below
  var tempTextField = UITextField()
  
  // create a UIAlertController object
  let alertController = UIAlertController(title: "Add New Item", message: "", preferredStyle: .alert)
  
  // create a UIAlertAction object
  let alertAction = UIAlertAction(title: "Done", style: .default) { (action) in
    // create a new item from our Item core data entity (we pass it the context)
    let newItem = Item(context: self.context)
    
    // if the text field text is not nil
    if let text = tempTextField.text {
      // set the item attributes
      newItem.title = text
      newItem.completed = false
      
      // append the item to our items array
      self.items.append(newItem)
      
      // call our saveItems() method which saves our context and reloads the table
      self.saveItems()
    }
  }
  
  alertController.addTextField { (textField) in
    textField.placeholder = "Title"
    tempTextField = textField
  }
  
  // Add the action we created above to our alert controller
  alertController.addAction(alertAction)
  // show our alert on screen
  present(alertController, animated: true, completion: nil)
}

func saveItems() {
  // wrap our try statement below in a do/catch block so we can handle any errors
  do {
    // save our context
    try context.save()
  } catch {
    print("Error saving context \(error)")
  }
  
  // reload our table to reflect any changes
  tableView.reloadData()
}
```

Give your app a try!

![](https://i.imgur.com/Y1oVwIF.png)

If you close and terminate your app, then restart it, what happens?

<a name="8"></a>
## 8) Reading Items

In order to read data using core data we need to create a fetch request.

Add the following method after the `saveData()` method.

```swift
func loadItems() {
  // create a new fetch request of type NSFetchRequest<Item> - you must provide a type
  let fetchRequest: NSFetchRequest<Item> = Item.fetchRequest()
  
  // wrap our try statement below in a do/catch block so we can handle any errors
  do {
    // fetch our items using our fetch request, save them in our items array
    items = try context.fetch(fetchRequest)
  } catch {
    print("Error fetching items: \(error)")
  }
  
  // reload our table to reflect any changes
  tableView.reloadData()
}
```

We should call `loadItems()` from `viewDidLoad()` so that we can retrieve our saved items whenever we open the app from the background.



<a name="9"></a>
## 9) Updating Items

Each time a user clicks an item, we toggle whether the item is completed or not. In order to update our item, we only need to save our context again, in the same way as we do when we create a new item.

Add a call to `saveData()` at the end of our tableView...didSelectRow method (after we've updated the completed attribute).

```swift
override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
  let item = items[indexPath.row]
  item.completed = !item.completed
  saveItems()
}
```
Now that the value is being changed AND saved to the database the checkmark will be present when ever this view is loaded (unless toggled back to completed = false):

![](https://i.imgur.com/9uWpuxS.png)

<a name="10"></a>
## 10) Deleting Items and SwipeCellKit

We are going to add a swipe to delete action to our cell.

Exercise:

Add the [SwipeCellKit](https://cocoapods.org/pods/SwipeCellKit) cocoapod to your project. Reminder: open your terminal and navigate to your project root. Use `pod init` to create a Podfile. Open your Podfile in Xcode using `open Podfile -a Xcode`. Uncomment the second line and add the SwipeCellKit pod. Head back over to terminal and run `pod install`. Don't forget to reopen your project using the newly created workspace. (5 mins)

Now that our pod is installed, add `import SwipeCellKit` at the top of our ItemListViewController.

Have our ItemListViewController conform to the SwipeTableViewCellDelegate protocol.

![](https://i.imgur.com/tLa8W0c.png)

Add a trash can asset. You can find one called trash.png in your assignment repo. Our trash can is very small so we can use one size and drag it into the 2x position.

![](https://i.imgur.com/WL6qdO6.png)

Add the following two lines of code at the top of our tableView...cellForRowAt method:

```swift
let cell = tableView.dequeueReusableCell(withIdentifier: "itemCell", for: indexPath) as! SwipeTableViewCell
cell.delegate = self
```

In order to cast the UITableViewCell to a SwipeTableViewCellupdate custom class update the cell in your Main.storyboard:
<img src="https://i.imgur.com/Ocfy2Fi.png" width="250" />

Add the following methods at the end of our controller.

```swift
func tableView(_ tableView: UITableView, editActionsForRowAt indexPath: IndexPath, for orientation: SwipeActionsOrientation) -> [SwipeAction]? {
  guard orientation == .right else { return nil }
  
  // initialize a SwipeAction object
  let deleteAction = SwipeAction(style: .destructive, title: "Delete") { _, indexPath in
    // delete the item from our context
    self.context.delete(self.items[indexPath.row])
    // remove the item from the items array
    self.items.remove(at: indexPath.row)
    
    // save our context
    self.saveItems()
  }
  
  // customize the action appearance
  deleteAction.image = UIImage(named: "trash")
  
  return [deleteAction]
}
```

Now when you swipe each cell, you should see a red delete button.

![](https://i.imgur.com/OkqA84a.png)


Let's make our app look nicer by making the row height of each cell larger.

Add `tableView.rowHeight = 85.0` as the last line of the `viewDidLoad()` method.

<a name="11"></a>
## 11) Organizing Our Files

As our apps become more complex, we may find our files (controllers especially) becoming large.

You can use MARK comments to organize your files, and navigate to a section of your code quickly.

<img src="https://i.imgur.com/EEXWRWU.png" width="400" />

![](https://i.imgur.com/UpfkFKB.png)


<a name="12"></a>
## 12) Extensions

We can use extensions to break up our larger files into smaller pieces without actually breaking up the class.

Add the following code at the end of the file and remove `UISearchBarDelegate` from the initial declaration. This pattern gives us more flexibility for organizing our code, and still playing nicely with the Cocoa Touch brand of MVC (our controllers become large quickly).

```swift
extension ItemListViewController: SwipeTableViewCellDelegate {
  func tableView(_ tableView: UITableView, editActionsForRowAt indexPath: IndexPath, for orientation: SwipeActionsOrientation) -> [SwipeAction]? {
    guard orientation == .right else { return nil }
    
    // initialize a SwipeAction object
    let deleteAction = SwipeAction(style: .destructive, title: "Delete") { _, indexPath in
      // delete the item from our context
      self.context.delete(self.items[indexPath.row])
      // remove the item from the items array
      self.items.remove(at: indexPath.row)
      
      // save our context
      self.saveItems()
    }
    
    // customize the action appearance
    deleteAction.image = UIImage(named: "trash")
    
    return [deleteAction]
  }
}
```

<a name="13"></a>
## 13) Search

Add a search bar between the navigation bar and the table view using the object library.

<img src="https://i.imgur.com/N8SwSAG.png" width="250" />

Have our ItemListViewController conform to the UISearchBarDelegate protocol by adding it to the class declaration.

Previously we set our controller as the delegate in the code. We can also set our delegate by control dragging from our search bar to the controller icon on the top of the scene.

<img src="https://i.imgur.com/00SVbEe.png" width="300" />

<img src="https://i.imgur.com/kiXlwAd.png" width="300" />

Add the following extension at the end of your ItemListViewController:

```swift
// MARK: Search Bar Methods
extension ItemListViewController: UISearchBarDelegate {

  func searchBarSearchButtonClicked(_ searchBar: UISearchBar) {
    // if our search text is nil we should not execute any more code and just return
    guard let searchText = searchBar.text else { return }
    searchItems(searchText: searchText)
  }

  func searchBar(_ searchBar: UISearchBar, textDidChange searchText: String) {
    if searchText.count > 0 {
      searchItems(searchText: searchText)
    } else if searchText.count == 0 {
      // show the full list of items
      loadItems()
    }
  }

  // this method's use is restricted to this file
  fileprivate func searchItems(searchText: String) {
    // our fetch request for items
    let fetchRequest: NSFetchRequest<Item> = Item.fetchRequest()
    
    // a predicate allows us to create a filter or mapping for our items
    // [c] means ignore case
    let predicate = NSPredicate(format: "title CONTAINS[c] %@", searchText)
    
    // the sort descriptor allows us to tell the request how we want our data sorted
    let sortDescriptor = NSSortDescriptor(key: "title", ascending: true)
    
    // set the predicate and sort descriptors for on the request
    fetchRequest.predicate = predicate
    fetchRequest.sortDescriptors = [sortDescriptor]
    
    // retrieve the items with the request we created
    do {
      items = try context.fetch(fetchRequest)
    } catch {
      print("Error fetching items: \(error)")
    }
    
    // reload our table with our new data
    tableView.reloadData()
  }
}
```

<img src="https://i.imgur.com/gNxozef.png" width="220" /> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;->&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <img src="https://i.imgur.com/InqdQ3d.png" width="220" />

Check out the [NSPredicate Cheat Sheet](https://academy.realm.io/posts/nspredicate-cheatsheet/) created by Realm to see all the different options for filtering data.


<a name="14"></a>
## 14) Categories

Let's make our app more useful by introducing categories. Each category will have an associated set of items.

In the main storyboard, add a CategoryViewController in between the NavigationController and ItemListViewController. You will have to delete the segue between NavigationController and ItemListViewController.

Create a new segue between NavigationController and the CategoryViewController. Set it as a Relationship Segue (root view controller)

![](https://i.imgur.com/4Nzbzlz.png)

Add a show segue between the CategoryViewController and the ItemListViewController. Be sure to set a segue identifier such as "showItems".

![](https://i.imgur.com/FWtZXow.png)

Your storyboard will look something like this:

![](https://i.imgur.com/orItKDI.png)


<a name="15"></a>
## 15) Entity Relationships

Open your data model and add a new Entity called Category.

It should have one attribute called name (a String).

We are going to build a one to many relationship between Category and Item (because a Category can have many Items, and an Item will have one Category). To make this easier to visualise, change the editor style to show the entity relationship diagram

![](https://i.imgur.com/TPmlr4C.png)

Click the relationship area on one entity, press control and drag to the other. Set the name and type for each entity relationship.

![](https://i.imgur.com/4sOqbEQ.png)

![](https://i.imgur.com/2V7Qzx8.png)

### Exercise

Create a CategoryViewController file (a subclass of UITableViewController) and build it out to be very similar to the ItemListViewController. It should enable a user to do the following:
* Add a new category (they can enter the name in an alert)
* Swipe to delete a category

It does not need:
* To show an item list when a cell is selected (yet)
* A search bar

(30 mins)

### Solution (Don't look until you've given it a good shot!):

```swift
import UIKit
import CoreData
import SwipeCellKit

class CategoryViewController: UITableViewController {

  let context = (UIApplication.shared.delegate as! AppDelegate).persistentContainer.viewContext
  
  var categories = [Category]()
  
  override func viewDidLoad() {
    super.viewDidLoad()
    
    loadCategories()
    
    tableView.rowHeight = 85.0
  }
  
  @IBAction func addCategoryButtonTapped(_ sender: UIBarButtonItem) {
    // we need this in order to access the text field data outside of the 'addTextField' scope below
    var tempTextField = UITextField()
    
    // create a UIAlertController object
    let alertController = UIAlertController(title: "Add New Category", message: "", preferredStyle: .alert)
    
    // create a UIAlertAction object
    let alertAction = UIAlertAction(title: "Done", style: .default) { (action) in
      // create a new category from our Category core data entity (we pass it the context)
      let newCategory = Category(context: self.context)
      
      // if the text field text is not nil
      if let text = tempTextField.text {
        // set the category attributes
        newCategory.name = text
        
        // append the category to our categories array
        self.categories.append(newCategory)
        
        // call our saveCategories() method which saves our context and reloads the table
        self.saveCategories()
      }
    }
    
    alertController.addTextField { (textField) in
      textField.placeholder = "Name"
      tempTextField = textField
    }
    
    // Add the action we created above to our alert controller
    alertController.addAction(alertAction)
    // show our alert on screen
    present(alertController, animated: true, completion: nil)
  }
  
  override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: "categoryCell", for: indexPath) as! SwipeTableViewCell
    cell.delegate = self
    
    let category = categories[indexPath.row]
    
    cell.textLabel?.text = category.name

    return cell
  }
  
  override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    return categories.count
  }
  
  func saveCategories() {
    // wrap our try statement below in a do/catch block so we can handle any errors
    do {
      // save our context
      try context.save()
    } catch {
      print("Error saving context \(error)")
    }
    
    // reload our table to reflect any changes
    tableView.reloadData()
  }
  
  func loadCategories() {
    // create a new fetch request of type NSFetchRequest<Category> - you must provide a type
    let fetchRequest: NSFetchRequest<Category> = Category.fetchRequest()
    
    // wrap our try statement below in a do/catch block so we can handle any errors
    do {
      // fetch our categories using our fetch request, save them in our categories array
      categories = try context.fetch(fetchRequest)
    } catch {
      print("Error fetching categories: \(error)")
    }
    
    // reload our table to reflect any changes
    tableView.reloadData()
  }
}

// MARK: SwipeTableViewCellDelegate Methods

extension CategoryViewController: SwipeTableViewCellDelegate {
  
  func tableView(_ tableView: UITableView, editActionsForRowAt indexPath: IndexPath, for orientation: SwipeActionsOrientation) -> [SwipeAction]? {
    guard orientation == .right else { return nil }
    
    // initialize a SwipeAction object
    let deleteAction = SwipeAction(style: .destructive, title: "Delete") { _, indexPath in
      // delete the item from our context
      self.context.delete(self.categories[indexPath.row])
      // remove the item from the items array
      self.categories.remove(at: indexPath.row)
      
      // save our context
      self.saveCategories()
    }
    
    // customize the action appearance
    deleteAction.image = UIImage(named: "trash")
    
    return [deleteAction]
  }
  
}

```

### Connecting the List Items

When a user clicks a category, we would like to show the associated list of items.

In order to do this we will need to tell the ItemListViewController which Category was selected.

Add an optional Category to the ItemListViewController that we can set when we prepare to perform a segue when one of our categories in selected.

![](https://i.imgur.com/sFREsuj.png)

When we perform a segue after a user selects a Category cell, we will need to set this property on the ItemListViewController.

Add the following code to your CategoryViewController.

```swift
override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
  performSegue(withIdentifier: "showItems", sender: self)
}

override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
  // get the destination view controller (the place where the segue will take us)
  let destinationVC = segue.destination as! ItemListViewController
  
  // get the indexPath of the selected cell
  if let indexPath = tableView.indexPathForSelectedRow {
    // set the category propert on the destination view controller (ItemListViewController)
    destinationVC.category = categories[indexPath.row]
  }
}
```

Give the app a try.

![](https://i.imgur.com/TU7Foyh.png)


What list of items do you see when you select a category.

<a name="16"></a>
## 16) Loading Associations

In order to retrieve the items for the selected category, we need to create a predicate (or filter) to retrieve them.

Head back over to the ItemListViewController.

The first thing we need to do, is set the category on every item when it is first created. In the `addItemButtonTapped` action, add the following line after we set the title and continued attributes on the new item.

```swift
newItem.category = self.category
```

Replace the `loadItems()` method with the following:

```swift
func loadItems() {
  // create a new fetch request of type NSFetchRequest<Item> - you must provide a type
  let fetchRequest: NSFetchRequest<Item> = Item.fetchRequest()
  
  // a predicate allows us to create a filter or mapping for our items
  let predicate = NSPredicate(format: "category.name MATCHES %@", category?.name ?? "")
  
  fetchRequest.predicate = predicate

  // wrap our try statement below in a do/catch block so we can handle any errors
  do {
    // fetch our items using our fetch request, save them in our items array
    items = try context.fetch(fetchRequest)
  } catch {
    print("Error fetching items: \(error)")
  }
  
  // reload our table to reflect any changes
  tableView.reloadData()
}
```

<a name="17"></a>
## 17) Compound Predicates

In order to retrieve the associated items for the category, and to apply the search text filter, we need to use a compound predicate.

Replace the `searchItems(searchText: String)` method with the following:

```swift
// this method's use is restricted to this file
fileprivate func searchItems(searchText: String) {
  // our fetch request for items
  let fetchRequest: NSFetchRequest<Item> = Item.fetchRequest()
  
  // a predicate allows us to create a filter or mapping for our items
  // [c] means ignore case
  let titlePredicate = NSPredicate(format: "title CONTAINS[c] %@", searchText)
  let categoryPredicate = NSPredicate(format: "category.name MATCHES %@", category?.name ?? "")
  
  // a compound predicate allows you to combine multiple predicates on the same request
  let compoundPredicate = NSCompoundPredicate(andPredicateWithSubpredicates: [categoryPredicate, titlePredicate])

  // the sort descriptor allows us to tell the request how we want our data sorted
  let sortDescriptor = NSSortDescriptor(key: "title", ascending: true)

  // set the predicate and sort descriptors for on the request
  fetchRequest.predicate = compoundPredicate
  fetchRequest.sortDescriptors = [sortDescriptor]

  // retrieve the items with the request we created
  do {
    items = try context.fetch(fetchRequest)
  } catch {
    print("Error fetching items: \(error)")
  }

  // reload our table with our new data
  tableView.reloadData()
}
```

<a name="18"></a>
## 18) Assignment

Create a simple personal journaling app called Noted Notables.

Your app should allow a user to do the following:

* Store a Note entity using CoreData (should have a title, date created, and body)
* View a list of all note titles (sorted by date created, newest to oldest)
* Search the list of notes by title.
* Add a new note in a ViewController (alert does not lend itself well to large text inputs)
* Delete a note by swiping the list item
* View a note ViewController by tapping the list item
* Delete a note by pushing a button in the detail view
