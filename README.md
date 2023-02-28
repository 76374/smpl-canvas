# Basic usage
All objects or their parts which need to be rendered, need to extend the [DisplayObject](https://github.com/alexbolbat/smpl-canvas#displayobject) class and be a direct or nested child of a [Stage](https://github.com/alexbolbat/smpl-canvas#stage) instance. There are some ready to use classes from the box, but in most cases there might be a need to write custom classes which extend [DisplayObject](https://github.com/alexbolbat/smpl-canvas#displayobject). To provide instructions on how to render an object, it’s necessary to override the `render` method and use [RenderTools](https://github.com/alexbolbat/smpl-canvas#rendertools) provided there as an argument to draw shapes, lines, text.
```
class Circle extends DisplayObject {
 override render (renderTools: RenderTools) {
   renderTools.circle({
     x: 40,
     y: 40,
     radius: 40,
     fillColor: 'blue'
   })
 }
}

const stage = new Stage(document.getElementById('canvas'))
stage.width = 100
stage.height = 100
stage.addChild(new Circle())
```
[Output](/ex001.png)
To make an object interactive i.e. receive input events, it needs to extend the [hitTest](https://github.com/alexbolbat/smpl-canvas#hittest-x-number-ynumber-boolean) method to define object bounds. If an area to click needs to be defined by object size and position, it’s enough to assign the [hitTestInBounds](https://github.com/alexbolbat/smpl-canvas#hittestinbounds-boolean-readwrite) property.
```
class Circle extends DisplayObject {
 constructor () {
   super()

   this.width = 80
   this.height = 80
   this.hitTestInBounds = true
   this.cursor = 'pointer'
 }
…
```
Now the mouse cursor is changed to “pointer” when it hovers the circle. But it is not a perfect solution in this particular case since the area is defined by the 80x80 square but not a circle. It changes the cursor out of the circle area (in the corners). So instead of changing the [hitTestInBounds](https://github.com/alexbolbat/smpl-canvas#hittestinbounds-boolean-readwrite) property, it is better to override the [hitTest](https://github.com/alexbolbat/smpl-canvas#hittest-x-number-ynumber-boolean) method.
```
override hitTest (x: number, y: number): boolean {
 const radius = Math.min(this.width, this.height) / 2
 // returning true if the distance to the center is less than radius
 return Math.sqrt((x - radius) ** 2 + (y - radius) ** 2) <= radius
}
```
To redraw the object, there is an [update](https://github.com/alexbolbat/smpl-canvas#updated-signal-read-only) signal property. It is used when visual changes need to be applied. It tells [Stage](https://github.com/alexbolbat/smpl-canvas#stage) the object was changed and it will redraw the object on the next frame
```
private isOn = false

constructor (radius: number) {
 super()

 this.width = radius * 2
 this.height = radius * 2
 this.cursor = 'pointer'

 // adding listeners for mouse click event
 this.click.add(this.handleClick, this)
}

private handleClick () {
 this.isOn = !this.isOn
 this.updated.emit()
}

override render (renderTools: RenderTools) {
 renderTools.circle({
   x: 40,
   y: 40,
   radius: 40,
   // changing color depending on isOn property
   fillColor: this.isOn ? '#d93434' : '#2ac942'
 })
}

override hitTest (x: number, y: number): boolean {
  const radius = Math.min(this.width, this.height) / 2
  return Math.sqrt((x - radius) ** 2 + (y - radius) ** 2) <= radius
}
```
Display objects can be grouped and stored in containers. Base class for containers is [DisplayObjectContainer](https://github.com/alexbolbat/smpl-canvas#displayobjectcontainer).
```
const container = new DisplayObjectContainer()
container.addChild(new Circle(40, 'green'))
container.addChild(new Circle(40, 'red')).x = 90
stage.addChild(container)
```
[Output](/ex002.png)
Their inheritance can be used as creating objects templates. As [DisplayObjectContainer](https://github.com/alexbolbat/smpl-canvas#displayobjectcontainer) extends [DisplayObject](https://github.com/alexbolbat/smpl-canvas#displayobject), it also can render primitives. Child components are drawn on top of their parents.
```
class PairOfCircles extends DisplayObjectContainer {
 constructor () {
   super()

   this.addChild(new Circle(40, 'green'))
   this.addChild(new Circle(40, 'red')).x = 90
 }
}

const container = new DisplayObjectContainer()
container.addChild(new PairOfCircles())
container.addChild(new PairOfCircles()).y = 90
stage.addChild(container)
[Output](/ex003.png)
```
Some components like `Grid`, `VerticalLayout` are able to manage their children's positioning automatically. It is achieved using the [updateLayout](https://github.com/alexbolbat/smpl-canvas#updatelayout-) method. This method is called before drawing and can be overridden to implement custom positioning and sizing.
```
const colors = ['#32a852', '#2387c4', '#631ad9', '#d91a7a', '#ff8940']
const grid = new Grid()
colors.forEach((color, i) => {
 grid.addInCell(new Circle(10, color), i, 0)
 grid.addInCell(new TextField(color), i, 1)
})
stage.addChild(grid)
[Output](/ex004.png)
```
# Class diagram
![Class diagram](/canvas.drawio.png)
# DisplayObject
A base class to draw primitives. This class is supposed to be overridden to provide instructions on: how to draw it, behave depending on mouse events, etc. An instance of the subclasses needs to be used as a child of a [Stage](https://github.com/alexbolbat/smpl-canvas#stage) instance to be rendered and receive input events.
## Instance properties
### cursor: CursorType (read/write)
Is used to change the type of mouse cursor when it is over the current display object. Checking if the object is under mouse cursor depends on the hitTest method.
### hitTestInBounds: boolean (read/write)
When the value is set to true, the hitTest method returns true (if it’s not overridden) when the mouse cursor is within a rectangle defined by width and height of the current display object. 
Example:
Makes the “pointer cursor” over a square area size of 20 pixels.
```
constructor () {
 super()

 this.height = 20
 this.width = 20
 this.cursor = 'pointer'
 this.hitTestInBounds = true
}
```
### mouseClick: Signal<MouseData> (read-only)
Triggers when the current object is clicked. 
Example:
```
constructor () {
  super()
  this.click.add(this.handleClick, this)
}

private handleClick (mouseData: MouseData) {
  // perform actions
}
```
**Note: Object bounds for mouse events are defined by the [hitTest](https://github.com/alexbolbat/smpl-canvas/blob/main/README.md#hittest-x-number-ynumber-boolean) method.**
### mouseMove: Signal<MouseData> (read-only)
Triggers when the mouse cursor moves over the current object.
### mouseOver: Signal<MouseData> (read-only)
Triggers when the mouse cursor enters the current object.
### mouseOut: Signal<MouseData> (read-only)
Triggers when the mouse cursor leaves the current object.
### height: number (read/write)
Describes the length of the current object in y-axis. 0 by default. This value can be set when the object instance is created (e.g. in the constructor) to have correct layout calculation and avoid overlapping with other objects within layout containers like `VerticalLayout`, `Grid` etc. It doesn’t stretch or shrink the object by default. To redraw the object according to its new height the setter can be overridden to call the update signal to force the redrawing process if the height takes place in the render method.
In the following example the display object is a vertical line, which is updated when its width changes.
```
override set height(value: number) {
 super.height = value
 this.updated.emit()
}

override render(tools: RenderTools) {
 tools.line([{ x: 0, y: 0 }, { x: this.height, y: 0}])
}
```
### name: string (read/write)
Defines an object instance name. Can be used to distinguish or find the object in the display objects tree.
### updated: Signal (read-only)
Needs to be triggered every time when the object instance is changed and requires redrawing. See the example in the [height](https://github.com/alexbolbat/smpl-canvas/blob/main/README.md#height-number-readwrite) property description.
### width: number (read/write)
Describes the width of the current object. Works similar to height property but for y axis, check details there.

## Instance methods
### render (tools: RenderTools)
The method is supposed to be overridden to use [RenderTools](https://github.com/alexbolbat/smpl-canvas/blob/main/README.md#rendertools) passed as the argument in order to provide instructions to render shapes, lines, etc. 
### hitTest (x: number, y:number): boolean
Takes coordinates of the mouse cursor in the object local dimension. By default returns `false`. Can be overridden to define a custom hit area for example if the object is a circle, line, etc. This method and mouse signals are called / triggered by [Stage](https://github.com/alexbolbat/smpl-canvas/blob/main/README.md#stage) automatically when the display object is direct or nested its child. If [hitTestInBounds](https://github.com/alexbolbat/smpl-canvas/blob/main/README.md#hittestinbounds-boolean-readwrite) property has been set to true, the method checks if the coordinates are within a rect defined by [width](https://github.com/alexbolbat/smpl-canvas/blob/main/README.md#width-number-readwrite) and [height](https://github.com/alexbolbat/smpl-canvas/blob/main/README.md#height-number-readwrite) of the current object.
```
// circle hit area. It checks if distance from mouse coordinates to the center of the circle is less or equal then its radius
override hitTest (x: number, y: number): boolean {
 return Math.sqrt((x - this.radius) ** 2 + (y - this.radius) ** 2) <= this.radius
}
```                                                                                 
### dispose
Cleans up all signals. Can be overridden to clean up custom ones or remove references to other objects to avoid memory leaks.
# RenderTools
Provides an interface to draw shapes, lines, text. A RenderTools instance is provided to the render method of [DisplayObject](https://github.com/alexbolbat/smpl-canvas/blob/main/README.md#displayobject) as an argument. By calling the methods described below it accumulates data which is used in `Render` to draw display objects. To make the drawing process work it is enough to create a [Stage](https://github.com/alexbolbat/smpl-canvas/blob/main/README.md#stage) instance and add a display object as its child. Draw methods return the current instance to chain calls.
```
const stage = new Stage(document.getElementById('canvas'))
const rect = new RectContainer()
rect.fillColor = 'blue'
stage.addChild(rect)
```
## Instance properties:
### renderProps: RenderProps[] (readonly)
Information which is used by Render about sequence of operation required for object drawing. 
## Instance methods:
### line (line: Point[], color?: string): RenderTools 
Draws a broken line defined by provided points abd color. Returns current instance to chain draw calls.
### circle (circle: Circle): RenderTools
Draws a circle. `Circle` data structure defines center, radius, fill and stroke colors. Returns current instance to chain draw calls.
### shape (shape: Shape): RenderTools
Draws a shape defined by `Shape` data structure which includes points to build various shapes, their fill and stroke color. Returns current instance to chain draw calls.
### text (text: string, textProps: TextProps): RenderTools
Takes text string and its properties like font, size, leading as arguments. It is used by `TextField` and in most cases text rendering can be done using that class. Returns current instance to chain draw calls.
### textLines (lines: string[], textProps?: TextProps): RenderTools
Similar to the [text](https://github.com/alexbolbat/smpl-canvas/blob/main/README.md#text-text-string-textprops-textprops-rendertools) property but take a string array as an argument to render multiline text.
# DisplayObjectContainer
Visual container for display objects. Child position becomes relative to its parent after adding. As it extends DisplayObject, it has all its possibilities for positioning, rendering etc,  and it can contain other containers as well.
## Static methods
### isContainer (obj: any): boolean
Checks if the argument is an instance of the `DisplayObjectContainer` class.
## Instance methods
### addChild (displayObject: DisplayObject): DisplayObject
Adds provided `displayObject` to the container and returns it.
### foreachChild (callbackFn: (child: DisplayObject) => void)
Similar to `foreach` in `Array`, makes loop through all children of the container. Provided function that takes a `DisplayObject` instance as the argument.
### removeChild (displayObject: DisplayObject)
Removes provided `displayObject` from the container.
### removeAll (withDispose = true)
Removes all children from the container. `withDispose` argument is `true` by default and when it is `true` it calls `dispose` method in all children when removing them.
### updateLayout ()
The method is called from a [Stage](https://github.com/alexbolbat/smpl-canvas/blob/main/README.md#stage) instance when the container is its direct or nested child. Depending on contained objects, it increases its size if the child is out of the container bounds defined by its width and height. For example, if the width of a container is 0, and it has a child with width = 5 and x = 10, the container's width will be 15 after the update. `updateLayout` call goes from the deeps child upward. Which means `updateLayout` is called in all child containers first. This method can be overridden to update the size differently or other properties before rendering.
## Inherited properties and methods
[cursor: CursorType](https://github.com/alexbolbat/smpl-canvas/blob/main/README.md#cursor-cursortype-readwrite)
 
[cursor: CursorType](https://github.com/alexbolbat/smpl-canvas/blob/main/README.md#hitTestInBounds-boolean-readwrite)
 
[click: Signal<MouseData>](https://github.com/alexbolbat/smpl-canvas/blob/main/README.md#mouseclick-signal-read-only)
 
[mouseMove: Signal<MouseData>](https://github.com/alexbolbat/smpl-canvas/blob/main/README.md#mousemove-signal-read-only)
 
[mouseOver: Signal<MouseData>](https://github.com/alexbolbat/smpl-canvas/blob/main/README.md#mouseover-signal-read-only)
 
[mouseOut: Signal<MouseData>](https://github.com/alexbolbat/smpl-canvas/blob/main/README.md#mouseout-signal-read-only)
 
[height: number](https://github.com/alexbolbat/smpl-canvas/blob/main/README.md#height-number-readwrite)
 
[name: string](https://github.com/alexbolbat/smpl-canvas/blob/main/README.md#name-string-readwrite)
 
[updated: Signal](https://github.com/alexbolbat/smpl-canvas/blob/main/README.md#width-number-readwrite)
 
[width: number](https://github.com/alexbolbat/smpl-canvas/blob/main/README.md#updated-signal-readonly)
 
[dispose()](https://github.com/alexbolbat/smpl-canvas/blob/main/README.md#dispose)
 
[render(tools: RenderTools)](https://github.com/alexbolbat/smpl-canvas/blob/main/README.md#render-tools-rendertools)
 
[hitTest(x: number, y: number): boolean](https://github.com/alexbolbat/smpl-canvas/blob/main/README.md#hittest-x-number-ynumber-boolean)
# Stage
Main container which handles render process and input events for nested display objects.
 
Update process includes next steps:
1. Cleaning up the canvas
2. Unregistering objects which are not belong to Stage direct or nested children. E.g. which were removed using removeChild method of any of nested DisplayObjectContainers.
2. Registering new objects that were added to the rendre hierarchy (become direct or nested children).
updating layouts: calling updateLayout methods of DisplayObjectContainers, TextFields starting from the deepest child going upwards until it meats Stage.
3. Updating canvas size
4. Rendering all display objects (using their render methods)
## Constructor
Required HTMLCanvasElement as an argument.
## Instance methods
### forceUpdate ()
Forces redraw process. Usually, when redrawing is required, `DispalyObject` instances call their `update` signal. But if for some reason update is needed in other way, `forceUpdate` can be calles instead. 
### localToGlobal (displayObject: DisplayObject): Point
Converts object position to global coordinates. It takes nested `DisplayObject` instance and returns its position in global coordinates
```
const stage = new Stage(canvas)

const container = new DisplayObjectContainer()
container.x = 10
container.y = 30
stage.addChild(container)

const button = new CircleButton(40)
button.x = 20
button.y = 20
container.addChild(button)

stage.localToGlobal(button) // { x: 30, y: 50 }
```
## Inherited properties and methods
cursor: CursorType
hitTestInBounds: boolean 
mouseClick: Signal<MouseData>
mouseMove: Signal<MouseData>
mouseOver: Signal<MouseData>
mouseOut: Signal<MouseData>
height: number
name: string
updated: Signal
width: number
addChild(): DispalyObject
dispose()
foreachChild(callbackFn: Function)
render(tools: RenderTools)
hitTest(x: number, y: number): boolean
removeChild(displayObject: DisplayObject)
removeAll()
updateLayout()
# Example
The following example represents a circle red button. It changes color from green to red after clicking on it.
```
class CircleButton extends DisplayObject {
 // the button color depends on this value
 private isOn = false

 constructor (private radius: number) {
   super()

   // set the cursor to "pointer" when it's hovered
   this.cursor = 'pointer'
   // updating size
   this.width = radius * 2
   this.height = radius * 2

   // adding listeners for mouse over/out events
   this.click.add(this.handleClick, this)
 }

 override set width (value: number) {
   super.width = value
   this.updated.emit()
 }

 private handleClick () {
   this.isOn = !this.isOn
   // emitting "update" to make getRender be called with updated isOn value
   this.updated.emit()
 }

 override render (tools: RenderTools) {
  tools.circle({
   radius: this.radius,
   // circle center
   x: this.radius,
   y: this.radius,
   // filling with red or green depending on if the button has been clicked
   fillColor: this.isOn ? '#d93434' : '#2ac942',
 })
}

 // overriding hitTest to make the area have shape of the circle
 override hitTest (x: number, y: number): boolean {
   // returning true if the distance to the center is less than radios
   return Math.sqrt((x - this.radius) ** 2 + (y - this.radius) ** 2) <= this.radius
 }
}

const canvas = document.getElementById('canvas')
const stage = new Stage(canvas as HTMLCanvasElement)
const button = new CircleButton(40)
button.x = 20
button.y = 20
stage.addChild(button)
```
