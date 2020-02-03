# iOS Development: Building a Single View App

1. [ Introduction ](#1)
2. [ Getting Started ](#2)
3. [ Interface Builder ](#3)
4. [ Positioning View Elements ](#4)
5. [ Adding Assets ](#5)
6. [ The Simulator ](#6)
7. [ Linking Interface Elements to Code ](#7)
8. [ In Class Assignment ](#8)
9. [ Assignment ](#9)

<a name="1"></a>
## 1) Introduction

During this lesson we'll be creating a slot machine app called SlotMaster, so we can get a feel for how apps are built in XCode and using Cocoa Touch.

Here's what the finished product will look like!
<br />
<br />
<img src="https://i.imgur.com/JGcGjr1.gif" width="250" />

<a name="2"></a>
## 2) Getting Started

Open XCode and create a new Single View App project called "SlotMaster".

![](https://i.imgur.com/LwWETkY.png)

Use *your name* as the organization name, and *com.yourname* as the organization identifier.

You can leave Team as None for now. When we want to try out our app on an actual device later, we will need to add a team.

![](https://i.imgur.com/RRSJFXY.png)

<a name="3"></a>
## 3) Interface Builder

Interface Builder (IB) in XCode allows us to visualize the view layer of our app by using special files called storyboards. These are XML files behind the scenes.

### Navigator Panel

In the project Navigator panel (on the left), open the `main.storyboard` file.

If you can't see your Navigator panel, look in the top right corner to show/hide panels in XCode.

![](https://i.imgur.com/1agAtfi.png)

### Document Outline Panel

The Document outline panel is used to visualize the view hierarchy in Interface Builder. In the Document Outline panel, click on the View.

![](https://i.imgur.com/2eh5a1D.png)

### Inspectors Panel

The inspectors panel is the panel on the right-hand-side, and is used for interacting with view elements. It contains 6 inspectors, but the one you will likely use the most is the Attributes Inspector. 

In the Inspectors panel, select the Attributes Inspector.

![](https://i.imgur.com/gr1XC4Z.png)

### Changing Attributes

Let's change the background colour. Select the background attribute, and choose custom.

![](https://i.imgur.com/op9dOSb.png)

![](https://i.imgur.com/QaFXag5.png)

Select "Black" from the color window.

### The Object Library

Now that our background is black. Let's try adding some components to our view.

Click the Object Library button in the top toolbar.

![](https://i.imgur.com/QVfRoGz.png)

There are many different UIView components to choose from. Type in label and drag the Label component onto your view.

![](https://i.imgur.com/DanK0HU.png)

### Changing the Font

Select the label element and open the Attributes Inspector in the right-hand panel. Find the Font attribute, select the T icon > custom. Select any family, style and size.

![](https://i.imgur.com/yrEOVTL.png)

### Exercise

Try changing the font of the label to Copperplate Bold 24.0 pt. Change the color of the font to white. Set the label text to "Slot Master". Start by selecting the label, and opening the Attributes Inspector.(5 mins)

<a name="4"></a>
## 4) Positioning View Elements

Let's position our label in our view. In the Inspectors panel, open the Size Inspector. Change the x value to 109, and the y value to 87.

Because we are currently viewing our screen in iPhone 8 dimensions, the screen is 375 points in width and 667 points in height.

![](https://i.imgur.com/MGYiz3u.png)

### Screen Sizes and Resolutions

[A useful resource for referencing iOS screen resolutions and sizes](https://www.paintcodeapp.com/news/ultimate-guide-to-iphone-resolutions)

### Points vs. Pixels

**Pixel**: A square of colour (or transparent).

**Point**: 1/72 inch (a measure of length)

**Resolution**: ie. 72 ppi (pixels per inch)

Retina screens can fit more pixels per inch than regular screens. Points are what we work in with IB. Pixels are what is rendered on the screen. When an iPhone 8 screen is rendered, it is 750 x 1334 pixels.

### Adding our Image View Elements

Let's start by adding some UIImageView elements to our Slot Master app. The images contained in our images views wills later be randomized when our "PLAY" button is pushed, but for now we'll just add the required image view elements.

### Exercise

Open up the Object Library, search for "image view". Drag 3 image views to your view and position them. They should be 100 width and 100 height (square). (5-10 mins)

You should end up with something like this:

![](https://i.imgur.com/NCV1q8U.png)

<a name="5"></a>
## 5) Adding Assets

Now that we have our UIImageView elements in our view, let's add some image assets.

You can get the image assets for this class [here](https://github.com/philweier/ssd-ios-6day/blob/master/class2Assets/slotMasterAssets.zip)

Open the downloaded folder, select all files, and drag them to the Assets.xcassets folder in your Navigator panel.

![](https://i.imgur.com/a8FA7BW.png)

Note the naming conventions of the files. There are some image files named using the 2x and 3x postfix.

1x (Normal) = 1 pixel per point
2x (Retina) = 4 pixels per point
3x (Retina) = 9 pixels per point

Once the files are copied over, you should see all the images in their various sizes when you select Assets.xcassets.

![](https://i.imgur.com/V9Yb3tc.png)

Let's go back to our main.storyboard file and add an image to each of our image views.

Click on one of the UIImageView elements. In the Attributes Inspector, find the image field, and start typing the name of the image asset you want to load. You should see a drop down of available assets.

![](https://i.imgur.com/MtfS6Mh.png)

<a name="6"></a>
## 6) The Simulator

Let's try to build and run our code. Since we have been working with the iPhone 8 screen size in IB, select the iPhone 8 simulator.

![](https://i.imgur.com/nU9339s.png)

Once you've selected the correct device size, click the run button in the top toolbar. The simulator should load your app. There's not much to do in our app right now. Let's add some functionality.

<a name="7"></a>
## 7) Linking Interface Elements to Code

Let's add a button to our Slot Master app. Open the Object Library and search for button. Drag a button onto your view.

### Exercise

Change the size of your button to width of 145 and a height of 73. Position it in the center of the view below the row of image views. Change the font and background color to something you like. Change the button text to "PLAY". (5 mins)

Your button might look something like this:

![](https://i.imgur.com/AACat2X.png)

### Actions

We need to decide what should happen when our button is tapped. Since we are building a slot machine app, we would like to randomize the image inside each image view.

We can do this by linking an action to our button.

Let's start by changing our XCode layout in order to make this easier to do. Click the Assistant Editor button. It also helps to close your Navigator and Inspector panels to free up more space.

You should see the ViewController.swift file in the right-hand editor.

![](https://i.imgur.com/V657XfM.png)

Select your button in IB. Click control and drag over to your view controller file

![](https://i.imgur.com/9CslcL8.png)

When you release the control drag, you should get a popup. Be sure to select Action as the connection, give the action a name like "playButtonTapped". Click "Connect".

![](https://i.imgur.com/PDB1Tmq.png)

In your code, you will now see an action. This function will now be executed every time your button is clicked. Before adding some code to our action, let's connect our image view elements to code.

### Outlets

Select one of the image view elements. Click control and drag across to the view controller file. 

![](https://i.imgur.com/Pt2cKK7.png)

Let's give our image view a name like "imageView1". Ensure the type is "UIImageView". Click "Connect".

![](https://i.imgur.com/eDvZDSB.png)
![](https://i.imgur.com/kF8lOTv.png)

### Exercise

Hook up the other two image views to code (3 mins).

### Randomization in Swift

Now that our elements are connected, let's add some code to our play button action so something actually happens when we click it. Since we are building a slot machine app, let's have the image view load a different image randomly when our play button is tapped.

In Swift 4, we have the method `randomElement()` which can be called on an array.

Let's create a string array of possible image names, select one at random, and load this image into our first image view each time our play button is tapped.

![](https://i.imgur.com/9nGCvev.png)

### Exercise

Try adding code to choose and assign a random image for the other two image views. (10 minutes)

### Solution

![](https://i.imgur.com/PRgaZoJ.png)

### Positioning Across Different Screen Sizes and Resolutions

We have been building in IB for iPhone 8. In the screen selector panel on the bottom of IB, select a different sized screen or orientation. Your UI elements may look skewed.

![](https://i.imgur.com/GBlTCd1.png)

![](https://i.imgur.com/CYeTmXa.png)

![](https://i.imgur.com/rKrF8Jb.png)

This is because we positioned our elements specifically for iPhone 8 and not all screens. In order to make our app look nice on all screens, we will need to use something called Auto Layout. We will learn more about this in the next class.

<a name="8"></a>
## 8) In Class Assignment

Add three UILabel elements to your Slot Master app. One label should go under your row of image view elements, and should show the user how many points they got during each play. The othere labels should show the total number of points the user has achieved (one will say "Total Points" and the other will show the cumulative number of points).

**When 0 images match = 0 points
When 2 images match = 2 points
When 3 images match = 5 points**

It should look something like this:
<br />
<br />
<img src="https://i.imgur.com/JGcGjr1.gif" width="250" />

<a name="9"></a>
## 9) Assignment

Create a Rock Paper Scissors game.

**It should be a Single View App and should have the following elements:**

- 2 UIImageView elements for each player's image
- 2 UIButtons, that when clicked, select a random image (rock, paper or scissors), and load this image into each player's UIImageView
- A title label
- A label that shows who won

**Rules:**

- 2 players
- Rock beats scissors
- Scissors beats paper
- Paper beats rock
- Players should be able to play more than once
- Once a player takes their turn, they are not able to take another turn until the other player has played
- It should not matter who goes first

**Your app might look something like this:**
<br />
<br />
<img src="https://i.imgur.com/MZqPGbh.png" width="250" />

Get the image assets for this project [here](https://github.com/philweier/ssd-ios-6day/blob/master/class2Assets/rockPaperScissorsAssets.zip)
