# Android_captive_portal_server
Android_captive_portal_server
这个项目用来消除安卓系统的网络叹号。
网络叹号是因为谷歌在中国无法连接。
叹号的功能是用来帮助你进行安卓的网络判定。
如你所见，它很简单，亦很重要。

PS:
1.我为什么要用github来创建呢？
真正意义上的永久有效！！！！！
因为众所周知，这是一个被gfw特意添加在白名单的一个站点，
稳定，github现在已经是这个世界上比Google更不可缺少的网站之一。
其实任何一个由个人组建的服务器都不稳定罢，所以我把它放在了github上。
高速，唯一在中国大陆可以很快打开的一个网站。
其他，还能测试你的网络通往国外网站的连通性。

使用方法:
第一种(无需root)：
①打开手机的USB调试，
②用数据线(或者网络ADB)连接手机和电脑
③打开命令提示符cmd(windows10：点左下角开始，输入cmd)
④输入  adb shell "settings put global captive_portal_https_url https://raw.githubusercontent.com/873578156/Android_captive_portal_server/master/generate_204"
⑤输入完成按enter(回车)
⑥打开飞行模式
⑦关闭飞行模式
⑧测试正常移除数据线
⑨如果还有网络叹号请重新操作

第二种(需要root)：
①下载一个终端模拟器，比如connectbot。
②进入本地终端。(connectbot里点击加号，在协议里选择local，昵称随便填写一个，然后点击连接。)
③输入 su 然后 回车 (这一步会要求你进行超级用户的授权)
④输入settings put global captive_portal_https_url https://raw.githubusercontent.com/873578156/Android_captive_portal_server/master/generate_204 然后回车
⑤打开飞行模式
⑥关闭飞行模式
⑦测试成功，叹号消失，关闭终端。

第三种(需要root)：
使用下面的小狐狸的工具。




以下引用自小狐狸的博客[https://www.noisyfox.io/android-captive-portal.html]，但是他的服务器真的很不稳定。

7.1.2
谷歌又玩我23333

自7.1.2（开始？），”captive_portal_detection_enabled”设置已被废弃，现在改为了”captive_portal_mode”选项，该选项可设置为以下3种值：

0：彻底禁用检测(Don’t attempt to detect captive portals.)
1：检测到需要登录则弹窗提醒（默认值）(When detecting a captive portal, display a notification that prompts the user to sign in.)
2：检测到需要登录则自动断开此热点并不再自动连接(When detecting a captive portal, immediately disconnect from the network and do not reconnect to that network in the future.)
叹号杀手已经更新以支持该版本。

但愿以后谷歌不要再乱改了233333

感谢 jingyu9575 的帮助 https://github.com/Noisyfox/NoExclamation/issues/2

7.1.1
从7.1.1开始，检测用的服务器地址储存格式发生了变化，改为了：
private static String getCaptivePortalServerHttpsUrl(Context context) {
    return getSetting(context, Settings.Global.CAPTIVE_PORTAL_HTTPS_URL, DEFAULT_HTTPS_URL);
}
以及
public static String getCaptivePortalServerHttpUrl(Context context) {
    return getSetting(context, Settings.Global.CAPTIVE_PORTAL_HTTP_URL, DEFAULT_HTTP_URL);
}
可以看到，系统不会自动加入”generate_204″的后缀了，这意味着url可以设计的更加灵活，同时也意味着在设置的时候需要填入完整的url：
adb shell "settings put global captive_portal_https_url https://www.noisyfox.cn/generate_204"
当然如果只有http的话，可以执行：
adb shell "settings put global captive_portal_use_https 0"
adb shell "settings put global captive_portal_http_url http://www.noisyfox.cn/generate_204"
复原方法见下文。

7.0-7.1.0
需要服务器支持https。

或者使用命令

adb shell "settings put global captive_portal_use_https 0"
禁用https即可。

恢复可用

adb shell "settings put global captive_portal_use_https 1"
或者
adb shell "settings delete global captive_portal_use_https"
5.0-6.0
升级了安卓5.0的同学们一定对网络图标上面的那个感叹号感到十分郁闷。安卓5.0引入了一种新的网络评估机制来评估网络状况，当你有网络请求时会自动选择网络连接条件最好的一个网络进行连接。该机制的代码实现如下：

enter image description here

简要来说就是，如果该网络是VPN，那么直接使用这个网络进行连接，否则调用 isCaptivePortal() 函数进行网络状况的判定，再根据判定结果决定是否选用此网络。 而罪魁祸首就是这个 isCaptivePortal() 函数，它会访问 clients3.google.com/generate_204 并根据返回结果来判断网络联通状况。正是这个google的网址被墙导致安卓没有办法评估网络，这样就导致了那个蛋碎的感叹号一直存在，以及wifi用着用着突然自动连回数据连接了。

本来我想直接把 isCaptivePortal() 函数给屏蔽掉，让他一直返回成功，但是看了下google的代码，发现这个函数是非常有用处的，为什么呢？这个函数有个非常重要的作用，那就是判断当前网络是否需要登录。

想必大家都连接过那些需要验证才能使用的wifi热点吧，当你们连接这些热点的时候，android会自动弹出提示询问你是否需要登录。而这个功能就是依靠了 isCaptivePortal() 这个函数才得以实现，具体原理如下：

enter image description here

安卓先访问 clients3.google.com/generate_204 这个网址，而这个网址如字面所说，会产生一个 http 204 返回值。204返回值的意思就是空内容。如果当前wifi是需要登录才可以连接，那么当试图访问google的服务器的时候，wifi的验证机制一定会自动跳转到一个登录页面，这个时候http请求的返回值就必然不是204了。就是通过这一机制，便可以区分当前wifi是否需要验证，不得不佩服想出这个办法的人来。

然而这就导致了如果简单的屏蔽掉这个函数的功能，那么就没有办法自动提示登录了，但是如果不屏蔽掉那么这个网址被墙掉了，因此会有一个难看的感叹号。想来想去我想到了一个曲线救国的办法，那就是我们把这个网址改成国内的网址不就可以了？我们自己搭一个服务器，来产生这个204返回值给它，问题不就迎刃而解了吗？

那么下面就给出解决方法（无需root）：
1.完全屏蔽网络检查功能，最简单快速，但是就没有办法提示wifi登录：
adb shell "settings put global captive_portal_detection_enabled 0"
2.用国内的服务器替换掉google的服务器：
adb shell "settings put global captive_portal_server noisyfox.cn"
这个服务器是我自己建的，也就是本站：http://noisyfox.cn/ 我在服务器上写了个简单的204页面，网址是 http://noisyfox.cn/generate_204 只要用这个网址替换掉google的网址，就可以正常访问并检测网络状态了。不过由于本人的服务器速度并不快，所以感叹号还是会显示一小会儿的，不过应该很快就会消失。

3.恢复默认值
对于第一条指令，恢复默认只需要执行：
adb shell "settings put global captive_portal_detection_enabled 1"
或者
adb shell "settings delete global captive_portal_detection_enabled"
第二条指令的恢复直接delete即可：
adb shell "settings delete global captive_portal_server"
如果你对本站提供的服务速度不满意，可以在文末找到网友提供的其它服务地址。

enter image description here

是不是看着很舒服呢？烦人的感叹号没有了~

经过靠谱的确认，该修改方式具有持久性，重启依旧有效，除非刷机或者清除数据。

如何建立自己的服务器
1. 对于apache服务器，如果你的服务器安装了rewrite模块，那么只需要在网站的.htaccess中加入以下代码：
<IfModule mod_rewrite.c>;
  RewriteEngine On
  RewriteCond %{REQUEST_URI} /generate_204$
  RewriteRule $ / [R=204]
</IfModule>;
2. 对于nginx，直接加入以下设置即可：
location /generate_204 { return 204; }
3. 如果以上方法都无效，那么就要利用代码中的一个小trick来完成，直接在网站的根目录下建立一个叫做“generate_204”的空文件即可，因为安卓的源码中写了如果返回的内容为空那么也会当成204（毕竟一个空的页面怎么想都不可能是登录页面嘛！）。

一键设置工具（需要root）
锵锵锵！由于有些人不太熟悉adb之类的操作，因此就做了一个小工具方便大家直接在手机上设置！


下载地址
最新版请移步：https://github.com/Noisyfox/NoExclamation/releases

NoExclamation Portal Server 修改器 1.2
Download Now!1827 Downloads
NoExclamation Portal Server 修改器 2.0
叹号杀手 2.0

支持 Android 7.1.1
1.5

修正了应用崩溃的问题
1.4

增加图标
替换网址为英文网址
优化了重置网址功能
优化界面，在修改网址时不会导致界面卡顿
Download Now!905 Downloads
一些其它服务网址
我会尽我所能提供长期有效的服务，但是由于本站服务器不是很快，而且网络状况有时候会不稳定，因此无法保证100%可靠的服务。不过有一些热心网友提供了其它服务网址，速度和稳定性或许会比本站要好。故在此特别列出供大家选用。如果给提供者带来不便之处请回复告知，我会及时删除。

by fengz: captive.v2ex.co V2EX建立的服务，速度不错，稳定性也很不错，具体信息请查看 https://www.v2ex.com/t/303889

by lkebin: liukebin.avosapps.com 架设于LeanCloud服务器，据lkebin称是永久有效

by Zohar: www.iwch.me 热心网友的个人站点

引用完毕。
