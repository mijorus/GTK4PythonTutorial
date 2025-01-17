# Taiko's GTK4 Python tutorial

Wanna make apps for Linux but not sure how to start with GTK? This guide will hopefully help!
The intent is to show you how to do some common things with basic code examples so that you can get up and running making your own GTK app quickly.

Ultimately you want to be able to refer to the official documentation [here](https://docs.gtk.org/gtk4/) yourself. But I find it can be hard getting started
without an initial understanding of how to do things. The code examples here should hopefully help.

How to use this tutorial: You can either follow along or just use it to refer to specific examples.

Prerequisites: You have learnt the basics of Python. Ideally have some idea of how classes work. You will also need the following packages installed on your system: GTK4, PyGObject and Libadwaita.

Topics covered:

 - A basic GTK window
 - Widgets: Button, check button, switch, slider
 - Layout: Box layout
 - Adding a header bar
 - Showing an open file dialog
 - Adding a menu-button with a menu
 - Custom drawing with Cairo
 - Handling mouse input
 - Setting the cursor
 - Setting dark colour theme
 - Spacing and padding
 - Custom drawing with Snapshot

For beginners, I suggest walking through each example and try to understand what each line is doing. I also recommend taking a look at the docs for each widget.

Helpful is the [GTK4 Widget Gallery](https://docs.gtk.org/gtk4/visual_index.html) which shows you all the common widgets.


## A most basic program

```python
import gi
gi.require_version('Gtk', '4.0')
from gi.repository import Gtk

def on_activate(app):
    win = Gtk.ApplicationWindow(application=app)
    win.present()

app = Gtk.Application()
app.connect('activate', on_activate)

app.run(None)

```

This should display a small blank window.

![A blank GTK window](blank.png)

This is a minimal amount of code to show a window. But we will start off with a better example:

 - Making the code into classes. 'Cause doing it functional style is a little awkward in Python.
 - Switching to **Libawaita**, since many GNOME apps now use its new styling.
 - Pass in the app arguments.
 - Give the app an application id.

Here's what we got now:

### A better structured basic GTK4 + Adwaita


```python
import sys
import gi
gi.require_version('Gtk', '4.0')
gi.require_version('Adw', '1')
from gi.repository import Gtk, Adw


class MainWindow(Gtk.ApplicationWindow):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        # Things will go here

class MyApp(Adw.Application):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.connect('activate', self.on_activate)

    def on_activate(self, app):
        self.win = MainWindow(application=app)
        self.win.present()

app = MyApp(application_id="com.example.GtkApplication")
app.run(sys.argv)

```

Soo we have an instance of an app class and a window which we extend! We run our app and it makes a window!

> **Tip:** Don't worry too much if you don't understand the `__init__(self, *args, **kwargs)` stuff for now.

> **Tip:** For a serious app, you'll need to think of your own application id. It should be the reverse of a domain or page you control. If you don't have your own domain you can do like "com.github.me.myproject".


### So! What's next?

Well, we want to add something to our window. That would likely be a ***layout*** of some sort!

Most basic layout is a [Box](https://docs.gtk.org/gtk4/class.Box.html).

Lets add a box to the window! (Where the code comment "*things will go here*" is above)

```python
self.box1 = Gtk.Box()
self.set_child(self.box1)
```

We make a new box, and attach it to the window. Simple. If you run the app now you'll see no difference, because there's nothing in the layout yet either.


## Add a button!

One of the most basic widgets is a [Button](https://docs.gtk.org/gtk4/class.Button.html). Let's make one and add it to the layout.

```python
self.button = Gtk.Button(label="Hello")
self.box1.append(self.button)
```

Now our app has a button! (The window will be small now)

But it does nothing when we click it. Let's connect it to a function! Make a new method that prints hello world, and we connect it! 

Here's our MainWindow so far:

```python
class MainWindow(Gtk.ApplicationWindow):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.box1 = Gtk.Box(orientation=Gtk.Orientation.VERTICAL)
        self.set_child(self.box1)

        self.button = Gtk.Button(label="Hello")
        self.box1.append(self.button)
        self.button.connect('clicked', self.hello)

    def hello(self, button):
        print("Hello world")
```

Cool eh?

By the way the ***Box*** layout lays out widgets in like a vertical or horizontal order. We should set the orientation of the box. See the change:

```python
self.box1 = Gtk.Box(orientation=Gtk.Orientation.VERTICAL)
```

## Set some window parameters

```python
self.set_default_size(600, 250)
self.set_title("MyApp")
```

## More boxes

You'll notice our button is stretched with the window. Let's add two boxes inside that first box we made. 

```python
self.box1 = Gtk.Box(orientation=Gtk.Orientation.HORIZONTAL)
self.box2 = Gtk.Box(orientation=Gtk.Orientation.VERTICAL)
self.box3 = Gtk.Box(orientation=Gtk.Orientation.VERTICAL)

self.button = Gtk.Button(label="Hello")
self.button.connect('clicked', self.hello)

self.set_child(self.box1)  # Horizontal box to window
self.box1.append(self.box2)  # Put vert box in that box
self.box1.append(self.box3)  # And another one, empty for now

self.box2.append(self.button) # Put button in the first of the two vertial boxes
```

Now that's more neat!

## Add a check button!

So, we know about a button, next lets add a [Checkbutton](https://docs.gtk.org/gtk4/class.CheckButton.html).

```python
    ...
    self.check = Gtk.CheckButton(label="And goodbye?")
    self.box2.append(self.check)


def hello(self, button):
    print("Hello world")
    if self.check.get_active():
        print("Goodbye world!")
        self.close()
```


![Our window so far](twoitems.png)

When we click the button, we can check the state of the checkbox!

### Extra Tip: Radio Buttons

Check buttons can be turned into radio buttons by adding them to a group. You can do it using the `.set_group` method like this:

```python
radio1 = Gtk.CheckButton(label="test")
radio2 = Gtk.CheckButton(label="test")
radio3 = Gtk.CheckButton(label="test")
radio2.set_group(radio1)
radio3.set_group(radio1)
```

You can handle the toggle signal like this:

```python
radio1.connect("toggled", self.radio_toggled)
```

When connecting a signal it's helpful to pass additional parameters like as follows. This way you can have one function handle events from multiple widgets. Just don't forget to handle 
the extra parameter in your handler function.


```python
radio1.connect("toggled", self.radio_toggled, "test")
```

(This can apply to other widgets too)


## Add a switch

For our switch, we'll want to put our switch in a ***Box***, otherwise it'll look all stretched.

```python
        ...
        self.switch_box = Gtk.Box(orientation=Gtk.Orientation.HORIZONTAL)

        self.switch = Gtk.Switch()
        self.switch.set_active(True)  # Let's default it to on
        self.switch.connect("state-set", self.switch_switched) # Lets trigger a function

        self.switch_box.append(self.switch)
        self.box2.append(self.switch_box)

    def switch_switched(self, switch, state):
        print(f"The switch has been switched {'on' if state else 'off'}")
```

Try it out!

Our switch is looking rather nondescript, so let's add a label to it!
 

## ...with a Label

A label is like a basic line of text

```python
self.label = Gtk.Label(label="A switch")
self.switch_box.append(self.label)
self.switch_box.set_spacing(5) # Add some spacing

```

It should look like this now:

![Our window including switch and label](switch.png)

The file `part1.py` is an example of the code so far.

## Adding a slider (Aka scale)

Here's an example of adding a [Scale](https://docs.gtk.org/gtk4/ctor.Scale.new.html) with a range from 0 to 10

```python
        self.slider = Gtk.Scale()
        self.slider.set_digits(0)  # Number of decimal places to use
        self.slider.set_range(0, 10)
        self.slider.set_draw_value(True)  # Show a label with current value
        self.slider.set_value(5)  # Sets the current value/position
        self.slider.connect('value-changed', self.slider_changed)
        self.box2.append(self.slider)

    def slider_changed(self, slider):
        print(int(slider.get_value()))
```

## Adding a button into the header bar

First we need to make a header bar

```python
        self.header = Gtk.HeaderBar()
        self.set_titlebar(self.header)
```

Simple.

Now add a button

```python
        self.open_button = Gtk.Button(label="Open")
        self.header.pack_start(self.open_button)
```

We already know how to connect a function to the button, so I've omitted that.

Done! But... it would look nicer with an icon rather than text.

```python
        self.open_button.set_icon_name("document-open-symbolic")
```

This will be an icon name from the icon theme. 

For some defaults you can take a look at `/usr/share/icons/Adwaita/scalable/actions`.

If you were adding a new action icon it would go in `/usr/share/icons/hicolor/scalable/actions`

> **Help! Todo!** Is this the best way? How do icons work in a development environment?

## Open file dialog

Let's make that open button actually show an open file dialog

```python
        self.open_dialog = Gtk.FileChooserNative.new(title="Choose a file",
                                                     parent=self, action=Gtk.FileChooserAction.OPEN)

        self.open_dialog.connect("response", self.open_response)
        self.open_button.connect("clicked", self.show_open_dialog)

    def show_open_dialog(self, button):
        self.open_dialog.show()

    def open_response(self, dialog, response):
        if response == Gtk.ResponseType.ACCEPT:
            file = dialog.get_file()
            filename = file.get_path()
            print(filename)  # Here you could handle opening or saving the file

```

The action type can also be **SAVE** and **SELECT_FOLDER**

If you wanted to restrict the file types shown, you could add a filter. For example:

```python
        f = Gtk.FileFilter()
        f.set_name("Image files")
        f.add_mime_type("image/jpeg")
        f.add_mime_type("image/png")
        self.open_dialog.add_filter(f)
```

## Adding a button with menu

For this there are multiple new concepts we need to introduce:

 - The [***MenuButton***](https://docs.gtk.org/gtk4/class.MenuButton.html) widget.
 - The [***Popover***](https://docs.gtk.org/gtk4/class.Popover.html), but here we will use a [***PopoverMenu***](https://docs.gtk.org/gtk4/class.PopoverMenu.html) which is built using an abstract menu model.
 - A [***Menu***](https://docs.gtk.org/gio/class.Menu.html). This is an abstract model of a menu.
 - [***Actions***](https://docs.gtk.org/gio/class.SimpleAction.html). An abstract action that can be connected to our abstract menu.

So, we click a MenuButton, which shows a Popover that was generated from a MenuModel that is composed of Actions.

First make sure `Gio` is added to the list of things we're importing from  `gi.repository`:

```python
from gi.repository import Gtk, Adw, Gio
```

```python
        # Create a new "Action"
        action = Gio.SimpleAction.new("something", None)
        action.connect("activate", self.print_something)
        self.add_action(action)  # Here the action is being added to the window, but you could add it to the
                                 # application or an "ActionGroup"

        # Create a new menu, containing that action
        menu = Gio.Menu.new()
        menu.append("Do Something", "win.something")  # Or you would do app.something if you had attached the
                                                      # action to the application

        # Create a popover
        self.popover = Gtk.PopoverMenu()  # Create a new popover menu
        self.popover.set_menu_model(menu)

        # Create a menu button
        self.hamburger = Gtk.MenuButton()
        self.hamburger.set_popover(self.popover)
        self.hamburger.set_icon_name("open-menu-symbolic")  # Give it a nice icon

        # Add menu button to the header bar
        self.header.pack_start(self.hamburger)

    def print_something(self, action, param):
        print("Something!")

```

![A basic menu in headerbar](menu1.png)

## Custom drawing area using Cairo

There are two main methods of custom drawing in GTK4, the Cairo way and the Snapshot way. Cairo provides a more high level
drawing API but uses slow software rendering. Snapshot uses a little more low level API but uses much faster hardware accelerated rendering.

To draw with Cairo we use the [***DrawingArea***](https://docs.gtk.org/gtk4/class.DrawingArea.html) widget.

```python

        self.dw = Gtk.DrawingArea()

        # Make it fill the available space (It will stretch with the window)
        self.dw.set_hexpand(True)
        self.dw.set_vexpand(True)

        # Instead, If we didn't want it to fill the available space but wanted a fixed size
        #self.dw.set_content_width(100)
        #self.dw.set_content_height(100)

        self.dw.set_draw_func(self.draw, None)
        self.box3.append(self.dw)

    def draw(self, area, c, w, h, data):
        # c is a Cairo context

        # Fill background with a colour
        c.set_source_rgb(0, 0, 0)
        c.paint()

        # Draw a line
        c.set_source_rgb(0.5, 0.0, 0.5)
        c.set_line_width(3)
        c.move_to(10, 10)
        c.line_to(w - 10, h - 10)
        c.stroke()

        # Draw a rectangle
        c.set_source_rgb(0.8, 0.8, 0.0)
        c.rectangle(20, 20, 50, 20)
        c.fill()

        # Draw some text
        c.set_source_rgb(0.1, 0.1, 0.1)
        c.select_font_face("Sans")
        c.set_font_size(13)
        c.move_to(25, 35)
        c.show_text("Test")

```

![A drawing area](draw.png)

Further resources on Cairo:

 - [PyCairo Visual Documentation](https://seriot.ch/pycairo/)

Note that Cairo uses software rendering. For accelerated rendering, Gtk Snapshot can be used (todo)

## Input handling in our drawing area

### Handling a mouse / touch event

```python
        ...
        evk = Gtk.GestureClick.new()
        evk.connect("pressed", self.dw_click)  # could be "released"
        self.dw.add_controller(evk)

        self.blobs = []

    def dw_click(self, gesture, data, x, y):
        self.blobs.append((x, y))
        self.dw.queue_draw()  # Force a redraw

    def draw(self, area, c, w, h, data):
        # c is a Cairo context

        # Fill background
        c.set_source_rgb(0, 0, 0)
        c.paint()

        c.set_source_rgb(1, 0, 1)
        for x, y in self.blobs:
            c.arc(x, y, 10, 0, 2 * 3.1415926)
            c.fill()
        ...

```

![A drawing area with purple dots where we clicked](dots.png)

Ref: [GestureClick](https://docs.gtk.org/gtk4/class.GestureClick.html)

Extra example. If we wanted to listen to other mouse button types:

```python
        ...
        evk.set_button(0)  # 0 for all buttons
    def dw_click(self,  gesture, data, x, y):
        button = gesture.get_current_button()
        print(button)
```


See also: [EventControllerMotion](https://docs.gtk.org/gtk4/class.EventControllerMotion.html). Example:

```python
        evk = Gtk.EventControllerMotion.new()
        evk.connect("motion", self.mouse_motion)
        self.add_controller(evk)
    def mouse_motion(self, motion, x, y):
        print(f"Mouse moved to {x}, {y}")
```

See also: [EventControllerKey](https://docs.gtk.org/gtk4/class.EventControllerKey.html)

```python
        evk = Gtk.EventControllerKey.new()
        evk.connect("key-pressed", self.key_press)
        self.add_controller(evk)  # add to window
    def key_press(self, event, keyval, keycode, state):
        if keyval == Gdk.KEY_q and state & Gdk.ModifierType.CONTROL_MASK:   # Add Gdk to your imports. i.e. from gi import Gdk
            self.close()
```

## Setting the cursor

We can set a cursor for a widget.

First we need to import **Gdk**, so we append it to this line like so:

```python
from gi.repository import Gtk, Adw, Gio, Gdk
```

Now setting the cursor is easy.

```python
        self.cursor_crosshair = Gdk.Cursor.new_from_name("crosshair")
        self.dw.set_cursor(self.cursor_crosshair)
```

You can find a list of common cursor names [here](https://docs.gtk.org/gdk4/ctor.Cursor.new_from_name.html).

# Setting a dark color scheme

We can use:

```python
        app = self.get_application()
        sm = app.get_style_manager()
        sm.set_color_scheme(Adw.ColorScheme.PREFER_DARK)
```

See [here](https://gnome.pages.gitlab.gnome.org/libadwaita/doc/1.0.0/styles-and-appearance.html) for more details.


# Spacing and padding

For a better look we can add spacing to our **layout**. We can also add a margin to any widget, here I've added a
margin to our **box** layout.

```python
        self.box2.set_spacing(10)
        self.box2.set_margin_top(10)
        self.box2.set_margin_bottom(10)
        self.box2.set_margin_start(10)
        self.box2.set_margin_end(10)
```

![Spacing and padding](spacing.png)

# Custom drawing with Snapshot

As mentioned in the Cairo section, Snapshot uses fast hardware accelerated drawing, but it's a little more complicated to
use. Treat this section as more of a general guide of how it works than a tutorial of how you should do things.

First, we create our own custom widget class which will implement the [***Snapshot***](https://docs.gtk.org/gtk4/class.Snapshot.html) virtual method. 
(To implement a virtual method we need to prepend `do_` to the name as it is in the docs.)

```python

class CustomDraw(Gtk.Widget):
    def __init__(self):
        super().__init__()

    def do_snapshot(self, s):
        pass
```

Then it can be added in the same way as any other widget. If we want to manually trigger a redraw we can use
the same `.queue_draw()` method call on it.

If we want the widget to have a dynamic size we can set the usual `.set_hexpand(True)`/`.set_vexpand(True)`, but if it
is to have a fixed size, you would need to implement the [**Measure**](https://docs.gtk.org/gtk4/vfunc.Widget.measure.html) virtual method.

Have a read of the [***snapshot***](https://docs.gtk.org/gtk4/class.Snapshot.html) docs. It's a little more complex, but once you know what you're doing you
could easily create your own helper functions. You can use your imagination!

Here's some examples:

### Draw a solid rectangle

Here we use:
 - [**RGBA Struct**](https://docs.gtk.org/gdk4/struct.RGBA.html)
 - [**Rect**](http://ebassi.github.io/graphene/docs/graphene-Rectangle.html)

```python
    def do_snapshot(self, s):
        colour = Gdk.RGBA()
        colour.parse("#e80e0e")
        
        rect = Graphene.Rect().__init__(10, 10, 40, 60)   # Add Graphene to your imports. i.e. from gi import Graphene

        s.append_color(colour, rect)
```

### Draw a solid rounded rectangle / circle

This is a little more complicated...

 - [***RoundedRect***](https://docs.gtk.org/gsk4/struct.RoundedRect.html)

```python
        colour = Gdk.RGBA()
        colour.parse("rgb(159, 222, 42)") # another way of parsing

        rect = Graphene.Rect().init(50, 70, 40, 40)
        
        rounded_rect = Gsk.RoundedRect()  # Add Gsk to your imports. i.e. from gi import Gsk
        rounded_rect.init_from_rect(rect, radius=20)  # A radius of 90 would make a circle
        
        s.push_rounded_clip(rounded_rect)
        s.append_color(colour, rect)
        s.pop()   # remove the clip
```

### Outline of rect / rounded rect / circle

Fairly straightforward, see [append_border](https://docs.gtk.org/gtk4/method.Snapshot.append_border.html).

### An Image

 - See [***Texture***](https://docs.gtk.org/gdk4/class.Texture.html).

```python
    texture = Gdk.Texture.new_from_filename("example.png")
    # Warning: For the purposes of demonstration ive shown this declared in our drawing function,
    #  but of course you would REALLY need to define this somewhere else so that its only called 
    # once as we don't want to reload/upload the data every draw call.
    
    # Tip: There are other functions to load image data from in memory pixel data
    
    rect = Graphene.Rect().__init__(50, 50, texture.get_width(), texture.get_height())  # See warning below
    s.append_texture(texture, rect)    

```

Warning: On a HiDPI display the logical and physical measurements may differ in scale, typically by a factor of 2. In most places
we're dealing in logical units but these methods give physical units. So... you might not want to define the size of the rectangle
by the texture.

### Text

Text is drawn using Pango layouts. Pango is quite powerful and really needs a whole tutorial on its own, but here's
a basic example of a single line of text:

```python
        colour = Gdk.RGBA()
        colour.red = 0.0    # Another way of setting colour
        colour.green = 0.0
        colour.blue = 0.0
        colour.alpha = 1.0

        font = Pango.FontDescription.new()
        font.set_family("Sans")
        font.set_size(12 * Pango.SCALE)  # todo how do we follow the window scaling factor?

        context = self.get_pango_context()
        layout = Pango.Layout(context)  # Add Pango to your imports. i.e. from gi import Pango
        layout.set_font_description(font)
        layout.set_text("Example text")
        
        point = Graphene.Point()
        point.x = 50  # starting X co-ordinate
        point.y = 50  # starting Y co-ordinate
        
        s.save()
        s.translate(point)
        s.append_layout(layout, colour)
        s.restore()
    
```

## Todo...

Text box: [Entry](https://docs.gtk.org/gtk4/class.Entry.html) 

Number changer: [SpinButton](https://docs.gtk.org/gtk4/class.SpinButton.html)

Picture.

UI from XML.

Custom Styles.




