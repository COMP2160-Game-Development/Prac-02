# COMP2160 Prac 02: Car Chase
  
## Topics covered: 
* Vectors 
* Coordinate spaces
* Input

>## Discussion: Working Together (15 min)
>You are working on a large project for your studio. Your team leader has pretty much left you all to your own devices to work on things, <b>trusting</b> you all to get your work done. While things have been going smoothly, you soon notice a programmer colleague of yours is beginning to get overworked. As a result, they're cutting corners and missing milestones.
>
>A major deadline for the project is coming up. If you’re being <b>honest</b>, you’re not confident your colleague can make it without burning out, or turning in un-finished work, which would be a bit <b>disrespectful</b> to the rest of the team. You are not in any sort of management position, and the rest of the team seem to be unaware of the issue. What should you do? Refer back to the ACS code of professional ethics for how to navigate the situation.

## Today's Task
In this prac you will implement a car-chase game: 
https://uncanny-machines.itch.io/comp2160-week-2-prac<UPDATE>
 
The player controls the orange car with the following keyboard controls:

Movement: WASD or Arrow Keys<br>
Turbo Boost: SpaceBar

The blue car chases the orange car. The blue car slows to a stop when it gets close to the orange car.

The two main pieces of documentation for today are [The Transform class](https://docs.unity3d.com/2023.1/Documentation/ScriptReference/Transform.html) and [The new Input System and the Action Asset workflow](https://docs.unity3d.com/Packages/com.unity.inputsystem@1.6/manual/Workflow-ActionsAsset.html). Refer back to these pages whenever you get stuck.

## Step 1 – Get moving (10 min)
Open the SampleScene in the Scenes folder. It contains a single orange car (a rectangular block).

Click on the car with the Move tool in the Scene view and notice how its <b>local coordinate space</b> is aligned. Make sure you have the tool handle view set to Pivot and Local, using the tools at the top left of the Scene window. Your view should look something like this:

![An image of an orange cube representing a car in the Unity scene view. It's local axis is visible, with the world axis in the top right corner.](images/Week2_1.png)

Note the difference between the car's coordinate space and the <b>world coordinate space</b>, represented by the widget in the top right of the Scene view.

Create a script called `Drive` that moves the car forwards at a constant velocity using ```transform.Translate()```. You'll need to figure out which direction is "forwards" first. Taking a look at the coordinate systems, we can see that the car's "forward" direction appears to be going along its Z-axis (the blue arrow). We can therefore define a "forward" Vector3, as well as engine power, by adding the following to our `Drive` scripts constructor:

```
[SerializeField] private float power = 0f;
private Vector3 direction = new Vector3(0f,0f,1f);
```

We will then need to add some code to our ```Update``` method to move our car (don't forget to set your power in the Inspector):

```
transform.Translate(direction * power * Time.deltaTime);
```

Attach the script to the car and hit play. By default, Unity will apply this translation to an object's local coordiante space. However, you can specify the coordinate space. Experiment by replacing your translation with the following one by one, and hit play teach time to see what happens:

```
transform.Translate(direction * power * Time.deltaTime, Space.Self);
transform.Translate(direction * power * Time.deltaTime, Space.World);
transform.Translate(direction * power * Time.deltaTime, Camera.main.transform);
```

Write down your observations.

### CHECKPOINT! Save, commit and push your work now

## Step 2 – Going in circles (15 min)
Add code to make your car turn at a constant rate, using ```transform.Rotate()```. First, specifiy a turning rate parameter (in degrees):

```
[SerializeField] private float turningRate;
```

Remember to specify this in the inspector before running your game. Don't forget to check the [documentation for more info on Rotate()](https://docs.unity3d.com/2023.1/Documentation/ScriptReference/Transform.Rotate.html).

Figuring out which axis to rotate around can be a bit tricky. The way to think of this is we want to rotate around the axis that ISN'T changing. So, if an object is going to be pointing in different X and Z directions, then we want to apply the rotation on the Y axis. 

If we had to define this vector ourselves, it would be (0,1,0). However, Unity has a number of built-in Vector3 short-hands that we can use to make our life easier. Look at the [Vector3 scripting reference](https://docs.unity3d.com/2023.1/Documentation/ScriptReference/Vector3.html) and find a static property to use instead. Define a variable with a meaningful name (like `rotationAxis`) and assign it this Vector3.

```
transform.Rotate(rotationAxis * turningRate * Time.deltaTime, Space.Self);
```
Experiment with different parameters for your Drive script. How do the speed and the turning rate affect the radius of the car's turning circle? What is the formula for this? 

Consider which parameter is more designer-friendly: turning speed (in degrees per second) or turning-circle radius (in metres)? Why?

You may even want to expose some of your Vectors in the inspector so you can experiment with different directions for movement and axes for rotation.

### Transform Hierarchy

Let's make it a bit easier to visualise our car's front. Create a new cube object, making it a child of the car and repositioning/resizing it so we get something like this, representative of headlights:

![An image of the orange car, with a white box as its headlights.](images/Week2_2.png)

Create a new script called `Headlights` that prints the object's `transform.localPosition` and `transform.localRotation` to the console every Update (remember how we did this last week? Go check your code if you don't!). Attach this script to the Headlights object then press Play.

Note how its position and rotation values don't change, despite its transformations within the scene. Why is this happening? Note down your ideas. You can also chanage from `transform.localPosition` / `transform.localRotation` to `transform.position` / `transform.rotation` and view any differences.

### CHECKPOINT! Save, commit and push your work now.

## Step 3 - Chase car (20 min)
We often want objects in our scene to read and react to the transforms of other objects. Let's try this now.

Add another car to the scene and give it the provided blue material. Create a ```Chase``` script for this car. Add a parameter to the Chase MonoBehaviour to hold the transform of the target it is chasing. Make sure it is editable in the inspector:

```
public class Chase : MonoBehaviour 
{     
   [SerializeField] private Transform target;  
} 
``` 

### Basic chase behaviour 
In the Inspector, drag the orange car into the target slot on the chase car. Notice how Unity automatically finds the Transform component of the car. In general, if a field has a particular component type (Transform, SpriteRenderer, etc) then Unity will allow you to drop an object in that slot as long as it has the appropriate component. 

![An image showing the Orange Car's transform added to the Chase Class' Target parameter](images/Week2_Chase.png)

In the Chase MonoBehaviour's Update method, we want to get the distance between the two cars as a float. There are two problems to solve:
* First, get the vector from transform.position to target.position - how do you do it? Check the lecture notes and ask your demonstrator for help.
* Then, we need a method to find the magnitude of this vector (it's length). Have a look at the [Unity Documentation for Vector3](https://docs.unity3d.com/ScriptReference/Vector3.html) and try to find a Static Method that you can use to get the magnitude. This is our distance value.
  
If the car is more than some minimum distance (tunable parameter) from the target, it should move forwards in local coordinates. Otherwise, it should stop. Code this and test it. 

### Turning
To make the car turn appropriately, we need to know whether the target is to the left or the right. We can do this by transforming the target’s position into the car’s local coordinate space, thereby getting the target's position *relative* to the chase car.

Take a look at the following methods to determine which one you should use and why. If you're a bit stuck, have a look at the documentation for [TransformPoint](https://docs.unity3d.com/ScriptReference/Transform.TransformPoint.html) and [InverseTransformPoint](https://docs.unity3d.com/ScriptReference/Transform.InverseTransformPoint.html).


```
transform.TransformPoint(target.position)
transform.InverseTransformPoint(target.position)
target.TransformPoint(transform.position)
target.InverseTransformPoint(transform.position)
```
 
Once we have the point in the right coordinate space, we can check whether it is on the left or right by examining its x position. Negative x is on the left, positive is on the right.

Use this x value, plus the pattern for rotating the orange car you've already learnt, to get the blue car to turn to face the orange one. Call over your demonstrator if you get stuck!

### Slowing down
Having the car stop suddenly when it reaches the minDistance is a bit jarring. What value might you factor into your movement equation to achieve this? Write down some ideas and give them a try. Don't be afraid to [explore more documentation](https://docs.unity3d.com/ScriptReference/Mathf.Clamp.html) for a solution. Ask your demonstrator for clues!

## Step 4 – Adding input (30 min)
Let's now take control of the car's movement and turning by adding player input. We'll need to add the New Unity Input System to our project and disable the old one to get started.

Import the new Input System by opening the Package Manger (Window > Package Manager). Set the Packages filter to "Unity Registry" in the top left of the new Window (so it should read "Packages: Unity Registry"). Locate the Input System and click "Install", then press <b>Yes</b> to enable the new Unity backends.

![An image of the Package Manager with the Input System selected](images/Week2_3.png)

### Adding an Actions Asset
The new Input System offers many approaches. We will be taking the approach we believe is most suitable for learning the fundamentals.

Create a new Input Actions Asset (Assets > Create > Input Actions). Make sure to give it a meaningful name. I'm calling mine ```PlayerActions```. You'll also want to store it in a folder.

Select your Input Actions Asset to make it appear in the Inspector, then click "Edit asset" to start adding Inputs. <b> IMPORTANT! MAKE SURE YOU TICK THE "Auto-Save" box! </b>

![An image of the InputActions editor, with the "Auto-Save" box ticked.](images/Week2_4.png)

Let's consider what actions we want. We want the player to...

* Move the car forwards and backwards in local space by pressing "forwards" and "backwards" buttons respectively.
* Turn the car to the left and right in local space by pressing "left" and "right" buttons respectively.

We will need both a **Horizontal** and **Vertical** axis input.

Each Input Actions Asset contains both <b>Action Maps</b> and <b>Actions</b>. Action Maps can be thought of as different collections of Actions, allowing for different key-bindings. E.g, you might have a different Action Map for a menu and one for driving, then actions for navigating the menu, moving the car, etc. 

In our case, we only need one Action Map. Press the + symbol next to "Action Maps" to create a new Action Map. Let's call it "driving".

We can then create Actions that are part of this Action Map. Click the + next to "Actions" and name this first Action "movement". In the "Action Properties" column, we want to tell Unity how to interpret the input leading to this Action. Set the Action Type to "Value", which tracks changes to a Control state continuously, so is useful for things like movement.

We then need to set what Control Type we want to send to the Action Type. This is the data type used. In our case, we want to set this to "Vector 2", as we will be reading both X and Y values to determine the car's transformations.

The next step is to tell the Input System which inputs to read for this particular Action. Press the small + next to the movement action and select "Add Up\Down\Left\Right Composite". This will create a 2D Vector for this binding, where you can populate each direction with its own button.

For each of these directions, we need to add a binding. For today, let's set this to either the Arrow Keys or the WASD controls, whichever you prefer. To do this, select one of these directions and press the drop-down next to "Path". You can then press the key to have a list of corresponding keys appear.

Your Input Action Asset should looks something like this:

![An image of the Input Actions Asset, with the movement action set to a Composite binding.](images/Week2_5.png)

We can convert our Input Actions Asset into a C# Class, which we can then use in our scripts. In the Inspector, tick the "Generate C# Class" box now, then hit "Apply". The name you set for your asset is now the class name. As mine was called PlayerActions, this is what the examples will demonstrate.

We'll be making additional changes to our Actions as we go. With "auto-save" and  "Generate C# Class" both ticked, these changes will be written to the script as we go.

### CHECKPOINT! Save, commit and push your work now.

We now have our inputs all sorted out, but still need to tell our car how to react to them.

We want to give our Drive script its own copy of the PlayerActions class we've created. We also want to retrieve the actual movement action.

First, you'll need to add the InputSystem library to this script, so add `using UnityEngine.InputSystem;` at the top of your `Drive` class. Then, add the following to the constructor:

```
    private PlayerActions actions;
    private InputAction movementAction;
```
We then need to initiate things. We need to do this in the `Awake()` method, which runs before the `Start()` method (This order is detailed in the [execution of Unity methods documentation](https://docs.unity3d.com/Manual/ExecutionOrder.html)).

```
void Awake()
{
    actions = new PlayerActions();
    movementAction = actions.driving.movement;
}
```
Finally, we must enable the actions we want to use. Actions can be enabled individually, or we can enable an entire ActionMap at once. We will be enabling Actions individually to have greater control over when they are turned on and off. We want to do this in the ```OnEnable()``` method, which is called between ```Awake()``` and ```Start()```:

```
void OnEnable()
{
    movementAction.Enable();
}
```

As best practice, we should also Disable our Actions to prevent actions from being called in the event of the player being destroyed or otherwise disabled. The method ```OnDisable()``` is useful for this, as it runs just before an object is destroyed, and is traditionally used for any clean-up code.

```
void OnDisable()
{
    movementAction.Disable();
}
```

### Reading input: polling versus event-driven
We now need to decide how we are going to read the Input in this script, and what we are going to do with it. There are two ways to think about this problem: polling and event-driven. We'll be exploring both in this task.

We will use polling to control our car's movement and rotation. To do this, we want to read the input every frame and write it to a value. Because we have defined this input as a Vector2, we know that whatever keys we bound to "up" and "down" will be read as the positive and negative of that Vector's y axis. In `Update()`, add the following:

```
float acceleration = movementAction.ReadValue<Vector2>().y;
```
This will store the y value between -1 ("Down") and 1 ("Up"). We can then add this to our Translation equation:
```
transform.Translate(Vector3.forward * speed * acceleration * Time.deltaTime, Space.Self);
```

Save your script, hit play and test the car's behaviour. You might want to turn your `turningRate` to 0 first to give yourself some more control. You can also print the input value to the console for debugging, and to get more practice at using the console for debugging.

Next, following the patterns you've learnt so far, use the x value of your input to control the car's rotation.

## Step 5 - Modulating speed (15 min)
We're now going to give our car a "turbo" boost option, which allows the player to increase their speed for a short period of time. We'll use an event-driven approach to get this value.

Open your Input Action Asset again and add a new Action to the ActionMap. Name it "turbo". This time, we want the Action Type to be set to "Button", as we want to receive whether it is pressed or not. For more info on the difference between these types, [see the documentation](https://docs.unity3d.com/Packages/com.unity.inputsystem@1.0/manual/Actions.html#action-types).

The default binding only takes a single button/key. This will do fine for our purposes, so simply set the path of the Binding to a button of your choice - I'd recommend the SpaceBar.

In the Drive script, add an InputAction for the turbo and set-it up the same way you did movement in the ```Awake()``` and ```OnEnable()``` methods.

Rather than polling to handle our turbo, we will be using an event-driven approach, where we assign a method we want to call every time the turbo button is pressed to our turbo button delegate. See the lecture notes if you want to brush-up on delegates and how they operate.

First, we'll need two new variables: a private ```boost``` variable which we'll factor into our movement calculation, and a tunable ```boostRate``` that we can modify in the inspector. Following the same patterns we have so far, add this into your code. Should the boost be multipled into our movement, or added? What is the difference, and how might this impact gameplay?

Next, we should set-up the method that we want to call. ```OnTurbo``` seems like a good name:

```
void OnTurbo(InputAction.CallbackContext context)
{
    boost = boostRate;
}
```
We now need to assign it to our delegate so it is called whenever the turbo button is pressed. Each input action has three different callbacks: started, performed and canceled. In our case, we want to use performed, so we will assign this in the Awake method:

```
turboAction.performed += OnTurbo;
```

If you play the game now, pressing the boost button should make the player's car speed-up, but it never slows back down! Add another method, such as ```EndTurbo(InputAction.CallbackContext context)```, and assign it to your turbo action's ```canceled``` Callback. Use this method to turn off the turbo feature, so the boost will turn on when the player presses the button, and off when they release.

### Cleaning up your code
We've made a lot of long equations today! If you haven't already, consider going through your methods and seeing where things can be cleaned up. While we want to make sure our code is efficient, we also want to ensure it is readable, and values aren't being repeated, so as to avoid code smell. Consider some of the values you are working with and if anything can be made a bit neater. (Hint: do you still need to calculate the distance between cars to get magnitude?)

Chat to your demonstrator when being marked off for advise on making your code clearer.

## Prac Complete! To receive your mark for today, show your demonstrator:
* Your code for moving the cars.
* Your code for handling boosting, and why you did it that way.
* Your controls.

Good job! Don't forget to save, commit and push your work! See you next time!