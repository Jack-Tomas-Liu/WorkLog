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

4 微信存在的缓存问题可能导致的问题

* 更换签名后无法调起支付界面
* 分享来源的文本和icon不能及时更新
解决方式，清除缓存甚至删除应用。重新安装
=============================
1 如何将自己的代码库共享，让其它的开发者通过 ```compile ****``` 就可以获取到。[链接地址。](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0623/3097.html)

=============================

2 listview和scrollview 会出现ScrollView无法定位到顶部的现象，修复方式：
                    scrollView.post(new Runnable() {
                        public void run() {
                            scrollView.fullScroll(ScrollView.FOCUS_UP);
                        }
                    });
