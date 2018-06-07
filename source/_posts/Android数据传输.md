---
title: Android在不同Activity之间传输数据
tags: Android数据传输
date: 2016-06-30 15:18:23
categories: Android
---
### Android中的数据传输

<!--more-->
**在Android中有几种层次的数据传输，首先说在同一Activity中的不同线程里面传输使用Handler来实现，具体怎么实现，在我的博客里有。还有就是在不同的Activity中进行数据传输，下面先放代码**

**MainActivity中的代码**

``` java
package com.example.administrator.myapplication;

import android.content.Intent;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.EditText;

public class MainActivity extends AppCompatActivity {

    private EditText editText;
    public final static String MESSAGE = "com.example";
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    public void sendMessage(View view){
        Intent intent = new Intent(this,Main2Activity.class);
        editText = (EditText)findViewById(R.id.EditTextId);
        String message = editText.getText().toString();
        intent.putExtra(MESSAGE,message);
        startActivity(intent);
    }
}

```

**首先要定义一个字符标记**
`public final static String MESSAGE = "com.example";`

**然后再xml文件中编写Button设置**
``` xml
<Button
        android:id="@+id/ButtonId"
        android:layout_width="wrap_content"
        android:onClick="sendMessage"
        android:text="send"
        style="@style/Widget.AppCompat.Button.Colored"
        android:layout_height="wrap_content"
        android:layout_below="@+id/EditTextId"
        android:layout_alignEnd="@+id/EditTextId"
        android:layout_marginTop="52dp" />
```

**其中只要看这个设置**
`android:onClick="sendMessage"`

**该设置是为Button设置监听器，然后将行为导向sendMessage方法**
``` java
 public void sendMessage(View view){
        Intent intent = new Intent(this,Main2Activity.class);
        editText = (EditText)findViewById(R.id.EditTextId);
        String message = editText.getText().toString();
        intent.putExtra(MESSAGE,message);
        startActivity(intent);
    }
```

**在sendMessage方法里面只要是定义意图对象Intent来导向另外一个Activity，最后startActivity(intent)启动，其中用于将数据传输的介质是Extra-Message，主要方法是intent.putExtra(),然后在另一个Activity中得到该数据，使用方法**
` String message = intent.getStringExtra(MainActivity.MESSAGE);`

### 另一个Activity中的代码
``` java
package com.example.administrator.myapplication;

import android.content.Intent;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.widget.EditText;
import android.widget.TextView;

public class Main2Activity extends AppCompatActivity {

    private TextView textView;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main2);

        textView = (TextView)findViewById(R.id.TextView2Id);
        Intent intent = getIntent();
        String message = intent.getStringExtra(MainActivity.MESSAGE);
        textView.setText(message);
    }
}

```

**同时要留意这行代码**
`Intent intent = getIntent();`

**具体怎么个传输步骤还得多看代码，一直看，看到懂为止**