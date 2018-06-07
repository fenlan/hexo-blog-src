---
title: Android Handler学习记录
tags: Android Handler
date: 2016-06-17 17:18:23
categories: Android
---

**这个需要学习三节课程，这是第一节，多的不说，直接放代码**

<!--more-->

### 第一节课程

``` bash
package com.example.administrator.handler;

import android.os.Handler;
import android.os.Message;
import android.provider.Settings;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;

public class MainActivity extends AppCompatActivity {

    private Button button;
    private Handler handler;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        button = (Button)findViewById(R.id.Button);
        button.setOnClickListener(new ButtonListener());
		handler = new FirstHandler();
    }

    class ButtonListener implements View.OnClickListener{
        @Override
        public void onClick(View view){
            Message message = handler.obtainMessage();
            message.what = 2;
            handler.sendMessage(message);
        }
    }

    class FirstHandler extends Handler{
        @Override
        public void handleMessage(Message message){
            int what = message.what;
            System.out.println("what = " + what);
        }
    }
}
```

**附上参考地址**
https://developer.android.com/reference/android/os/Handler.html

**怎么说呢，上一节课程的代码其实我自己运行失败，主要是java基础还是没过关，不管怎样，先学完了再回头来看是怎么回事，今天来记录第二节课的学习笔记**

### 第二节课程

**主要内容**
1. 通过Handler实现线程间通信
2. 在主线程中实现Handler和handleMessage（）方法
3. 在Worker Thread当中通过Handler发送消息

**今天学得什么我也不太清楚，大概是讲一个worker线程如何通过handler来改变main线程里面的东西，多说不如直接放代码**

``` bash
package com.example.administrator.handler2;

import android.os.Handler;
import android.os.Message;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;

public class MainActivity extends AppCompatActivity {

    private Button button;
    private TextView textView;
    private Handler handler;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        button = (Button)findViewById(R.id.ButtonId);
        textView = (TextView)findViewById(R.id.TextViewId);

        handler = new MyHandler();

        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Thread thread = new NetWorkThread();
                thread.start();
            }
        });
    }

    class MyHandler extends Handler {
        @Override
        public void handleMessage(Message msg){
            String s = (String)msg.obj;
            textView.setText(s);
        }
    }

    class NetWorkThread extends Thread{
        @Override
        public void run(){
            try {
                Thread.sleep(2 * 1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            String s = "your app is working!";
            Message msg = handler.obtainMessage();
            msg.obj = s;
            handler.sendMessage(msg);
        }
    }
}

```

**这次运行成功了**

### 第三节课程

**记录之前有个小插曲，之前那个网站的视频到第二节课程就没有了，害得我到处找才找到**

**今天主要内容**

1. 准备Looper对象
2. 在workerThread当中生成Handler对象
3. 在MainThread当中发送消息

**先放代码再说问题**

``` bash
package com.example.administrator.handler3;

import android.os.Handler;
import android.os.Looper;
import android.os.Message;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;

public class MainActivity extends AppCompatActivity {

    private Button button;
    private Handler handler;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        button = (Button)findViewById(R.id.ButtonId);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Message msg = handler.obtainMessage();
                String s = "your app is working!";
                msg.obj = s;
                handler.sendMessage(msg);
            }
        });

        Thread thread = new MyThread();
        thread.start();
    }

    class MyThread extends Thread{
        public void run(){
            Looper.prepare();
            handler = new Handler(){
                public void handleMessage(Message msg) {
                    String s = (String)msg.obj;
                }
            };
            Looper.loop();
        }
    }
}

```

**写代码时我将button监听器里面的内容设置成了启动线程，然后再button后面设置handler，结果不能运行，难道是因为那样运行到另外一个线程的时候发现handler还没有设置好。或许是吧，没时间想了，已经一点了，代码发布了就睡**