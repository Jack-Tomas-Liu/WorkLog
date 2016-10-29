1 微信签名和新浪微博等签名工具，是通过包名找到应用。当应用用同一个签名文件签名时，即使更改报名也不会导致通过上述工具取得的签名串的改变。

2 Android studio在build variants面板改成release时，debug功能是不能使用的。当然可以在build.gradle 文件中设置。

3 Android studio 更改build.gradle可以做到开发工程的包名和打包后对外暴露的包名不同。
例如开发包名是com.shengjing360

```
defaultConfig {
        applicationId "com.sjzhvip"
        minSdkVersion 18
        targetSdkVersion 22
        versionCode 1
        versionName "0.3"
    }
```
实际安装后的包名是com.sjzhvip
但是需要特别注意工程中整体的包可以不改。但是跟微信回调相关的必需建立新包com.sjzhvip。放置相关的文件WXEntryActivity等。


4 微信存在的缓存问题可能导致的问题

* 更换签名后无法调起支付界面
* 分享来源的文本和icon不能及时更新
解决方式，清除缓存甚至删除应用。重新安装
=============================
1 如何将自己的代码库共享，让其它的开发者通过 ```compile ****``` 就可以获取到。[链接地址。](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0623/3097.html)

2 listview和scrollview 会出现ScrollView无法定位到顶部的现象，修复方式：
                    scrollView.post(new Runnable() {
                        public void run() {
                            scrollView.fullScroll(ScrollView.FOCUS_UP);
                        }
                    });

================================
1 RecycleView

* 在只有一种item的情况下，缓存的ViewHolder的数目为RecyclerView在滑动过程中所能在一屏内容纳的最大item个数+2。(单布局，全屏最多的item数＋2）
* 而有至少两种item显示的情况下，每种item的ViewHolder的缓存个数为单种item在一屏内最大显示个数+1（一种以上的布局，全屏显示item最多的布局所显示出的item数＋1）


2 textview过长，显示省略。。

            android:singleLine="true"
            android:ellipsize="end"
            android:maxEms="7"

==================================

gradle 在命令行中执行
**1 配置gradle**

Android Studio下能自动执行gradle。如果需要在Command Lien中之行，需要配置环境变量。过程如下：

第一步：

找到finder下的路径。默认会下载到/Users/liuxin/.gradle/wrapper/dists/。例如我的目录是/Users/liuxin/.gradle/wrapper/dists/gradle-2.1-all/488seql5pimt7vjvdsuqhh1ut/gradle-2.1

第二步：

在~.bash_profile文件中添加

	export GRADLE_HOME=/Users/liuxin/.gradle/wrapper/dists/gradle-2.1-all/	488seql5pimt7vjvdsuqhh1ut/gradle-2.1

	export PATH=${PATH}:${GRADLE_HOME}/bin
e.g  当时错误的配置`	export PATH=${PATH}:GRADLE_HOME/bin`

第三步：

source .bash_profile 使配置生效。

`gradle` 生效。


**2 **

            android:singleLine="true"
            android:ellipsize="end"
            android:maxEms="7"
