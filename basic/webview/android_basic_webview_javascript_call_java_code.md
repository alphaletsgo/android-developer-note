# JavaScript call Java code
Android WebView开发中经常会用到在javascript中调用java代码，比如点击webview中的图片放大，或者保存图片到本地等等。

Android的WebView组件对此提供了很完善的支持，下面对在WebView中使用javascript调用java代码做一个简单地使用教程。

### 步骤
- 在html代码中动态的加入javascript代码
- 启用webview的javascript功能
- 使用java代码创建一个类，用于在javascript代理对象
- 通过`addJavascriptInterface(Object object, String name)`方法设置代理

``` java
public static final String TAG = "MainActivity";
private WebView mWebView = null;
private static final String html = "html代码";

@SuppressLint("JavascriptInterface")
private void initView() {
    mWebView = (WebView) this.findViewById(R.id.webView);
    mWebView.getSettings().setJavaScriptEnabled(true);
    mWebView.addJavascriptInterface(new JavaScriptInterface(), "javaObject");
    mWebView.loadDataWithBaseURL("", handleImageClickEvent(html), "text/html", "UTF-8", "");
}

class JavaScriptInterface {

    @JavascriptInterface
    public void imageClick(String path) {
        Log.d(TAG, "imageClick------" + path);

    }
}

//修改html代码--加入javascript点击事件
public String handleImageClickEvent(String htmlData) {
    Document doc = Jsoup.parse(htmlData);//依赖Jsoup库
    Elements es = doc.getElementsByTag("img");

    for (Element e : es) {
        String imgUrl = e.attr("src");
        String imgName;
        File file = new File(imgUrl);
        imgName = file.getName();
        if (imgName.endsWith(".gif")) {
            e.remove();
        } else {
            String str = "window." + "javaObject" + ".imageClick('" + imgUrl + "')";
            e.attr("onclick", str);
        }
    }
    String body = doc.html();
    return body;
}

```
备注：代理类的方需要有`@JavascriptInterface`注解，否则可能会出现无法回调成功。

[更多错误参考](http://droidyue.com/blog/2014/09/20/interaction-between-java-and-javascript-in-android/)

Jsoup参考[文档](https://www.ibm.com/developerworks/cn/java/j-lo-jsouphtml/)