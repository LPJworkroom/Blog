更新：为github desktop设置代理加速请见[https://www.jianshu.com/p/21e8b4f438d3](https://www.jianshu.com/p/21e8b4f438d3)


就算是从未合作开发过的人，也都听说过github的名气，但大多无法理解它真正的优点；我也曾是其中之一。直到最近和同学合作开发一个项目，才发现它有多方便。

 多人合作一个项目，一定要把代码放到一个地方，并随时同步自己本地的和公共的代码。我们也曾这么做，租用一台服务器，修改完本地的代码就复制到服务器上，而开始编写前把服务器上的复制到本地。然而这样做有以下缺点：

1.麻烦。就算用ftp软件同步文件，选择文件并复制也很繁琐。
2.缺乏更新信息的交流。对代码进行修改，却不说明这次修改了哪些部分，没有更新日志，这将造成开发者对项目的严重误读，而传统的更新日志方式往往只是在社交软件进行简单的口头说明，不够严谨、丰富。
3.没有版本还原机制。假如一个人不幸地把错误的文件复制到服务器上，在错误被发现之前，很可能被传播给更多人。
4.存在版本更新冲突。假如A和B都修改了同一文件，则同步代码至服务器后，服务器上只会有最后同步者的更新，两人对此却浑然不觉。这实际上也是缺少更新日志造成的。

git本身只有控制台版本，很难用。github desktop（以下简称desktop）是git的图形化界面版本，正好解决了上述问题：

1.项目在本地更新后，可一键同步到代码仓库，也可一键将代码仓库同步到本地。
2.更改文件，或者增加删除文件都可以(实际上是必须)附上更新说明，既有简述又有详细说明。还会保存整个项目的更新日志。
3.会保存文件的过去版本，因此可撤销自己的，甚至是他人的改动，减轻了发生错误的影响。
4.可一键检查公共仓库有无更新，这样就不会出现修改过期版本文件的情况。
5.修改过本地文件后，本地会自动检测哪些文件被修改，可以直接对这些文件分别作更新说明，让更新日志更简洁。

对于简单的开发，这些功能已经足够。接下来我将介绍通过desktop使用git的方法。

首先需要在github网页版上操作。
创建一个代码仓库，不在此赘述。
为了允许合作者修改你的代码仓库，需要先邀请他人。点击settings/collaborators，输入合作者的昵称，点击add collaborators，便会向他发出邀请。待他在github网页版上接受，便有权限修改你的仓库了。
![](https://upload-images.jianshu.io/upload_images/14546900-2d859d44afe3eaa8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/14546900-91c5288293e64861.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





接下来便可以使用desktop。
安装好desktop，登录。点击菜单栏file/clone a repository，填入代码仓库的URL或 仓库创建者的昵称/仓库名称（如，Alice/Something），输入你希望保存的路径，便可将代码仓库保存到路径下。
![](https://upload-images.jianshu.io/upload_images/14546900-729809ad11094b43.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
要修改项目，直接修改在本地的代码仓库内的文件。修改完成后，打开desktop，软件应已经检测到修改的文件，并会列出有改动的以及删除的，添加的文件。确定修改无误，在左下方输入对这次修改的总结和详细介绍。尽管可以写的简单，但这对整个项目是至关重要的，一定要好好写。
![](https://upload-images.jianshu.io/upload_images/14546900-17c911bfd2866fd3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
完成描述后，点击commit to master。此时改动还没有同步到代码仓库。再点击菜单栏repository/push，即可完成同步。

若要检测代码仓库是否有变化，点击fetch origin，检测后便会显示。点击菜单栏repository/pull，即可把当前的代码仓库同步到本地。
![](https://upload-images.jianshu.io/upload_images/14546900-0c120be714fbbe8e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
要查看代码仓库的更新日志，点击history选项卡，便会列出本仓库所有改动。
还可以在一个改动上右击，revert this commit，便会撤销这次改动。当然，我们希望合作者间沟通顺畅，尽量少的使用撤销功能。
![](https://upload-images.jianshu.io/upload_images/14546900-cc808372d8078e80.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


desktop的基本使用方法至此结束。软件是绝对的好软件，祝各位使用顺利。

