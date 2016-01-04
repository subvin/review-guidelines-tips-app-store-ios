# ios-app-store-review-guidelines-tips

Tips:
1.个人App Store的审核似乎没有企业App Store账号审核严格。

2.上传app store时，提示兼容64位失败。   
  奇怪的是，虽然上传app store时失败并提示兼容64位设备失败，但是在真机调试运行时，无论是64位的iphone 6还是32位的iphone 5设备，都能正常运行，我也不知道什么原因。   
  后来修改了配置中的other linker Flags 中的参数，把 -all load ,改成 -all load -lz 却能成功上传app store。   
  虽然问题解决了，但是感觉还是瞎蒙的，上网找了下资料，有对上述参数的描述，顺便贴出来，方便大家了解。
  
  下面逐个介绍几个常用参数：
－ObjC：加了这个参数后，链接器就会把静态库中所有的Objective-C类和分类都加载到最后的可执行文件中

－all_load :会让链接器把所有找到的目标文件都加载到可执行文件中，但是千万不要随便使用这个参数！假如你使用了不止一个静态库文件，
然后又使用了这个参数，那么你很有可能会遇到ld: duplicate symbol错误，因为不同的库文件里面可能会有相同的目标文件，所以建议在遇到-ObjC失效的情况下使用-force_load参数。

-force_load：所做的事情跟-all_load其实是一样的，但是-force_load需要指定要进行全部加载的库文件的路径，这样的话，你就只是完全加载了一个库文件，不影响其余库文件的按需加载   

##### 3.iOS 应用被拒的的原因之 PLA 3.3.12 。

   在审核报告里面说的很清楚，我们可以用strings工具或者otool工具对我们的项目进行检测看有没有class: ASIdentifierManager、selector: advertisingIdentifier framework: AdSupport.framework，首先把在项目里面搜索这几个关键字，可惜，没搜到，唯一的可能就是在静态库里面，用strings工具，打开终端，cd 到 静态库所在的文件夹下 strings LangQin/libMobClickLibrary.a | grep advertisingIdentifier 或者子文件夹下，grep  后面是所要在静态库里面查找的字符串，回车如果查找到，会在显示在终端的下一行命令之中，反之如果没有，则什么都不显示，就这样把静态库都试一遍发现只有友盟里面有这个东西，那说明是友盟统计的问题，通过网上调研，在友盟官方上有这么一句话“由于Appstore禁止不使用广告而采集IDFA的app上架，友盟提供IDFA版和不含IDFA版两个SDK，两个SDK在数据上并没有差异，采集IDFA是为了防止今后因为苹果可能禁止目前使用的openudid而造成的数据波动”以及发布APP Store时的说明。对于两种SDK使用的说明，如果使用前一种SDK，发布时要选上1.serve advertisements within the app
服务应用中的广告。如果你的应用中集成了广告的时候，你需要勾选一下三项。
√2.Attribute this app installation to a previously served advertisement.
跟踪广告带来的安装。√3.Attribute an action taken within this app to a previously served advertisement跟踪广告带来的用户的后续行为。√4.Limit Ad Tracking setting in iOS
这一项下的内容其实就是对你的应用使用idfa的目的做下确认，只要你选择了采集idfa，那么这一项都是需要勾选的。
最后还有一句话“如果您仍因为采集IDFA被Appstore审核拒绝，建议您集成任意一家广告或选用友盟无IDFA版SDK”，我最后使用了无IDFA版SDK，





