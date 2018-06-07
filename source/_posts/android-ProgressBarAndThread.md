---
title: android中ProgressBar和线程学习记录
date: 2016-06-17 16:18:23
categories: Android
tags:
  - Android
  - ProgressBar
  - 线程
---

**感觉没什么说的，就直接放代码吧**

### 代码如下

<!--more-->

```
package com.example.administrator.prograssbarandthread;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.ProgressBar;

public class MainActivity extends AppCompatActivity {

    private Button button;
    private ProgressBar progressBar;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        button = (Button)findViewById(R.id.Button);
        progressBar = (ProgressBar)findViewById(R.id.ProgressBar);
        button.setOnClickListener(new ButtonListener());
    }

    class ButtonListener implements View.OnClickListener{
        @Override
        public void onClick(View view){
            MyThread thread = new MyThread();
            thread.start();
        }
    }

    class MyThread extends Thread{
        @Override
        public void run(){
            for (int i=0;i<=100;i++){
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                progressBar.setProgress(progressBar.getProgress() + 1);
            }
        }
    }
}

```

**不过关于ProgressBar的具体使用在下面地址里有**
https://developer.android.com/reference/android/widget/ProgressBar.html