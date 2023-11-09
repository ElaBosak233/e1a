---
title: helloword
date: 2023-11-09
categories:
  - CTF
  - Reverse
---

这道题非常简单，单纯是学习怎么用 Jadx，把下载下来的 [apk](https://files.buuoj.cn/files/c8f1972d61784f14dc7f86eea83a791f/6176cfc5-d473-4453-a000-29e480ace634.apk) 文件直接拖到 Jadx 里，看主类

```java
package com.example.helloword;

import android.os.Bundle;
import android.support.v7.app.ActionBarActivity;
import android.view.Menu;
import android.view.MenuItem;

/* loaded from: classes.dex */
public class MainActivity extends ActionBarActivity {
    /* JADX INFO: Access modifiers changed from: protected */
    @Override // android.support.v7.app.ActionBarActivity, android.support.v4.app.FragmentActivity, android.app.Activity
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        "flag{7631a988259a00816deda84afb29430a}".compareTo("xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx");
    }

    @Override // android.app.Activity
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.main, menu);
        return true;
    }

    @Override // android.app.Activity
    public boolean onOptionsItemSelected(MenuItem item) {
        int id = item.getItemId();
        if (id == R.id.action_settings) {
            return true;
        }
        return super.onOptionsItemSelected(item);
    }
}
```

这 Flag 就在脸上了好吧