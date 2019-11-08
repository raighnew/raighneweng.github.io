---
title: 修改Android drawableRight的大小并设置Ontouch监听
toc: true
tags: [Android, Share]
categories: Android
date: 2017-03-23 10:22:50
thumbnail: /images/drawable.png
banner: /images/drawable.png
---

最近的安卓项目中遇到了设置edittext drawableRight属性后，图像太大与edittext不协调的情况，搜索了解决方案如下。

<!--more-->
网上的一些解决方法已经被弃用了，以下代码在7.1中通过测试。

```
//首先获取到EditText对象
pwdEdit = (EditText) findViewById(R.id.PasswordEdit);

// 获取Drawable并设置
Drawable invisibledrawable = ResourcesCompat.getDrawable(getResources(), R.drawable.invisible,null);
invisibledrawable.setBounds(0,0,40,40);
pwdEdit.setCompoundDrawables(null,null,invisibledrawable,null);


pwdEdit.setOnTouchListener(new OnTouchListener() {
    private static final int DRAWABLE_RIGHT = 2;
    @Override
    public boolean onTouch(View v, MotionEvent event){
        if (pwdEdit.getCompoundDrawables()[DRAWABLE_RIGHT] == null) {
            return false;
        }
        //这里一定要对点击事件类型做一次判断，否则你的点击事件会被执行2次
        if (event.getAction()!=  MotionEvent.ACTION_UP) {
            return false;
        }
        if (event.getX() > pwdEdit.getWidth() - pwdEdit.getCompoundDrawables()[DRAWABLE_RIGHT].getBounds().width()) {
            //do something you want
            int passInputTypeCode=pwdEdit.getInputType();
            if(passInputTypeCode==129){
                pwdEdit.setInputType(145);
                Drawable invisibledrawable = ResourcesCompat.getDrawable(getResources(), R.drawable.visible,null);
                invisibledrawable.setBounds(0,0,40,40);
				pwdEdit.setCompoundDrawables(null,null,invisibledrawable,null);

            }else if(passInputTypeCode==145){
                pwdEdit.setInputType(129);
                Drawable invisibledrawable = ResourcesCompat.getDrawable(getResources(), R.drawable.invisible,null);
                invisibledrawable.setBounds(0,0,40,40);
                pwdEdit.setCompoundDrawables(null,null,invisibledrawable,null);
            };
            return true;
        }
        return false;
    }
});
```
passInputTypeCode129是不明文显示，设置为145为明文显示。

