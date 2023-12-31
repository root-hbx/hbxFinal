# 对游戏手柄的理解

这里不考虑触屏，只考虑电脑端的键鼠和手柄连接。

| 手柄按键 | 手柄含义 | 键鼠 |
|:-:|:-:|:-:|
| A | 确认 | 鼠标左键 |
| B | 返回/取消 |  |
| X |  |  |
| Y |  |  |
| 左摇杆 | 光标移动 | 鼠标移动 |
| 左摇杆按下 |  |  |
| 右摇杆 | 上下左右 | 上下左右 |
| 右摇杆按下 |  |  |
| D-pad | 上下左右 | 上下左右 |
| 左肩 | 往左一页 |  |
| 右肩 | 往右一页 |  |
| 左扳机 |  |  |
| 右板机 |  |  |
| share | 退出 | esc |
| home |  |  |
| menu | 选项 | 鼠标右键 |

对于鼠标支持良好的游戏，手柄需要绑定如下按键：

- 左摇杆：鼠标移动
- A：鼠标左键
- menu：鼠标右键
- share：esc

基本上是用手柄模拟出了鼠标可以做到的所有操作，这对绝大部分游戏来说已经足够。比较关键的是，手柄的光标最好能够圈出物体（其实鼠标也应该做到这个效果），这样在点击 A 的时候更明确点击的物体。

在此基础上支持手柄，需要适配：

- B：返回
- 右摇杆：上下左右
- D-pad：上下左右
- 左右肩：左右翻页

剩下的手柄按键可以自定义：

- XY
- 左右摇杆按下
- 左右板机

但是需要注意，还可以有长按和双击等多种操作。每一个界面的按键操作可能还不一样。

得去进一步理解手柄的底层 IO 逻辑。最好就是学一下 Chrome 对手柄输入的处理，看 JS 如何能监听或者收取到手柄的输入。
