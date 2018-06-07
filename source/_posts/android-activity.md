---
title: android生命周期函数记录
tags: Android生命周期
date: 2016-06-16
categories: Android
---

### 函数及解释如下
``` bash
onCreate    在activity对象被第一次创建时调用
onStart     在activity变得可见时调用该函数
onResume    在activity开始准备与用户交互时调用该方法
onPause     当系统即将启动另外一个activity之前调用该方法
onStop      当前activity变得不可见时调用该方法
onDestroy   当前activity被销毁之前将会调用该方法
onRestroy   当一个activity再次启动之前将会调用该方法
```


### 来张图片帮助理解

<!--more-->
![one](http://images2015.cnblogs.com/blog/342595/201511/342595-20151120164811546-1166196919.png)

### 提供java源代码

``` bash
 package com.example.activitytest;
 
 import android.app.Activity;
 import android.content.Intent;
 import android.os.Bundle;
 import android.util.Log;
 import android.view.View;
 import android.view.View.OnClickListener;
 import android.widget.Button;
 
 public class MainActivity extends Activity {
   
     private Button btn;
     private static final String TAG = "ActivityTest";
     @Override
     protected void onCreate(Bundle savedInstanceState) {
         super.onCreate(savedInstanceState);
         Log.d(TAG, "MainActivity onCreate");
         setContentView(R.layout.activity_main);
         btn = (Button)findViewById(R.id.btn);
         btn.setOnClickListener(new OnClickListener() {
             
             @Override
             public void onClick(View v) {
                    Intent intent = new Intent(MainActivity.this,SecondActivity.class);
                    startActivity(intent);
             }
         });
         
     }
     @Override
     protected void onPause() {
         Log.d(TAG, "MainActivity onPause  ");
         super.onPause();
     }
     @Override
     protected void onResume() {
         Log.d(TAG, "MainActivity onResume  ");
         super.onResume();
     }
     @Override
     protected void onStart() {
         super.onStart();
         Log.d(TAG,"MainActivity onStart  ");
     }
     @Override
     protected void onStop() {
         super.onStop();
         Log.d(TAG, "MainActivity onStop  ");
     }
     @Override
     protected void onDestroy() {
         super.onDestroy();
         Log.d(TAG, "MainActivity onDestroy  ");
     }
     @Override
     protected void onRestart() {
         super.onRestart();
         Log.d(TAG, "MainActivity onRestart  ");
     }
 }
```