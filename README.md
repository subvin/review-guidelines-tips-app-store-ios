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

##### 3.PLA 3.3.12   Advertising Identifier 审核被拒解决方法 。
  分析如下：
    由于Apple修改了审核标准，苹果禁止不使用广告而采集IDFA的APP上架，IDFA只能用于广告服务。
    "You and Your Applications (and any third party with whom you have contracted to serve advertising) may us the Advertising Identifier, and any information obtained through the use of the Advertising Identifier, only for the purpose of serving advertising. If a user resets the Advertising Identifier, then You agree not to combine, correlate, link or otherwise associate, either directly or indirectly, the prior Advertising Identifier and any derived information with the reset Advertising Identifier.We also found that your app uses the Advertising Identifier but does not include ad functionality. This does not comply with the terms of the Apple Developer Program License Agreement, as required by the App Store Review Guidelines.If your app does not serve ads, please check your code - including any third-party libraries - to remove any instances of:class: ASIdentifierManager
selector: advertisingIdentifier framework: AdSupport.framework ......

报这条错误的原因如下

1 使用了第三方的库，第三方的库根据IDFA进行跟踪用户，同时APP没有加载广告。

2 使用了第三方的库，第三方的库根据IDFA进行跟踪用户，同时加载了iAD广告。

3 同时使用了iAD+ADMOB等广告

对应的解决方法：

第一种情况解决方法：
需要把和IDFA相关的代码和接口去除，因为IDFA只可以用于广告服务。

第二种情况解决方法：
iAD不使用IDFA，具体怎么实现的，iOS内部搞的，所以要解决这个问题需要把iAD换成类似Admob一类的广告服务，或者按照第一种情况来解决，就是去除第三方中IDFA相关的代码和接口。

第三种情况解决方法:
大概比较费解，明明加了Admob等广告，为啥还是给我拒绝了呢，这种情况要看广告的加载机制，一般开发者会优先加载iAD,如果没有广告源，则加载Admob（Admob是使用了IDFA），问题就出现了在这里，审核人员一般在美国，那里是有iAD的，或者现在app的状态还没有上线，iad属于测试状态，所以iAD的广告是可以获取，这样就给审核人员一个印象：app使用了IDFA（admob中），但是只是展示了iAD的广告，没有看到其他的广告服务，他们会怀疑你使用IDFA做了其他的事情，所以拒了！！！

解决方式：1.用终端命令在项目中查找那个文件中带有advertisingIdentifier、ASIdentifierManager等字样的字符串 strings LangQin/libMobClickLibrary.a | grep advertisingIdentifier ，在友盟统计中找到了带有advertisingIdentifier标识的字符串，而我们的应用没有加载任何广告，显然属于第一种情况，对应这种情况，在友盟官方提供了两套的SDK（即有无获取IDFA版的）。

解决方式:2.另外，官方还提供另外一种方法，正确填写在Appstore上填写IDFA选项。IDFA选项有四个（汉字是对这个四个选项的说明）

1.serve advertisements within the app
服务应用中的广告。如果你的应用中集成了广告的时候，你需要勾选这一项。

√2.Attribute this app installation to a previously served advertisement.
跟踪广告带来的安装。

√3.Attribute an action taken within this app to a previously served advertisement
跟踪广告带来的用户的后续行为。

√4.Limit Ad Tracking setting in iOS
这一项下的内容其实就是对你的应用使用idfa的目的做下确认，只要你选择了采集idfa，那么这一项都是需要勾选的。

如果应用没有使用广告而采集IDFA，则2，3，4项必须选上，但官方最后还有一句话“如果您仍因为采集IDFA被Appstore审核拒绝，建议您集成任意一家广告或选用友盟无IDFA版SDK”。这么做还是有可能被拒，即使没这么做过，我最后还是替换了无IDFA版的友盟SDK。



##### 4、 后台播放模式没录视屏审核被拒之解决之道-----问题盘查之比较分析法。

  问题描述：
  
  朗琴项目迭代版本提交审核报了这样的问题 “We began the review of your app but are not able to continue because we need access to a video that demonstrates your app in use on an iOS device. Specifically, since your app utilizes the background audio mode, please include the demonstration of such background mode(s) in use on an actual iOS device within your demo video (app running in the background)”.需要我们提供带有后台播放的Video再上传上去。
    
  朗琴中性版本首版本提交审核以后报了这样的问题“Multitasking Apps may only use background services for their intended purposes: VoIP, audio playback, location, task completion, local notifications, etc.Your app declares support for audio in the UIBackgroundModes key in your Info.plist but did not include features that require persistent audio.The audio key is intended for use by applications that provide audible content to the user while in the background, such as music player or streaming audio applications. Please revise your app to provide audible content to the user while the app is in the background or remove the "audio" setting from the UIBackgroundModes key.
”

  问题分析：
    
  这两个问题，虽然说法不一样，给人的映像却是一样的问题，只是暂时说不出个所以然，令我感到奇怪的是，为什么朗琴的首版本不报这个问题，非要到迭代版本了，才说要带有后台播放的视频，既然它要，那么给它就是了，对于第二个问题，通过百度、bing\stackOverFlow等等网页式搜索法也还是没找到确切的问题所在。很苦恼，“咋办”两字一直浮现在脑海里面，想过很多，一开始我还想他不就是想证明一下能后台播放吗？索性我第一次一打开应用，就让应用播歌，它播歌时如果回到后台就可以看到后台播放了，但这种方法是个程序员都知道不好，.....。后来娄哥提醒看到字面上的意思说后台播放好像还没有配置好，如果是这个原因，找其他的项目来看看，看看他们是不是和中性版本设置的一样，如果一样并且成上架了，那就不是这个问题，反之就是，如果不是，再找其它的可能，一个一个像if()elseif那样把所有的可能都考虑进去再比较排除，总有一个是对的，由于自身能力有限，暂时就只找到了后台视频的问题了，不管怎样，如果还有其它的问题再后补吧，先录下视频，发版本验证下自己的思考，最后找找之前的应用是不是都没有录后台播放的视频。如果没有录的也上传失败了，那就是这个原因。重录视屏。
    
  问题解决：
    
  把车载项目找来了看后台配置的代码，确实不一样，车载项目，在启动成功的方法里面，设置了后他播放的代码AVAudioSession *session = [AVAudioSession sharedInstance]; [session setActive:YES error:nil];[session setCategory:AVAudioSessionCategoryPlayback error:nil];而朗琴项目一个也没有，有点欣喜若狂的感觉，又找来了蜗灯项目，蜗灯项目和朗琴一样，没有在启动成功的方法里面设置，也成功了，在找了其它的几个项目，不管设置与否，都成功了。否决了这一命题，接下来就是有没有视频的事儿了，确实，问了下测试，之前的项目都有录后台播放视屏的事，但是后来的这几个视频没录，之前有几个项目也因为没有录后台播放视频，也是审核不过，被驳回了，朗琴合并版和中性版的一样，都没有录后台播放的视频，其它被驳回的APP后面录了视屏以后才成功上传。经过一番思考，还说不定就是这样的问题，貌似也只有这个可能了，录下视频。把应用传上去了再继续修改这里所写的东西。


##### 5、 含云音乐或下载功能但无相关证书被拒。   
  被拒原因描述：   
        8.6 - Apps that include the ability to download music or video content from third party sources (e.g. YouTube, SoundCloud, Vimeo, etc) without explicit authorization from those sources will be rejected 
8.6 Details 

We found that your app allows users to download music or video content without authorization from the relevant third-party sources. 

We’ve attached screenshot(s) for your reference. 

Next Steps 

Please provide documentary evidence of your rights to allow music or video content download from third-party sources. If you do not have the requested permissions, please remove the music or video download functionality from your app.   

  解决办法：   
        利用公司服务器做一个是否显示和隐藏云音乐功能的开关，在审核期间，服务器将云音乐显示开关设置为关闭状态，即移除改云音乐功能；待app store审核通过后，再打开服务器的云音乐开关，显示云音乐按钮，此时云音乐功能恢复。   
        

##### 6、苹果根证书过期、“此证书签发者无效”、企业包打不了的问题

  问题描述：
  
  打企业包的时候，意外的报了这样的错误,Xcode attempted to locate or generate matching signing assets and failed to do so becaue of the following issues
  
  Missing IOS Distribution signing identity for XXXXXX Co.,Ltd
  
   “missing ios distribution signing identity for XXX interactive marketing planning co ltd”或“wildcard APP IDS can not be used to create in house provisioning profiles please use an explicit app id”
   
   问题解决：
   通过查找stackoverFlow 得到了这样的回答
   
  1、 Download https://developer.apple.com/certificationauthority/AppleWWDRCA.cer
  
  2、Double-click to install to Keychain.
  
  3、Then in Keychain, Select View -> "Show Expired Certificates" in Keychain app.It will list all the expired certifcates.
  
  4、Delete "Apple Worldwide Developer Relations Certificate Authority certificates" from "login" tab And also delete it from "System" tab.
  
  也就是说重新下载根证书，双击打开，打开钥匙串，点击显示按钮，选中显示过期证书，从“登录”和“系统”选中删除过期的“Apple Worldwide Developer Relations Certificate Authority certificates”证书，你会发现“此证书签发者无效”变成了“此证书有效”，不出意外，打包就没问题了。
  
  附上截图
  
  双击运行下载过来的根证书。
  
  ![image](https://raw.githubusercontent.com/Arsenals/review-guidelines-tips-app-store-ios/master/pictures/screenshot3.png)
  
  从登录和系统中找到失效的根证书"Apple Worldwide Developer Relations Certificate Authority certificates"并删除
  
  ![image](https://raw.githubusercontent.com/Arsenals/review-guidelines-tips-app-store-ios/master/pictures/screenshot2.png)
  
  点击新的下载安装好的根证书你会看见
  
  ![image](https://raw.githubusercontent.com/Arsenals/review-guidelines-tips-app-store-ios/master/pictures/screenshot4.png)
  
  ![image](https://raw.githubusercontent.com/Arsenals/review-guidelines-tips-app-store-ios/master/pictures/solve.png)

 现在，不出意外的话，打包就没问题了。

##### 7.发布ad hoc 测试包客户无法安装的问题总结：
a.确认是否添加了客户的uuid,可以进入开发者中心 (https://developer.apple.com) 查看后台是否已经添加。   
b.确认客户手机是否已经安装了该应用的app store 版本或企业版本而导致安装失败的原因。   
解决办法：卸载之前的应用即可。   


***********

2.1 - Apps that crash will be rejected
2.16 - Multitasking Apps may only use background services for their intended purposes: VoIP, audio playback, location, task completion, local notifications, etc.
2.16 Details

Your app declares support for location in the UIBackgroundModes key in your Info.plist file but does not declare any features that require persistent location. Apps that declare support for location in the UIBackgroundModes key in your Info.plist file must have features that require persistent location.

Next Steps

Please revise your app to include features that require the persistent use of real-time location updates while the app is in the background. Please also add the following battery use disclaimer in your Application Description:
"Continued use of GPS running in the background can dramatically decrease battery life."

If your app does not require persistent real-time location updates, please remove the "location" setting from the UIBackgroundModes key. You may wish to use the significant-change location service or the region monitoring location service if persistent real-time location updates are not required for your app features.

Resources

For more information, please review the Starting the Significant-Change Location Service and Monitoring Shape-Based Regions. 

2.1 Details

During review, your app crashed on iPhone running iOS 9.2.1 when we tapped on the 我 tab.

This occurred when your app was used: 
- On Wi-Fi
- On cellular network

We have attached detailed crash logs to help troubleshoot this issue.

Next Steps

Please revise your app and test it on a device to ensure that it runs as expected.

Resources

For information on how to symbolicate and read a crash log, please see Tech Note TN2151 Understanding and Analyzing iPhone OS Application Crash Reports.

If you have difficulty reproducing this issue, please try testing the workflow described in Testing Workflow with Xcode's Archive feature.

If you have code-level questions after utilizing the above resources, you may wish to consult with Apple Developer Technical Support. When the DTS engineer follows up with you, please be ready to provide:
- complete details of your rejection issue(s)
- screenshots
- steps to reproduce the issue(s)
- symbolicated crash logs - if your issue results in a crash log

If you have difficulty reproducing a reported issue, please try testing the workflow described in Technical Q&A QA1764: How to reproduce bugs reported against App Store submissions.

If you have code-level questions after utilizing the above resources, you may wish to consult with Apple Developer Technical Support. When the DTS engineer follows up with you, please be ready to provide:
- complete details of your rejection issue(s)
- screenshots
- steps to reproduce the issue(s)
- symbolicated crash logs - if your issue results in a crash log


**************
通过Application Loader上传.ipa文件到App Store发现以下错误：

```
Package Summary:
1 package(s) were not uploaded because they had problems:
	/var/folders/2b/lksn_kq118n_sttybl8s25vw0000gn/T/B16954D4-43BA-49A9-BE6E-E99F431CA82A/1092025201.itmsp - Error Messages:
		ERROR ITMS-4238: "Redundant Binary Upload. There already exists a binary upload with build version '1' for train '1.2.1'" at SoftwareAssets/PreReleaseSoftwareAsset
```

说明：以上错误是没有注意version 和 build的升级命名问题。参照以下标准：http://www.ifeegoo.com/recommended-mobile-application-version-name-management-specification.html


后台音乐播放需要出示演示视频。

##### 8.发布app store 被拒(云音乐未隐藏)
被拒理由：

2.1 - Apps that crash will be rejected
8.6 - Apps that include the ability to download music or video content from third party sources (e.g. YouTube, SoundCloud, Vimeo, etc) without explicit authorization from those sources will be rejected
8.6 Details

We found that your app still allows users to download music or video content without authorization from the relevant third-party sources.

We’ve attached screenshot(s) for your reference.

Next Steps

Please provide documentary evidence of your rights to allow music or video content download from third-party sources. If you do not have the requested permissions, please remove the music or video download functionality from your app. 


2.1 Details

During review, your app crashed on iPhone running iOS 9.3.1 when we tapped on the download button. 

This occurred when your app was used: 
- On Wi-Fi
- On cellular network

We have attached detailed crash logs to help troubleshoot this issue.

Next Steps

Please revise your app and test it on a device to ensure that it runs as expected.

Resources

For information on how to symbolicate and read a crash log, please see Tech Note TN2151 Understanding and Analyzing iPhone OS Application Crash Reports.

If you have difficulty reproducing this issue, please try testing the workflow described in Testing Workflow with Xcode's Archive feature.

If you have code-level questions after utilizing the above resources, you may wish to consult with Apple Developer Technical Support. When the DTS engineer follows up with you, please be ready to provide:
- complete details of your rejection issue(s)
- screenshots
- steps to reproduce the issue(s)
- symbolicated crash logs - if your issue results in a crash log
    
   
解决办法：   在提交app store审核期间，隐藏云音乐及所有涉及第三方播放器的控件和相关操作，例如下载等功能。
	

