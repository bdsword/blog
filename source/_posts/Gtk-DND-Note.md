title: Gtk DND Note
date: 2014-09-19 11:39:35
tags: Gtk DND
---

Gtk Drag and Drop Note
======================

> 一個誠摯、熱心，為著光明而鬥爭的人，不能夠不是刻苦而負責的 - 列夫·托爾斯泰

因為研究pygobject遇到一點障礙，所以馬上把DND的一點心得寫下來...以免以後忘記...

Gtk的Drag and Drop(DND)分成兩個角色：

- Source： 也就是DND事件發生時，被拖著移動的那個物件
- Dest： 是Drop事件發生處。

Gtk的DND事件發生過程中會觸發數個事件/訊號(Signal)：

1. Drag開始，Source獲得 __drag-begin__ 訊號，可以用來作一些Drag發生之前的設定，例如設定drag icon等等(使用 _drag_source_set_icon_ 相關函數)。
2. 拖移Source的過程中，Dest獲得 __drag-motion__ 訊號。
3. 如果拖移過程中，拖出了Dest的範圍，則Dest獲得 __drag-leave__訊號。
4. Drop發生，Dest獲得 __drag-drop__ 訊號。
此時會發生drag的data request，有點像是Dest向被拖移的Source要求一些data，才知道是什麼資料被拖移了、該如何處理等等。
5. Drag的data request，Source獲得 __drag-data-get__ 訊號。
6. Drop的data received（有可能發生另一個程式身上），如果發生 __drag-data-get__ 訊號時，callback的selection_data參數有被加入、設定資料，Dest會獲得 __drag-data-received__ 訊號。
7. Drag準備結束，Dest在drag-data-received的handler有責任呼叫Gtk.drag_finish來決定drag事件是否成功，由第三個參數決定Source是否需要刪除Data，如果是的話，Source獲得 __drag-data-delete__ 訊號。（但測試結果好像不需要手動呼叫Gtk.drag_finish，只要使用Gdk.DragAction.MOVE，就會觸發drag-data-delete訊號）
8. Drag結束，Source獲得 __drag-end__ 訊號。

介紹完了整體流程，仍然有許多細節在使用pygobject實作時得注意，以下介紹實作時的流程：

- 分別對Source Widget呼叫 __drag_source_set__ 方法，Dest Widget呼叫 __drag_dest_set__ 方法。

> __drag_source_set__ 方法讓Widget可以接受拖曳。 __drag_dest_set__ 方法則讓Widget可以接受Drag物件的放置。

>__drag_source_set(start_button_mask, targets, actions)__

>> __start_button_mask__ ：Gdk.ModifierType型態的常數，我把他看成可以觸發DND事件的按鍵。常用 Gdk.ModifierType.BUTTON1_MASK

>> __targets__ ：一個Table，裡面儲存著這個Widget support的類型（一些識別類型用的常數），可以視為此Widget是哪種類型的Drag Source，DND的dest也必須設定成能夠接受此種類型的Souce，否則Dest將無法接受該Source的Drop。

>> __actions__ ：Gdk.DragAction型態的常數，設定此Widget DND事件類型。常用Gdk.DragAction.MOVE、Gdk.DragAction.COPY

> __drag_dest_set__ 方法與上述類似，詳情參考 [Pygobjec API文件](http://lazka.github.io/pgi-docs/#Gtk-3.0/classes/Widget.html#Gtk.Widget.drag_dest_set)

> 值得一提的是，兩個方法的targets參數，可以在事後指定 - 使用 __drag_dest_set_target_list__ 方法設定targets table，或者使用 __drag_source_add_*_targets__ 方法設定擁有預設值的targets。

- 將需要處理的訊號透過connect連結到widget上面。

- 如果有需要，透過前述的方法將targets加到widget的targets list裡面。

附上測試用的程式碼（使用Python 3.4）：

{% codeblock lang:python %}
#!/usr/bin/env python3

from gi.repository import Gtk
from BDLibrary import bdshorthands
from gi.repository import Gdk

TARGET_ENTRY_PIXBUF = 10
TARGET_ENTRY_TEXT = 11

class MyWindow(Gtk.Window):
    def __init__(self):
        Gtk.Window.__init__(self, title="TestDrag")

        self.layout = Gtk.Layout()
        self.add(self.layout)

        self.image_box = Gtk.EventBox.new()
        self.image_box.add(bdshorthands.generate_image('winnie.jpg'))
        self.image_box.drag_source_set(Gdk.ModifierType.BUTTON1_MASK, [], Gdk.DragAction.MOVE)
        self.image_box.connect('drag-begin', self.on_drag_begin)
        self.image_box.connect('drag-data-delete', self.on_drag_data_delete)
        self.image_box.connect('drag-end', self.on_drag_end)
        self.image_box.connect('drag-data-get', self.on_drag_data_get)
        self.layout.put(self.image_box, 10, 10)

        self.layout.modify_bg( Gtk.StateType.NORMAL, Gdk.Color.from_floats(1.0, 0.0, 0.5) ) #修改Drop area背景顏色
        self.layout.drag_dest_set( Gtk.DestDefaults.ALL, [], Gdk.DragAction.MOVE )
        self.layout.connect('drag-drop', self.on_drag_drop)
        self.layout.connect('drag-data-received', self.on_drop_data_received)
        self.layout.connect('drag-leave', self.on_drag_leave)
        self.layout.connect('drag-motion', self.on_drag_motion)

        targets = Gtk.TargetList.new([])
        targets.add_text_targets(TARGET_ENTRY_TEXT) # 要注意target的類型是Pixbuf還是Text否則Data在填充之後，drag-data-received仍不會被觸發

        self.image_box.drag_source_set_target_list(targets)
        self.layout.drag_dest_set_target_list(targets)
        # self.image_box.drag_source_add_image_targets() # 也可以使用這個方法Append support的target，不過這樣的話target的info將會是預設的0
        # self.drag_dest_add_image_targets() # 也可以使用這個方法Append support的target，不過這樣的話target的info將會是預設的0

    def on_drag_begin(self, widget, context):
        print('On drag begin')
        widget.set_opacity(0.5)

    def on_drag_motion(self, widget, context, x, y, time):
        # print('On drag motion')
        pass

    def on_drag_drop(self, widget, context, x, y, time):
        print('On drag drop')
        # print(context.get_source_window())

    def on_drag_leave(self, widget, context, time):
        print('On drag leave')

    def on_drag_data_delete(self, widget, context):
        print('On drag data delete')

    def on_drag_end(self, widget, context):
        print('On drag end')
        widget.set_opacity(1)

    def on_drag_data_get(self, widget, drag_context, data, info, time):
        print('On drag data get!')
        data.set_text('Hello World!', -1)

    def on_drop_data_received(self, widget, drag_context, x, y, data, info, time):
        print('On drop data received!')
        self.layout.move(self.image_box, x, y)
        # delete_or_not = (drag_context.get_selected_action() == Gdk.DragAction.MOVE)
        # Gtk.drag_finish(drag_context, False, delete_or_not, time) # 實測結果不需要手動呼叫此函數

my_window = MyWindow()
my_window.connect('delete-event', Gtk.main_quit)
my_window.set_size_request(640, 480)
my_window.show_all()
Gtk.main()
{% endcodeblock%}


參考：
----

[https://wiki.gnome.org/GnomeLove/DragNDropTutorial](https://wiki.gnome.org/GnomeLove/DragNDropTutorial)
[http://www.pygtk.org/pygtk2tutorial/ch-DragAndDrop.html](http://www.pygtk.org/pygtk2tutorial/ch-DragAndDrop.html)
[http://python-gtk-3-tutorial.readthedocs.org/en/latest/drag_and_drop.html](http://python-gtk-3-tutorial.readthedocs.org/en/latest/drag_and_drop.html)
