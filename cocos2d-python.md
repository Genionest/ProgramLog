cocos2d python

1. hello world文档，路径：cocos_test01

```python
import cocos

class hello(cocos.layer.Layer):
    def __init__(self):
        super(hello, self).__init__()
        # 创建标签，这里就是引用pyglet
        label = cocos.text.Label('Hello',
                                 font_name = 'Times New Roman',
                                 font_size = 32,
                                 anchor_x = 'center', anchor_y = 'center')
        # 获得导演窗口的宽度和高度，是一个二元组
        width, height = cocos.director.director.get_window_size()
        # 设置标签位置
        label.position = width // 2, height // 2
        # 添加标签到HelloWorld层
        self.add(label)
        
if __name__ == "__main__":
    # 初始化导演
    cocos.director.director.init(width=640, height=480, caption="hello world")
    # 创建层
    layer = hello()
    # 创建场景，添加层进来
    main_scene = cocos.scene.Scene(layer)
    # 启动场景
    cocos.direcotr.director.run(mian_scene)
```

2. 旋转图层文档，路径：cocos_test03

   ```python
   import cocos
   from cocos.action import Repeat, Reverse, ScaleBy, RotateBy
   
   # 继承了带颜色属性的层类
   class HelloWorld(cocos.layer.ColorLayer):
       def __init__(self):
           #层调成蓝色
           super(HelloWorld, self).__init__(64, 64, 224, 255)
           label = cocos.text.Label('Hello World',
                                    font_name = 'Times New Roman',
                                    font_size = 32,
                                    anchor_x = 'center', anchor_y = 'center',)
           label.position = 320, 240
           self.add(label)
           
           # 新建一个精灵，在这里是一个人
           sprite = cocos.sprite.Sprite('icon.png')
           # 精灵锚点默认在正中间，只设置位置就好
           sprite.position = 320, 240
           # 放大三倍，添加到层，z轴设为1，比层更靠前
           sprite.scale = 3
           self.add(sprite, z=1)
           
           # 定义一个动作，即2秒内放大三倍
           scale = ScaleBy(3, duration=2)
           # 标签的动作：重复执行放大三倍缩小三倍又放大。。。
           label.do(Repeat(scale + Reverse(scale)))
           # 精灵的动作：重复执行缩小三倍放大三倍又缩小。。。
           sprite.do(Repeat(Reverse(scale) + scale))
           # 层的动作：重复执行10秒内360度旋转
           self.do(RotateBy(360, duration=10))
           
   cocos.director.director.init()
   main_scene = cocos.scene.Scene(HelloWorld())
   cocos.director.director.run(main_scene)
   ```

3. 鼠标键盘事件，路径：cocos_test04

   ```python
   import cocos
   from cocos.director import director
   import pyglet
   
   class KeyDisplay(cocos.layer.Layer):
       
       is_event_handler = True
       
       def __init__(self):
           super(Keydisplayer, self).__init__()
           
           self.text = cocos.text.Label("Keys: ", font_size=18, x=100, y=280)
           self.add(self, text)
           
       def update_text(self):
           key_names = [pyglet.window.key.symbol_string(k) for k in self.keys_pressed]
           self.text.element.text = "Keys: " + ",".join(key_names)
           
       def on_key_press(self, key, modifiers):
           # 按下按键自动触发本方法
           self.keys_pressed.add(key)
           self.update_text()
           
       def on_key_release(self, key, modifiers):
           # 松开按键自动触发本方法
           self.keys_pressed.remove(key)
           self.update_text()
           
   class MouseDisplay(cocos.layer.Layer):
       
       is_event_handler = True
       
       def __init__(self):
           super(MouseDisplayer, self).__init__()
           
           self.texgt = cocos.text.Label("Mouse @", font_size=18, x=100, y=240)
           self.add(self.text)
           
       def on_mouse_motioin(self, x, y, dx, dy):
           # dx, dy为向量，表示鼠标移动方向
           self.text.element.text = 'Mouse @ {}, {}, {}, {}'.format(x, y, dx, dy)
           
       def on_mouse_drag(self, x, y, dx, dy, buttons, modifiers):
           self.text.element.text = 'Mouse @ {}, {}, {}, {}'.format(x, y, buttons, modifiers)
           
       def on_mouse_press(self, x, y, buttons, modifiers):
           # 按下鼠标按键不仅更新鼠标位置，还改变标签的位置，这里使用director.get_virtual_coord
           self.text.element.text = 'Mouse @ {}, {}, {}, {}'.format(x, y, buttons, modifiers)
           self.text.elementx, self.text.element.y = director.get_virtual_coordinates(x, y)
           
   # 这次创建的窗口带调整大小的功能
   director.init(resizable = True)
   main_scene = cocos.scene.Scene(KeyDisplay(), MouseDisplay())
   director.run(main_scene)
   ```
