---
title: java和android中的回调方法
date: 2018-03-18 21:52:07
tags: java
categories: 
- 基础
---
最近看第一行代码，遇到了回调这个问题，尽管书中其他知识很细致，却没有对这个作过多解释，看来我的java没学好。这里作一个学习总结。
学习主要参考这篇[文章](http://blog.csdn.net/u012441545/article/details/52259466)。
回调用语言直接描述就是A类中调用B类中的方法b，B类中反过来调用A类中的方法a，方法a就叫做回调方法。
```
Class A实现接口CallBack callback——背景1
class A中包含一个class B的引用b ——背景2
class B有一个参数为callback的方法f(CallBack callback) ——背景3
A的对象a调用B的方法 f(CallBack callback) ——A类调用B类的某个方法 C
然后b就可以在f(CallBack callback)方法中调用A的方法 ——B类调用A类的某个方法D
```
一个很好的例子
```
有一天小王遇到一个很难的问题，问题是“1 + 1 = ?”，就打电话问小李，小李一下子也不知道，就跟小王说，等我办完手上的事情，就去想想答案，小王也不会傻傻的拿着电话去等小李的答案吧，于是小王就对小李说，我还要去逛街，你知道了答案就打我电话告诉我，于是挂了电话，自己办自己的事情，过了一个小时，小李打了小王的电话，告诉他答案是2
```
### 异步回调
```
public interface CallBack {  
    /** 
     * 这个是小李知道答案时要调用的函数告诉小王，也就是回调函数 
     * @param result 是答案 
     */  
    public void solve(String result);  
}  
```
```
/** 
 * 这个是小王 
 * @author xiaanming 
 * 实现了一个回调接口CallBack，相当于----->背景一 
 */  
public class Wang implements CallBack {  
    /** 
     * 小李对象的引用 
     * 相当于----->背景二 
     */  
    private Li li;   
  
    /** 
     * 小王的构造方法，持有小李的引用 
     * @param li 
     */  
    public Wang(Li li){  
        this.li = li;  
    }  
      
    /** 
     * 小王通过这个方法去问小李的问题 
     * @param question  就是小王要问的问题,1 + 1 = ? 
     */  
    public void askQuestion(final String question){  
        //这里用一个线程就是异步，  
        new Thread(new Runnable() {  
            @Override  
            public void run() {  
                /** 
                 * 小王调用小李中的方法，在这里注册回调接口 
                 * 这就相当于A类调用B的方法C 
                 */  
                li.executeMessage(Wang.this, question);   
            }  
        }).start();  
          
        //小网问完问题挂掉电话就去干其他的事情了，诳街去了  
        play();  
    }  
  
    public void play(){  
        System.out.println("我要逛街去了");  
    }  
  
    /** 
     * 小李知道答案后调用此方法告诉小王，就是所谓的小王的回调方法 
     */  
    @Override  
    public void solve(String result) {  
        System.out.println("小李告诉小王的答案是--->" + result);  
    }  
      
}  
```
```
/** 
 * 这个就是小李啦 
 * @author xiaanming 
 * 
 */  
public class Li {  
    /** 
     * 相当于B类有参数为CallBack callBack的f()---->背景三 
     * @param callBack   
     * @param question  小王问的问题 
     */  
    public void executeMessage(CallBack callBack, String question){  
        System.out.println("小王问的问题--->" + question);  
          
        //模拟小李办自己的事情需要很长时间  
        for(int i=0; i<10000;i++){  
              
        }  
          
        /** 
         * 小李办完自己的事情之后想到了答案是2 
         */  
        String result = "答案是2";  
          
        /** 
         * 于是就打电话告诉小王，调用小王中的方法 
         * 这就相当于B类反过来调用A的方法D 
         */  
        callBack.solve(result);   
  
          
          
    }  
      
}  

```
```

/** 
 * 测试类 
 * @author xiaanming 
 * 
 */  
public class Test {  
    public static void main(String[]args){  
        /** 
         * new 一个小李 
         */  
        Li li = new Li();  
  
        /** 
         * new 一个小王 
         */  
        Wang wang = new Wang(li);  
          
        /** 
         * 小王问小李问题 
         */  
        wang.askQuestion("1 + 1 = ?");  
    }  
}  
```
### 同步回调
安卓的button点击事件，onclick就是继承于View类的一个回调方法。
```
//这个是View的一个回调接口  
/** 
 * Interface definition for a callback to be invoked when a view is clicked. 
 */  
public interface OnClickListener {  
    /** 
     * Called when a view has been clicked. 
     * 
     * @param v The view that was clicked. 
     */  
    void onClick(View v);  
}  
```
```
package com.example.demoactivity;  
  
import android.app.Activity;  
import android.os.Bundle;  
import android.view.View;  
import android.view.View.OnClickListener;  
import android.widget.Button;  
import android.widget.Toast;  
  
/** 
 * 这个就相当于Class A 
 * @author xiaanming 
 * 实现了 OnClickListener接口---->背景一 
 */  
public class MainActivity extends Activity implements OnClickListener{  
    /** 
     * Class A 包含Class B的引用----->背景二 
     */  
    private Button button;  
  
    @Override  
    public void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
        button = (Button)findViewById(R.id.button1);  
          
        /** 
         * Class A 调用View的方法,而Button extends View----->A类调用B类的某个方法 C 
         */  
        button.setOnClickListener(this);  
    }  
  
    /** 
     * 用户点击Button时调用的回调函数，你可以做你要做的事 
     * 这里我做的是用Toast提示OnClick 
     */  
    @Override  
    public void onClick(View v) {  
        Toast.makeText(getApplication(), "OnClick", Toast.LENGTH_LONG).show();  
    }  
  
}  
```
View类文档如下:
```
/** 
 * 这个View就相当于B类 
 * @author xiaanming 
 * 
 */  
public class View implements Drawable.Callback, KeyEvent.Callback, AccessibilityEventSource {  
    /** 
     * Listener used to dispatch click events. 
     * This field should be made private, so it is hidden from the SDK. 
     * {@hide} 
     */  
    protected OnClickListener mOnClickListener;  
      
    /** 
     * setOnClickListener()的参数是OnClickListener接口------>背景三 
     * Register a callback to be invoked when this view is clicked. If this view is not 
     * clickable, it becomes clickable. 
     * 
     * @param l The callback that will run 
     * 
     * @see #setClickable(boolean) 
     */  
      
    public void setOnClickListener(OnClickListener l) {  
        if (!isClickable()) {  
            setClickable(true);  
        }  
        mOnClickListener = l;  
    }  
      
      
    /** 
     * Call this view's OnClickListener, if it is defined. 
     * 
     * @return True there was an assigned OnClickListener that was called, false 
     *         otherwise is returned. 
     */  
    public boolean performClick() {  
        sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);  
  
        if (mOnClickListener != null) {  
            playSoundEffect(SoundEffectConstants.CLICK);  
              
            //这个不就是相当于B类调用A类的某个方法D，这个D就是所谓的回调方法咯  
            mOnClickListener.onClick(this);  
            return true;  
        }  
  
        return false;  
    }  
}  
```

