# WUIDom

## Motivation for WUI

Wizcorp UI (hereafter referred to as 'WUI') is a series of UI elements that were created to minimize
programmer interaction with the DOM and eliminate the need for programmers to create HTML.
Here at Wizcorp, we make single-page applications that often have many thousands of lines of
generated HTML. Therefore, even a single DOM query can be very slow, resulting in an unacceptably
slow user experience. WUI allows the programmer to create HTML elements while minimizing
interaction with the DOM, and maintain fine-level control over exactly when, where and in what
manner data is displayed and how events are handled.

WUI also allows for simple interaction with Tomes, another Wizcorp API, to allow DOM elements to be
updated whenever the underlying data associated with them changes. This is useful for such things as
validation, real-time data display and for asynchronous processing. By fully leveraging the
non-blocking nature of Node.js, WUI will perform nearly real-time updating of data and allow the
programmer to avoid DOM calls, have more compact code and allow for more reusable user interface
components.

## What WUIDom is

WUIDom is the base object of the WUI system. It represents the smallest unit of programmatic logic,
an object corresponding to an underlying HTML element and that associated data (including, but not
limited to, a Tome), events and other properties. WUIDom allows the programmer to keep a reference
to generated elements (if desired) or to create throw-away elements. A simple function can create a
generic UI template or create a custom view. Because WUIDom inherits from EventEmitter,
programmer-defined events can be conditionally emitted and handled by WUI or other modules.

## WUI Components

One of the major goals of WUI, aside from reducing interaction with the DOM, is to enable developers
to create re-usable user interface components, common behaviours and reusable views. When creating
WUI components, please try to keep that goal in mind. The ideal WUI component can be used on many
views and ideally that means to not making too many assumptions about the underlying data or the the
business logic of the game.

Below is a partial list of the existing WUI components, and what their primary functions are:

### WUIButton

You can read the WUIButton documentation [here].

### WUIView

You can read about the WUIView documentation [here].

### WUIImage

You can read more about the WUIImage documentation [here].

### Creating a new WUIDom element

There are a few ways to create a WUIDom element. The first, and easiest, way is to create a
WUIDom element is to use the createChild method. Another way is to use createElement and then
insert the element using one of WUIDom's insertion methods. Yet another way to create a WUIDom is
to use the WUIDom constructor and use the assign function, which is more useful for creating the
first WUIDom element on a page.


#### Creating the first WUIDom element on a page

Example:

```javascript
var examplePage = mage.loader.getPage('examplePage')
var htmlElement = page.querySelector('.someClass');
var newWuiElement = new WUIDom();
newWuiElement.assign(htmlElement);
```

In the above example, we see how we might create a new WUIDom object on a page called examplePage.
Please note that the page does not always need to be loaded when creating a new WUIDom element.
After the call to mage.loader.getPage(), we can query the DOM with the call to querySelector().
This call should be done at most once per page, because the call to the DOM is rather slow. If you
have a sub-component, it is better to pass in a a WUIDom element, or to use document.createElement.
After that we create a new WUIDom element by calling the constructor.

The constructor for WUIDom should not need to be called in most cases, as it is already done
for you by WUIDom.createChild() or WUIDom.createElement(). In this case, we are calling the
constructor so that the new instance can be assigned directly to the existing HTML element.
Once an initial WUIDom element exists, we should use it to create any other new WUIDom elements.
This allows for pages to be built iteratively.

#### createChild

NOTE: To use this method, you **must** have an existing WUIDom parent element.

Calling the method:

```javascript
parentElement.createChild(tagName, options);
```

The method createChild will call the function createElement to make the element you specify,
and then immediately call appendChild to insert the element. The result is a new child element is
immediately inserted as the last child of parentElement. Please note that this method is the
preferred way to create new WUIDom elements.

The createChild method takes an HTML tag name (such as 'input', 'label', 'div' or so forth)
and a list of associated options. WUI created an underlying rootElement (discussed at length below)
and appends the element as the last child element of the parent (caller) element.

An options element can contain numerous keys that are used by WUIDom to create the HTML element.
Consider the following example:

```javascript
var options = { text: 'Example', className: 'exampleClass', style: { margin: '5px', display:
'inline-block' }, attr: { type: 'text', maxlength: '4' }};
var newInputElement = parentElement.createChild('input', options);
```

In the above example, the code assumes parentElement already exists and is also a WUIDom object
(see above for creating the first WUIDom object in your code!). Here we create an input tag
(could just as easily be a 'div' or 'button'.)

WUI reads the options object next, and creating a child text-node based on the user-supplied 'text'
option, in this case using 'Example' for the text. It then applies the style and class name.
Finally, WUI will add each attribute specified by the keys in the 'attr' object. The resulting
HTML markup can be seen below:

```html
<input style="margin: 5px; display: inline-block;" class="exampleClass" type="text" maxlength="4" value="Example" />
```

This element is created, and immediately appended to the parentElement. In general, it is better
for you to add attributes under the 'attr' object at object initialization time. This minimizes the
number of repeated DOM calls (which are slow) and allows you to leverage WUIDom's attribute cache
for even greater performance.

After the underlying element is created, it is stored in the variable called 'newInputElement'.
This allows you to perform other operations on the newly created WUIDom element.

#### createElement

Calling the method from a WUIView:

```javascript
var newElement = this.createElement.call(tagName, options);
```

This method is used to create a new WUIDom element. Unlike the createChild method, please note that
the element 'newElement' is **not** appended to the parent element, even if it is called from a
parent element. Therefore, the element is created but not added. Note that the WUIDom reference is
still stored in newElement, and can be used by other functions. Events, tomes and so forth can still
be bound to a WUIDom element even before it is attached (in fact, the event might even be programmed
allow the WUIDom object to attach itself to a parent).

Read the section below for methods for inserting an existing WUIDom element. It is best to use this
method when you do not want to immediately insert a WUIDom element, when you want to conditionally
insert an element or when you want to insert a child element in a custom manner.


### Inserting a WUIDom element

There are many methods for inserting a WUIDom element on to the document. Note that these methods
all assume that a WUIDom element is being attached to another WUIDom element. For an example of
creating new WUIDom elements, please read above.

#### appendChild

Calling the method:

```javascript
var childElement = parentElement.createElement('label', { text: 'Example' });
parentElement.appendChild(childElement);
```

In the above code, we see that a childElement is created as a label with the text 'Example', and
then appended to a previously existing parentElement. In the normal case, it is better to call
parentElement.createChild(), since it will perform the appendChild automatically. However, it is
sometimes useful to not append an element immediately after it is created.

#### appendTo

Calling the method:

```javascript
if (someValue >= 5) {
    var childElement = parentElement.createElement('label', { text: 'Example' });
    childElement.appendTo(parentElement); // Equivalent to parentElement.appendChild(childElement);
}
```

In the above example code, if the value of someValue is greater than or equal to 5, a child
element is created using createElement, and then appended to a previously existing parentElement.
One of the main advantages of appendTo and appendChild, is that elements can be conditionally or
dynamically added based on programmatic logic. This is useful for views that will change depending
on program execution or external factors.

Please note that this method requires an existing parentElement.

#### insertBefore

Calling the method:

```javascript
var ListElement = function(textValue) {
    WUIDom.call(this);
    this.assign('li', { text: textValue});
}

var ulElement = pageElement.createChild('ul');
var secondElement = ulElement.createChild(new ListElement('Item 2' ));
var firstElement = ulElement.createElement(new ListElement('Item 1'));
firstElement.insertBefore(secondElement);
```

In this example, we create a ul element, and two li elements. First element is defined using
createElement and then inserted before the second element on the last line of the example.
This method is mostly useful for when you need to prepend an element to a list of existing elements,
such as a validation message notification panel.

Please note that the argument passed to this function must be a WUIDom element.
Also, it **must** have a parent node for this method to work successfully.

#### insertAsFirstChild

Calling the method:

```javascript
var ListElement = function(textValue) {
    WUIDom.call(this);
    this.assign('li', { text: textValue});
}

var ulElement = pageElement.createChild('ul');
ulElement.createChild(new ListElement('Second child'));
ulElement.insertAsFirstChild(new ListElement('First child'));
```

In this example, we create a ul element, and then append a ListElement second child.
The final line creates the first child, and inserts it as the first child. This method is useful for
when you want to prepend an item to a list, but might not have a readily available reference to the
current firstChild element (such as a dropdown list).

Please note that the object this method is called from is assumed to be the parent node. The
programmer should of course be careful to not try and make invalid child elements.

#### insertChildBefore

Calling the method:

```javascript
parentElement.insertChildBefore(newChildElement, elementToInsertBefore);

var ListElement = function(textValue) {
    WUIDom.call(this);
    this.assign('li', { text: textValue});
}
var ulElement = pageElement.createChild('ul');
ulElement.createChild(new ListElement('First child'));
var thirdChild = ulElement.createChild(new ListElement('Third child'));
ulElement.insertChildBefore(new ListElement('Second child'), thirdChild);
```

This method is different from the insertBefore method, in that it allows you to explicitly specify
the parentElement. The first argument of this function specifies a new child element, and the second
specifies a WUIDom element that the new child Element should be inserted before.

As in the previous examples, the parameters mentioned are all assumed to be WUIDom objects.

#### Binding to a Tome

WUIDom supports binding to a [Tome][1] object, and updating the underlying HTML element with
specific data. By binding a Tome to a WUIDom element, the programmer can leverage the full power
of Node.js to allow for real-time updating of interfaces when changes are made at a data-level.

The code below demonstrates how the contents of a Tome might be bound to a WUIDom element.

[1]: https://github.com/Wizcorp/node-tomes/blob/master/README.md

##### Example

```javascript
var questDisplay = pageElement.createChild('div');
questDisplay.bindToTome(mage.player.questLog, function () {
questDisplay.setHtml('');

for (var quest in mage.player.questLog) {
    if (mage.player.questLog.hasOwnProperty(quest)) {
            var questLineInput = questDisplay.createChild('input', { type: 'checkbox',
            disabled: 'disabled', checked: quest.isComplete ? 'checked' : ''});
            questLineInput.setText(quest.title);
        }
    }
});
```

In the example above, we bind the 'mage.player.questLog' Tome to the element questDisplay.
The first argument tells the questDisplay div which Tome it will use. The second argument is a
function that is called every time data within the Tome changes. So every time the questLog changes,
Tomes will fire an event and WUIDom will run this update function in order to refresh the player
quest log display.

In the case of this example, the quest log renders as a checkbox list.
Each checkbox is checked if the quest has been completed, and contains the title of the quest.
It is important to note that the function provided for argument 2 is executed every single time a
data change occurs, regardless of how significant the change is. Because of this, it's often best to
provide concise, efficient update functions.

#### Events

WUIDom inherits from EventEmitter, and as such it allows the programmer to define and catch their
own custom events. In addition, WUIDom can be configured to support the normal DOM events such as
onchange, onclick and others.

##### allowDomEvents

The allowDomEvents function can be called from any WUIDom element, in order to tell WUI that
the element will listen for DOM events. The correct syntax for DOM events is 'dom.event'.
Therefore the change event would be emitted as 'dom.change'.

Note that by default, DOM events are **not** propagated through WUI. **This method must be
called in order to enable DOM events on a WUIDom object.**

##### Example

```javascript
var playerXPElement = pageElement.createChild('input', { type: 'text', disabled: 'disabled' });

playerXPElement.allowDomEvents();

playerXPElement.on('dom.change', function() {
    var xp = playerXPElement.rootElement.value;
    var isLevelUp = checkLevelUp(xp);

    if (isLevelUp) {
        showLevelUpAnimation();
        playerXPElement.emit('levelUp');
    }
});
```

In this example, we create a WUIDom element called playerXPElement.
The function call to allowDomEvents tells WUI that the element that wants to consume DOM events.
In the final line, we tell the element to run the function every time the dom.change event is fired.

When the method runs, it will check to see if the player has gained a level, and then
possibly display an animation. If the player levels up, an event called levelUp is emitted.
This is just one example of how events might be used with WUI. Please see [here][0] for a list of
DOM events.

[0]: https://developer.mozilla.org/en-US/docs/DOM_Client_Object_Cross-Reference/DOM_Events

### Operations on a WUIDom element

WUIDom supports a number of functions that can be used to interact with the underlying element,
even after the element has already been added to the DOM. One primary purpose of WUI, is to allow
these operations to be performed without having to pay the cost of an inefficient DOM lookup every
time we wish to run a command.

#### assign

The assign method takes an HTML tag and an object map for options. Valid options include text, attr
(an object of HTML attributes, user-defined) and the option 'hidden', to allow the element to not
be immediately displayed.

Internally, each WUIDom object represents an HTML element (allowing API-level access to its child
elements and to its own properties). The job of the assign method is to tell the WUIDom element
which HTML element it is representing. **NOTE** This method is called automatically by createElement
and createChild.

Calling the method:

```javascript
newElement.assign(tagName, options)
```

Example 1:

```javascript
var WUIHiddenInputElement = function (textValue) {
    WUIDom.call(this);
    this.assign('input', { hidden: true, attr: { type: 'hidden', value: textValue } });
}
inherits(WUIHiddenInputElement, WUIDom);
module.exports = WUIHiddenInputElement;
```

Example 2:

```javascript
var newElement = this.assign(new WUIHiddenInputElement('hiddenValue'));
```

In the first example, we define a control called WUIHiddenInputElement. After calling WUIDom,
we use assign to set the rootElement of WUIDom. The assign statement tells the element that it is
an 'input' and that it starts out as hidden, with type hidden and a value set by the user.

The second example demonstrates that we can use the assign function to change the rootElement of a
WUIDom object to be an existing WUIDom or HTML element. The assign method can be called more than
once on the same object, but this should be avoided for consistency's sake.

#### setText

The setText method is used to set the inner text of a WUIDom element. The initial text value of a
WUIDom can also be set using the text property of the 'options' object, as described above.

Calling the method:

```javascript
newElement.setText('someText');
```

The setText method takes a text argument and sets the innerText property of the rootElement of the
WUIDom element. It's important to note that this does **not** change the value of the rootElement,
just the display text.

**Another use of this function**

```javascript
newElement.setText(value, interval);
```

The 'value' can also be a function, in which case the function is called repeatedly every 'interval'
milliseconds, until the element is destroyed using '.destroy()' or this function is called again.

#### getText

The getText method will retrieve the current innerText property of a WUIDom element.
**Note** that this value may very well be undefined if the text property has not yet been set

Calling the method:

```javascript
var text = newElement.getText();
```

#### setStyle

The setStyle method is used to set the style properties of an already-defined WUIDom element.

```javascript
childElement.setStyle(property, value);
```

Here the method will take a single style property and set its value. No validation is done on the
parameters, so the user should be careful to not set invalid styles with this method.

**Note** These style and class method should only be called once the assign, createChild or
createElement method has been called.

#### getStyle

The getStyle method is used to check the value of specific properties of the style attribute.
The method will return the value of the specified property, if it exists. If the property does
not exist, then the method will return 'undefined'.

Calling the method:

```javascript
childElement.getStyle(property);
```

#### unSetStyle

The unSetStyle method will set a specified style to null. It is equivalent to calling
setStyle(property, null).

Calling the method:

```javascript
childElement.unsetStyle(property);
```

#### getComputedStyle

The getComputedStyle method will return the computed style (CSS style text) for a given property
of the style element. If the property is not defined, WUIDom will return 'undefined'.

Calling the method:

```javascript
childElement.getComputedStyle(property);
```
#### getClassNames

The getClassNames function will return the class names associated with the WUIDom element.
If there is more than one class name, the class names are returned as an Array of strings.
If there is only one class name, then it is returned as a string.

**Note** If there are no class names, then this method will return undefined

Calling the method:

```javascript
childElement.getClassNames();
```

#### setClassNames

The setClassNames function is used to set the class names associated with a WUIDom element.
The method takes either a string or an Array of strings and overwrites the current classes
(if any) associated with the WUIDom element.

Calling the function:

```javascript
childElement.setClassNames(arrayOfClassNames);
```

**Note** that the old class names will be overwritten by this call.

#### addClassNames

The addClassNames function works like the setClassNames function, except that existing class names
are preserved. You can use this function without worrying about specifying duplicate class names,
since WUIDom will take care of that for you.

Calling the method:

```javascript
childElement.addClassNames(arrayOfClassNames);
```

#### hasClassName

The hasClassName function takes a class name as a string, and then will check if it exists or not
in the class names associated with the WUIDom element.

Calling the method:

```javascript
var hasClassName = childElement.hasClassName(className);
```

#### delClassNames

The delClassNames method will take an Array of class names and delete them from the class names,
if they exist.

Calling the method:

```javascript
childElement.delClassNames(arrayOfClassNames);
```


#### replaceClassNames

The replaceClassNames method effectively calls addClassNames and then delClassNames,
adding an array of classNames and then deleting others. Either parameter can be empty.

Calling the method:

```javascript
childElement.replaceClassNames(arrayOfClassNamesToAdd, arrayOfClassNamesToDelete);
```

#### query

The query method will allow DOM queries to be performed using the [Element.querySelector][0]
method as a base. One **important** difference is that WUI will cache the queries by selector,
so that repeated DOM queries are avoided. The cache will remain in place until '.destroy()' is
called. Query returns the first element that is a descendant of the caller (WUIDom element) and
matches the selector, or null.

**Note** that use of this method should be avoided whenever possible, because even one DOM access
on a single-page application can be very slow and have a huge performance impact.
Please read [here][1] for more information about DOM selectors.

Calling the method:

```javascript
var foundElement = childElement.query(selector);
```

[0]: https://developer.mozilla.org/en-US/docs/Web/API/Element.querySelector
[1]: https://developer.mozilla.org/en-US/docs/Web/Guide/API/DOM/Locating_DOM_elements_using_selectors

#### queryAll

The queryAll method functions as the query method, except it returns **all** WUIDom elements that
match the specified selector. This method uses a separate cache from the query method.

**Note** that use of this method should be avoided whenever possible, because even one DOM access
on a single-page application can be very slow and have a huge performance impact. This method is
much slower than query, because it searches all descendants of childElement even after a match is
found.

Calling the method:

```javascript
var foundElementsArray = childElement.queryAll(selector);
```