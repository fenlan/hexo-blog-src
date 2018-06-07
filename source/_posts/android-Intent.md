---
title: Intent对象使用
tags: Android Intent
date: 2016-06-17 15:18:23
categories: Android
---

### 使用Intent对象专递数据

1. 在Activity之间可以通过Intent对象传递数据
2. 使用putExtra()系列方法向Intent对象当中储存数据
3. 使用getXXXExtra()系列方法从Intent对象当中取出数据

<!--more-->

### 来个实例吧

**这是第一个Activity代码**

``` bash
package com.example.administrator.activitytest;

import android.content.DialogInterface;
import android.content.Intent;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;

public class MainActivity extends AppCompatActivity {

    private Button button;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        button = (Button) findViewById(R.id.button);
        button.setOnClickListener(new ButtonListener());
    }

    class ButtonListener implements View.OnClickListener{
        @Override
        public void onClick(View v){
            Intent intent = new Intent();
            intent.setClass(MainActivity.this,Main2Activity.class);
            EditText editText = (EditText)findViewById(R.id.editText);
            String name = editText.getText().toString();
            intent.putExtra("com.example.administrator.activitytest.Name",name);
            startActivity(intent);
        }
    }
}
```

### 这是第二个Activity代码
``` bash
package com.example.administrator.activitytest;

import android.content.Intent;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.widget.TextView;

public class Main2Activity extends MainActivity {

    private TextView textView;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main2);

        Intent intent = getIntent();
        String name = intent.getStringExtra("com.example.administrator.activitytest.Name");
        textView = (TextView)findViewById(R.id.textView);
        textView.setText(name);
    }
}

```