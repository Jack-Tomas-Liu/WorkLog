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
