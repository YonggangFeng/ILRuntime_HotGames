# ILRuntime_HotGames
基于ILRuntime的热更新能力实现的可以直接使用的框架。

AHotGames是C#热更项目。
UHotGames是Unity项目。

C#热更项目（AHotGames）请用VS2017打开。
Unity项目是用Unity2018创建的，不过2017版本以上的Unity应该也能打开，并没有用到什么高级特性，只是一堆代码，目前只有Scene/Main场景有用，其实自己创建一个空场景，随便在哪个GameObject挂上入口类Enter类就可以跑了。

使用方法很简单，ILRuntime部分已经在Unity工程中整合，除非有未实现的ILRuntime适配器需要添加，或者ILRuntime有重大更新，否则不建议修改这部分。在C#热更项目中写好功能后编译，我已经写好编译后事件，VS会直接将生成到Unity项目的dll的扩展名修改成bytes，以避免Unity将热更dll直接编译入最终的Assemble中。
Unity项目中的Enter类为起始类，可以修改Config路径为自己的远程路径。
Unity项目中的UBuildTools类为编辑器辅助类，在Unity编辑器中运行，可以打最终包，也可以打AssetBundle包。

在C#热更项目部分新加的类建议都从AGameBase继承，这样可以直接使用很多基类方法。

已经实现的几个GUI游戏中发现一些小问题，原因是Unity的一些内置资源在热更项目里面无法直接获取，之后我会再想办法，不过想必身处8012年末期的大家应该也不会再用GUI来做游戏的，所以这个坑不是很着急填。

AGameBase类不是从MonoBehaviour继承的，ILRuntime的原作者建议热更项目中尽量不要继承自MonoBehaviour，所以我也就这么做了。 Update和OnGUI这两个需要每帧执行的方法似乎比较损耗性能，各位开发时也需要慎重，当然，亲测一些小游戏是完全没必要在意这些小损耗的。

ILRuntime项目地址：
https://github.com/Ourpalm/ILRuntime
向大佬致敬，感谢大佬带来了可以让我们用C#热更功能的ILRuntime。

目前已知限制：
 - 不能使用可空类型修饰符(?)
 
	示例： int? ivalue = null;

	这种用法暂时是无法使用的，会报错。
	
############

群里的同学说不知道应该怎么用，那么这篇帖子里就将带领大家走一遍完整流程，注意，我们将只会在Unity主工程中添加资源和预设，不会修改Github主工程中一行代码，所有的代码都将在AHotGames热更项目中添加和实现。

那么好，如果大家现在还没有将Github上的项目check下来，我可以给大家一些时间将项目下载下来。

好，相信大家现在已经把项目check下来了，现在一起打开2017及以上版本的Unity，我也打开我的Unity2018，大家现在看到Unity的启动界面，需要登录的就登录，不需要登录的就显示Loading，然后出现了你之前打开过的项目的列表，点击右上角的Open按钮，打开下载下来的项目中的UHotGames项目。

这个项目中现在什么都没有，只有三个没什么卵用的Scene，以及一大堆莫名其妙的代码，所以加载起来应该是很快的，我打这些字的同时大家的项目应该就已经加载好了。

好，现在项目打开了，在Project视图中，找到Scene文件夹，双击里面的Main场景。

单击Main Camera，双击右边Inspector视图中的Enter脚本（Script右边被一个小框框住的Enter），那么现在你的VS2017将被调起，如果没有的话，请关闭本页面。

VS2017启动起来后，你可以看到Start方法里有一群排成“人”字拖（划掉）的代码，这里就是下载并调用ILRuntime来加载我们的热更代码的部分了，这里我们将永远不会修改它们，请大家截图保存现有的代码，欢迎大家的监督。

好，接下来是一个匪夷所思的操作，请大家跟我一起按一下F6或者Ctrl+B，让VS左下角显示“已启动生成”然后显示“生成成功”。别问我为什么，问我我就会告诉你。

下面回到我们可爱的黑色或者白色的Unity，我们将建立一个小小的UI界面，然后在这个小UI界面上完成一些伟大的事情。

在GameObject/UI菜单栏里，点击Text，添加一个Text，左边的Hierarchy视图将会变成下图这样：

▷Main Camera

  ▽Canvas

       Text

假装上面是一张图片吧啊，那个向下的三角还挺不好找……

把那个Canvas选中，然后拖放到的Project视图的RemoteResources里面去，这样我们就得到了一个叫Canvas的预设，如果你觉得Canvas这个预设名字不吉利，可以给他改成FirstUI这么一个好听的名字，不建议把预设名字命名为中文，因为毕竟老外的软件，有的时候真的是会出现一些神级bug。

删除掉Hierarchy视图中没用了的Canvas，保持里面只有一个Main Camera。

好了，现在我们的第一个用户界面（FirstUI）就有了，备用。

接下来到了最激动人心的写代码的环节。

双击打开AHotGames目录里的AHotGames.sln解决方案，如果这时没有弹出VS2017，你知道你需要怎么做。

打开以后，解决方案管理器视图中，最下面的utils目录中是一些辅助工具类，有需要的时候会用到，大家有什么好的工具类也可以扔这里。

上面的games目录中就是我们的游戏类了，games目录下的aempty目录是AGameBase这个基类的所在地，这个基类没有特殊情况也不需要动它，AEmptyGame类是一个模版类，我一般新加一个游戏类时，就将它复制出这个aempty目录到games目录下（选中它按住Ctrl别撒手，然后拖它到games目录上可以完成这个操作），然后把复制出来的这个类的文件名和类名改成新的游戏类的名字就可以开始写功能了，这里我们将它重命名为FirstUI（不强行要求和预设名字一致，建议一致，原因是互相好对应，出问题了好找）。

单击打开这个类，现在里面只有股孤单的那的一个InitComponents方法，没错，这里我们开天辟地的地方，接下来是最最激动人心的环节了！

        var Text = FindWidget<Text>("Text");
        Text.text = "Hello ILRuntime and FS";


将上面两行代码复制到这个方法里。

然后，最最最激动人心的环节！到！来！了！

把AEntrance类的InitComponents中的所有代码全部删掉，替换成下面这段代码：

        LoadAnotherClass("FirstUI", "FirstUI.prefab");

好，F6或者Ctrl+B，编译这个项目，左下角显示“生成成功”后，切回Unity

由于我已经加入了生成事件，VS会将生成到Unity项目的RemoteResources/Dll目录下的dll的扩展名修改为bytes，所以Unity只会将它作为文本文件load一下，并不会进行编译。

接下来，见证奇迹的时！刻！

点击运行Unity，咦，好像什么都没发生？？ 别着急，因为要去我的美国小水管云服务器下载资源列表进行比对，所以有些慢。

好了，屏幕正中央打印出了一行大字！

          此处应该有一张图  


PS：编辑器模式下会加载本地目录下的dll及预设，方便测试，打包出来用的话，需要运行MyTools/打包工具 中 “Build AssetBundles”命令，将RemoteResources目录下的所有资源打包成ab，上传至服务器，项目中的Config.txt也要上传到你自己的服务器，修改里面resources=后面指向的路径，修改Unity工程的Enter类中的ConfigURL指向服务器上的Config.txt的路径即可。
