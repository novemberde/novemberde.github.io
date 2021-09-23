---
title: Android Webview에서 Javascript에러로 인해 뷰가 안나올 경우
tags: [android, webview, javascript, error, 에러]
date: "2017-12-19T11:30:03+00:00"
aliases: ["/android/2017/12/19/Android-Webview.html"]
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---
## Summary
---
Javascript WebView로 특정 URL의 컨텐츠를 보여주는데 화면이 나오지 않았다.
현상은 배경색까지 나타나고 DOM이 뿌려지지 않는 문제였다.
Webview에서 Unexpected token의 에러를 뿜었기 때문에 쉽게 Javascript 관련 오류라는 것을 알 수 있었고
Javascript error를 무시할 수 있도록 하는 메서드를 실행하였다.

---
## Webview의 Setting에 setDomStorageEnabled(true)를 추가하기
---

기존의 코드는 다음과 같다
```java
public class MyWebView extends AppCompatActivity {

    private WebView webView;

    @Override
    protected void onCreate(Bundle savedStateInstance){
        super.onCreate(savedStateInstance);
        setContentView(R.layout.webview);

        webView = ( WebView )findViewById( R.id.webview);
        webView.getSettings().setRenderPriority(WebSettings.RenderPriority.HIGH);
        webView.setWebViewClient(new WebViewClient());
        webView.setWebChromeClient(new WebChromeClient());
        webView.setNetworkAvailable(true);
        webView.getSettings().setJavaScriptEnabled(true);
        webView.loadUrl(url);
    }
}
```

변경한 코드는 다음과 같다.
```java
public class MyWebView extends AppCompatActivity {

    private WebView webView;

    @Override
    protected void onCreate(Bundle savedStateInstance){
        super.onCreate(savedStateInstance);
        setContentView(R.layout.webview);

        webView = ( WebView )findViewById( R.id.webview);
        webView.getSettings().setRenderPriority(WebSettings.RenderPriority.HIGH);
        webView.setWebViewClient(new WebViewClient());
        webView.setWebChromeClient(new WebChromeClient());
        webView.setNetworkAvailable(true);
        webView.getSettings().setJavaScriptEnabled(true);

        //// Sets whether the DOM storage API is enabled.
        webView.getSettings().setDomStorageEnabled(true);
        ////

        webView.loadUrl(url);
    }
}
```

요즘에는 Front Web Framework에서 DOM을 관리하기 때문에 HTML이 아닌 Javascript로 DOM을 뿌려주게 된다.

그렇게 되는 경우에 Javascript에 에러가 나서 Javascript로 생성된 모든 DOM객체를 뿌려주지 않는다면 아무런 화면을 확인할 수 없다.

그렇기 때문에 DomStorage를 Enable해야 에러가 생긴 지점까지 나온 DOM 객체를 뿌려주기 때문에, 기존의 웹사이트에서 보는 것을
Webview에서도 확인할 수 있다. 

---
## References
---

- [https://developer.android.com/reference/android/webkit/WebSettings.html](https://developer.android.com/reference/android/webkit/WebSettings.html)