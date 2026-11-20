# 双指拖动如何分配控点并在取消后恢复

2026-11-20

在几何有迹的手拉手模型中，一根手指拖 A 时，C 应保持原位置。两根手指分别按住 A、C 时，两点可以同时改变。要让这两个操作持续工作，需要把每次接触与它选中的控点绑定，并在接触结束时清理绑定。

English summary — Verify separate pointer ownership, stable capture targets, and recovery after canceled touch gestures in an interactive geometry model.

浏览器用 `pointerId` 区分同一页面中的活动指针。因此，每根手指可以拥有自己的拖动状态。标识值在接触结束后可能被重复使用，保存状态时要以一次接触的生命周期为范围。[W3C Pointer Events 3](https://www.w3.org/TR/pointerevents3/#pointerevent-interface)

在当前实现中，按下时选定控点，并记录手指接触位置与控点中心之间的偏移。移动时按该指针找到对应的控点，再把屏幕坐标换算到图内坐标。这样，手指按在点的边缘，也可以保持原来的接触位置，减少图形突然跳动。已经由某根手指控制的点会从新接触的候选中排除。

捕获事件的元素也会影响拖动是否连续。图形重绘时，控点节点可能被替换。几何有迹把指针捕获放在持续存在的 SVG 画布上，再根据指针标识分派给对应控点，使接触离开起始控点后仍能继续收到移动事件。[W3C 指针捕获](https://www.w3.org/TR/pointerevents3/#setting-pointer-capture)

一次拖动有多种结束方式。正常抬起会产生 `pointerup`，浏览器中断该输入流时可能产生 `pointercancel`，失去捕获则会产生 `lostpointercapture`。当前实现收到这些结束事件后，释放对应指针的拖动状态。下一次接触才能重新选中同一个点。[W3C 取消与捕获释放](https://www.w3.org/TR/pointerevents3/#the-pointercancel-event)

本次在 390×844 的浏览器触控模拟中，运行 v1.1.0 公开 APK 的页面内容，检查了五段操作。除取消后的恢复测试外，各组均从新页面开始，两个初始控点互相分开。

| 操作 | 核验到的表现 |
| --- | --- |
| 单指拖 A | A 的位置变化，C 保持原值 |
| 两指分别拖 A、C | 产生两个不同的活动指针，两点均变化 |
| 拖 A 时接触移到画布之外 | 捕获中的移动仍更新 A，C 保持原值 |
| 拖动后由测试驱动取消接触 | 页面收到取消事件并结束本次拖动 |
| 取消后再次拖同一点 | 新接触能继续改变该点 |

页面滚动还需要单独考虑。图中控点通过 `touch-action` 声明触控行为，图外正文承担页面滚动。是否捕获指针与浏览器是否接管滚动，分别涉及事件目标和默认手势处理。[W3C 触控行为声明](https://www.w3.org/TR/pointerevents3/#the-touch-action-css-property)

这些结果验证了该页面在浏览器模拟输入下的控点分配、事件捕获与取消恢复。Android 原生 WebView、系统来电或切换应用等真实中断，还需要设备上的测试。数学上的同步运动条件可结合另一篇文章继续核对。[同步播放与独立拖动](04-dragging-and-playback.md)
