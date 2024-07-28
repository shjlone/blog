---
title: BuildContext
toc: true
tags: Flutter
---


### BuildContext

BuildContext就是Widget对应的Element。


WidgetsFlutterBinding，其在创建的时候绑定了





如果Widget有更新，需要重新布局，Framework会将需要布局的RenderObject加入PipelineOwner的_nodesNeedingLayout中，然后当下一个VSync信号来临时，Framework会遍历_nodesNeedingLayout，对其中的每一个RenderObject重新进行布局，遍历_nodesNeedingLayout的函数。

