title: Gtk Container Note
date: 2014-09-24 23:22:25
tags:
- Gtk
- Container
---

Gtk Container, Widget Node
==========================

> 你若要喜愛你自己的價值，你就得給世界創造價值 - 歌德

Gtk的Container有分成很多種，有些只能加入一個 Child ，有些能放入多個 Child(繼承自[Gtk.Bin](http://lazka.github.io/pgi-docs/#Gtk-3.0/classes/Bin.html#Gtk.Bin)) ，有些放置 Child 只能依循 Columns, Rows (繼承自[Gtk.Grid](http://lazka.github.io/pgi-docs/#Gtk-3.0/classes/Grid.html#Gtk.Grid))，而少數能夠採用絕對座標(Static的Coordinate)來放置 Child Widget 的 Container(繼承自[Gtk.Fixed](http://lazka.github.io/pgi-docs/#Gtk-3.0/classes/Fixed.html#Gtk.Fixed))，目前找到的就只有：
1. [Gtk.Layout](http://lazka.github.io/pgi-docs/#Gtk-3.0/classes/Layout.html#Gtk.Layout)
2. [Gtk.Fixed](http://lazka.github.io/pgi-docs/#Gtk-3.0/classes/Fixed.html#Gtk.Fixed)

但要注意，Layout繼承 Gtk.Scrollable 以及 Gtk.Fixed ，因此他才有絕對座標的放置法；而 Gtk.Fixed 並沒有繼承 Gtk.Scrollable ，因此如果希望內部的 Widget 能夠放超出 Container 的可視範圍(Viewport)，最好是採用Gtk.Layout，否則一旦內部的 Widget 放置的超出 Container 的範圍， Container會變大。

附上範例程式碼：

```python dragBox.py
from gi.repository import Gtk


class DragBox(Gtk.EventBox):
    def __init__(self):
        super(DragBox, self).__init__()

    def get_state(self):
        return self.state
```

```python dropArea.py
from gi.repository import Gtk
from point import Point


class DropArea(Gtk.Layout):
    def __init__(self):
        super(DropArea, self).__init__()
        self.press_point = None  # in drag-box's coordinate space
        # self.set_events(Gdk.EventMask.BUTTON_PRESS_MASK)
        # self.set_events(Gdk.EventMask.BUTTON_RELEASE_MASK)
        # self.set_events(Gdk.EventMask.POINTER_MOTION_MASK)
        # self.connect('button-press-event', self.on_button_press)
        # self.connect('button-release-event', self.on_button_release)
        # self.connect('motion-notify-event', self.on_mouse_motion)

    def add_drag_box(self, drag_box, x, y):
        self.put(drag_box, x, y)
        drag_box.connect('button-press-event', self.on_drag_box_button_press)
        drag_box.connect('button_release_event', self.on_drag_box_button_release)
        drag_box.connect('motion-notify-event', self.on_drag_box_mouse_motion)

    def on_drag_box_button_press(self, widget, event):
        # print('on drag box button press')
        self.press_point = Point(event.x, event.y)

    def on_drag_box_button_release(self, widget, event):
        # print('on drag box button release')
        pass

    def on_drag_box_mouse_motion(self, widget, event):
        (x, y) = self.get_absolute_coord(widget, event.x, event.y)
        self.move(widget, x-self.press_point.x, y-self.press_point.y)

    def get_absolute_coord(self, widget, x, y):
        allocation = widget.get_allocation()
        return x+allocation.x, y+allocation.y

    ##########################################################
    #    This method is deprecated unless you want to        #
    #    forbidden widget from moving out of the container   #
    ##########################################################
    def out_range_adjust(self, widget, x, y):
        allocation = widget.get_allocation()
        self_allocation = self.get_allocation()
        if x < 0:
            x = 0
        elif x+allocation.width > self_allocation.width:
            x = self_allocation.width - allocation.width
        if y < 0:
            y = 0
        elif y+allocation.height > self_allocation.height:
            y = self_allocation.height - allocation.height
        return x, y

```

```python testDropArea.py
import dragBox
import dropArea
from gi.repository import Gtk
from gi.repository import Gdk


class TestWindow(Gtk.Window):
    def __init__(self):
        super(TestWindow, self).__init__(title="TestWindow")
        self.drop_area = dropArea.DropArea()
        self.drop_area.set_size(450, 450)
        self.drop_area.set_size_request(450, 450)
        self.drop_area.override_background_color(Gtk.StateType.NORMAL, Gdk.RGBA(0, 0.3, 0.5, 1))

        self.fixed = Gtk.Fixed()
        self.fixed.put(self.drop_area, 100, 100)
        self.add(self.fixed)

        drag_box = dragBox.DragBox()
        drag_box.set_size_request(200, 200)
        drag_box.override_background_color(Gtk.StateType.NORMAL, Gdk.RGBA(1, 0, 0.5))
        self.drop_area.add_drag_box(drag_box, 100, 20)

test_window = TestWindow()
test_window.set_size_request(640, 640)
test_window.connect('delete-event', Gtk.main_quit)
test_window.show_all()
Gtk.main()
```
