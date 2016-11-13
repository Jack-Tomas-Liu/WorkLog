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


===================================
list复用造成的问题
I**ListView错位**

滑动ListView后数据显示错位，是由于AbListView中获取getView()和滑动操作是异步进行的，其中滑动操作在一个FlingRunnable子线程中运行。这就导致了在ListView在滑动时可能已经滑动到了第十行，但可能第二行的数据这时就被直接使用了，这就是导致数据加载错乱的根本原因。

唯一的解决方案是：

只在convertview中缓存该ChildView的layout，并且缓存数据（重新获取数据并加载也可以）；

view的缓存，避免多次findviewByID

 		`static class ViewHolder {

        TextView subContentTextView1;
        TextView subNameTextView1;
        TextView subPraisedNumTextView1;
        ImageView subPraisedImageView1;
        RelativeLayout subComment1;
        LinearLayout ll_praise_area;//扩大点击区域

    }`
findviewById一次执行了

	`if (convertView == null) {
	            viewHolder = new ViewHolder();
	            convertView = LayoutInflater.from(context).inflate(R.layout.comment_answer_item_layout,null);
	            viewHolder.subContentTextView1 = (TextView) convertView.findViewById(R.id.subComment_item_content_textView);


	            convertView.setTag(viewHolder);
	        } else {
	            viewHolder = (ViewHolder) convertView.getTag();
	        }`
赋值操作


       ` final CommentAnswerListBean.AnswerRecord record = records.get(position);
        viewHolder.subNameTextView1.setText(record.getNickname());

        viewHolder.subContentTextView1.setText(record.getContent());
        viewHolder.subPraisedNumTextView1.setText(record.getUp_count());
        if (record.is_helpup()) {
            viewHolder.subPraisedImageView1.setSelected(true);
        } else {
            viewHolder.subPraisedImageView1.setSelected(false);
        }
`
每个Item 内需要修改的view和数据缓存

    `static class ChangeNum_Status{
        public ViewHolder viewHolder;//需要修改的子view所在的行。不需要修改的，不需要重复赋值
        public CommentAnswerListBean.AnswerRecord record;
    }`

给缓存赋值

	`final ChangeNum_Status changeNum_status = new ChangeNum_Status();
	        changeNum_status.commentId = record.getId();
	        changeNum_status.viewHolder = viewHolder;
	        changeNum_status.record = record;
	        viewHolder.ll_praise_area.setTag(changeNum_status);  `  

设置点击区域

	` viewHolder.ll_praise_area.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                final ChangeNum_Status changeNum_status1 = (ChangeNum_Status)view.getTag();
                if (UserManager.checkLocalLoginStatus()){
                    Operate(changeNum_status1.viewHolder,changeNum_status1.record);
                }else{
                    ((BaseActivity)context).startLogin(new Runnable() {
                        @Override
                        public void run() {
                            Operate(viewHolder,changeNum_status.record);
                        }
                    });
                }
            }
        });   `    

接受传值

	 private void Operate(final ViewHolder viewHolder, final CommentAnswerListBean.AnswerRecord record){
	        if(!viewHolder.subPraisedImageView1.isSelected()){

	            cachePraise(viewHolder,Integer.valueOf(viewHolder.subPraisedNumTextView1.getText().toString()),record);
	            //添加赞
	            BaseManager.submitPraiseToCommentResponse(record.getId(), new BaseManager.CommentOperationCallback() {
	                @Override
	                public void onSuccess() {
	                    praiseChangeListener.changeStatus(true);
	                }

	    }
也可以在事件，比如说点击事件中传过去。这样也能规避。上边的holder模式一定要注意所设置的tag的id不能是convientview一样的。这样会报转化错误。
========================================
scrollview在子布局不确定的时候跳动

##  1 ScrollView的滑动问题
ScrollView在和ListView和GridView等需要动态加载数据导致scrollView跳动的问题。

＝＝＝＝＝＝＝＝＝＝＝＝＝＝
解决方案1 调用scrollto 跳转到顶部。

	``` java
	      scrollView.post(new Runnable() {
	         public void run() {
	                scrollView.fullScroll(ScrollView.FOCUS_UP);
	                    }
	         });

	```


解决方案2 设置属性

  ```java
  		 <ScrollView>
  		 	<RelativeLayout>
  		 			<!--在scrollview的第一个子布局中加入下面两个属性值-->
  		 			android:focusable="true"  
					android:focusableInTouchMode="true"
  		 	</RelativeLayout>
  		 </ScrollView>


  ```


  [参考1]("http://blog.csdn.net/icyfox_bupt/article/details/15026299")
  [参考2](http://blog.csdn.net/huangbiao86/article/details/7388632)
===========================
android studio 不错的插件

## 集成开发环境的插件下载

[Intellij Idea Plugin官网地址]("http://blog.csdn.net/maosidiaoxian/article/details/44992655")

## Android Studio 如何安装插件
[从上述地址下载插件]("http://blog.csdn.net/maosidiaoxian/article/details/44992655")

============================
android studio关联源码失败

<h2>AS无法关联源码</h2>

 修改 jdk.table.xml

 先找到文件
 ```find / -name jdk*.xml```,找到sourcePath，修改内容如下


 ```java
 <sourcePath>
            <root type="composite">
                <root type="simple" url="file://Users/liuxin/Library/Android/sdk/sources/android-19" />
            </root>

        </sourcePath>
 ```
[参考自]("http://blog.csdn.net/nomousewch/article/details/38496133")

===============================
向下兼容到api16所做的工作，一个是adapter转换的问题，再一个是方法不存在的解决
<h2>本周主要完成</h2>
* 配合上线修改bug
* 修复登录失败的偶发错误，这个错误是由于```retrofit cookiejar的回调写入cookie和读取cookie没有锁导致的。```。开发时偶发也跟访问用户少，接口响应速度有变化相关。
* 项目向低版本适配：
	- UI上的问题，主要集中在margin－start属性引起的低版本位置错乱。
	- 低版本调用高版本方法：

	  ```Calling new methods on older version```和U```sing inlined constants on older versions```。
	  前者主要表现在：
	  * Call requires API level 16 (current min is 15): android.view.View#setBackground
	  * Call requires API level 16 (current min is 15): android.view.ViewTreeObserver#removeOnGlobalLayoutListener
	  * Call requires API level 16 (current min is 15): android.widget.TextView#getLineSpacingMultiplier
	  * android:windowTranslucentStatus requires API level 19 (current min is 15)
	  解决方法是，判断不同的系统api，然后区别的调用方法。
	  后者主要表现在：
	  * Field requires API level 16 (current min is 15): android.view.View#SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
	  * Field requires API level 16 (current min is 15): android.view.View#SYSTEM_UI_FLAG_LAYOUT_STABLE
	  解决方法就是自己定义相同值的变量。
* 方法调用时序问题。
  listview在前期版本，想要添加head或者foot，必须在```setAdapter方法```之前。这是因为添加head或者foot，默认的适配器由listadapter转为HeaderViewListAdapter。这样，向下兼容，如果不注意调用顺序，会造成adapter转换错误。解决方法就是取巧。

  ```java
  		listview.addHeaderview(footView);
  		listview.setAdapter();
  		listview.removeHeaderView(footview)//由于不需要，所以删除。只供转化adapter类型。

  ```
==============================
dialog需要的context，Windows layoutparams的各种type，阅读源码的姿势

<h2>dialog 该用哪种Context</h2>

 正常情况下，dialog的上下文context要用当前activity或者fragment的context，如果用```getApplicationContext()```,需要做两步

 * 添加flag```dialog.getWindow().setType(WindowManager.LayoutParams.TYPE_SYSTEM_ALERT)```

 * 添加权限```<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />```

 eg:WindowManager.LayoutParams的type

 - FIRST_APPLICATION_WINDOW 应用程序窗口
 - TYPE_BASE_APPLICATION 所有程序窗口的“基地”窗口,其他应用程序窗口都显示在它上面
 - TYPE_PHONE 电话窗口。它用于电话交互（特别是呼入）。它置于所有应用程序之上，状态栏之下
 - TYPE_SYSTEM_ALERT 系统提示。它总是出现在应用程序窗口之上。
 - TYPE_KEYGUARD.锁屏窗口。
 - TYPE_TOAST 信息窗口。用于显示toast。
 - TYPE_SYSTEM_OVERLAY 系统顶层窗口。显示在其他一切内容之上。此窗口不能获得输入焦点，否则影响锁屏。
 - TYPE_PRIORITY_PHONE电话优先，当锁屏时显示。此窗口不能获得输入焦点，否则影响锁屏。
 - TYPE_SYSTEM_DIALOG 系统对话框。（例如音量调节框）。
 - TYPE_KEYGUARD_DIALOG 锁屏时显示的对话框。
 - TYPE_SYSTEM_ERROR 系统内部错误提示，显示于所有内容之上。
 - TYPE_INPUT_METHOD 内部输入法窗口，显示于普通UI之上。应用程序可重新布局以免被此窗口覆盖。
 - TYPE_INPUT_METHOD_DIALOG 内部输入法对话框，显示于当前输入法窗口之上。
 - TYPE_WALLPAPER  墙纸窗口
 - TYPE_STATUS_BAR_PANEL 状态栏的滑动面板。
 - LAST_SYSTEM_WINDOW 系统窗口结束。
 - TYPE_SEARCH_BAR 搜索栏。只能有一个搜索栏；它位于屏幕上方
 - TYPE_STATUS_BAR 状态栏。只能有一个状态栏；它位于屏幕顶端，其他窗口都位于它下方。
 - LAST_SUB_WINDOW 子窗口结束。
 - TYPE_APPLICATION_MEDIA_OVERLAY 媒体信息。显示在媒体层和程序窗口之间，需要实现透明（半透明）效果。（例如显示字幕）


<h2>源码阅读的姿势</h2>

android studio有两种方式关联源码。一种是用sdk source下的源文件。另外一种是反编译android.jar。但是由于保护的机制，它认为应用层开放者不需要知道的内部细节都不会暴露出来。这种情况下跟代码往往会到 throw new RuntimeException("Stub!");。。。。balabala。。。
这个表示：实际运行时的逻辑会由Android ROM里面相同的类代替执行。

<h2>非常不错的源码库，各种特效，技术</h2>

[不错的特效]("https://github.com/Trinea/android-open-project")

==================================
登录失败,app升级出现的序列化和混淆的冲突

##android序列化和混淆##
有个应用本地登录信息需要序列号到sp文件中保存，每个网络请求都要带着序列化的cookie去发起网络请求。正常的使用都没问题。但是当升级的时候出现登录不上的问题。
###排查原因：###
1，升级安装并没有破坏数据库、sp文件等本地资源。

2，debug包，没有混淆，就不存在这个问题。

###可能的问题猜测：###
基本可以确定就是混淆造成的这个原因。搜索了一些涉及到序列化数据的地方只有登录cookie信息的保存。更加确定这个问题的出处。搜出的资料。

###序列化与serialVersionUID###
在一个对象（Student）和字节序列相互转化中，必须需要一个标记来证明它是这个Student的对象。如果不知道就不能反序列化。当然我们一般会省略这个声明过程，jvm自动根据包名、类名、继承关系、等因子计算出一个值。即使后边的改动不会真的影响反序列化，jvm还是认为有问题，抛出InvalidClassException。
这样，反编译出来看之前的版本编译出来的versionid是多少，新版本设置versionid一样就ok。
###混淆与serialVersionUID###
确保序列化的资源不被混淆。

```java
	-keepclassmembers class * implements java.io.Serializable {
	    static final long serialVersionUID;
	    private static final java.io.ObjectStreamField[] serialPersistentFields;
	    private void writeObject(java.io.ObjectOutputStream);
	    private void readObject(java.io.ObjectInputStream);
	    java.lang.Object writeReplace();
	    java.lang.Object readResolve();
	}
```
###另外###
由于这个序列化资源可能被其他的地方引用，当时工程目录里用到到地方都在同一个包下，所以用了下面的方式。

```java
	-keep class com.sjzh.net**{*;}
```
参考：http://www.tuicool.com/articles/UzIRbmB

==============================
滑动删除menu工程集成过程中遇到的问题：无法滑动。
从好的布局好的adapter中继承出来的文件有部分问题：图片不能滑动。

试想：把好的布局里加入不同的布局。是不是只有一行的行。
就是监听的问题。
因为给item布局添加了监听事件，造成滑动无效。

==============================
Scroller中的startScroll方法。

偏移量=起始位置-结束的位置（将要滑动到的位置）。
故有：X轴上，差为正是向左，为负是向右。

	  Y轴上，差为正是向上，为负是向下。
另外

scrollTo方法移动的是控件的内容而非控件自身。

================================
android比iOS卡的原因。

第一点：
架构优先级不同。IOS touch层优先级最高，安卓library（包括屏幕响应）优先级第三高。（应用与框架之后）
第二点：
安卓的手机硬件种类太多，应用开发者根本没办法针对一种机型进行更深度的优化，IOS只有一种垂直机型，不像没有水平机型，所以IOS可以在指令级别进行深度优化，可以说安卓为了能支持更多的手机硬件没办法在这个方向上做太多事情。同时苹果开发者也不用担心自己的应用或游戏能不能跑在多个硬件平台上，他们可以针对iphone进行力所能及的深度优化
第三点：平台特性
IOS是用objective-c开发的，属于编译语言，无需虚拟机，无需解释器。安卓用JAVA搞的，一个进程一个虚拟机，每个程序运行起来都需要一个解释器边解释代码边运行，这就牺牲了很大的性能。

==================================
序列化和反序列化过程，重写readObject和writeObject方法。

第一点，为什么要重写：
一个类实现类序列化接口，那么内部的变量还是内部类都不需要继续实现接口就可以直接序列化了。但是，如果有类实现类接口，但是成员变量中有外部类，而这个类切好没有实现接口，你又无法修改这个类（这个类在jar包中），就直接重写writeObject和readObject好了。
第二点，方法访问权限是私有的，如何调用：
ObjectOutputStream使用了反射来寻找是否声明了这两个方法。因为ObjectOutputStream使用getPrivateMethod，所以这些方法不得不被声明为priate以至于供ObjectOutputStream来使用。已经测试过，换成其他的访问权限不成。
======================================
