## 0. 开发环境

- IDE：Android Studio
- JDK：1.8
- Gradle Plugin：3.5.3
- Gradle：5.4.1

## 1. AndroidManifest.xml中申请权限

AndroidManifest.xml文件中加入对应权限的静态申请，注意格式，权限申请在application节点外层

~~~xml
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
~~~

## 2. 动态权限申请

~~~java
    public void requestPermission() {
        //权限检查
        if (ContextCompat.checkSelfPermission(this,
                Manifest.permission.WRITE_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
            //申请权限
            ActivityCompat.requestPermissions(this,
                    new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE}, 1);
        }
    }
~~~

## 3. 申请权限回调

~~~java
    private AlertDialog mDialog;

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);

        if (requestCode == 1) {
            for (int i = 0; i < permissions.length; i++) {
                //已授权
                if (grantResults[i] == 0) {
                    continue;
                }

                if (ActivityCompat.shouldShowRequestPermissionRationale(this, permissions[i])) {
                    //选择禁止
                    AlertDialog.Builder builder = new AlertDialog.Builder(this);
                    builder.setTitle("授权");
                    builder.setMessage("需要允许授权才可使用");
                    builder.setPositiveButton("去允许", (dialog, id) -> {
                        if (mDialog != null && mDialog.isShowing()) {
                            mDialog.dismiss();
                        }
                        ActivityCompat.requestPermissions(MenuActivity.this,
                                new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE}, 1);
                    });
                    mDialog = builder.create();
                    mDialog.setCanceledOnTouchOutside(false);
                    mDialog.show();
                } else {
                    //选择禁止并勾选禁止后不再询问
                    AlertDialog.Builder builder = new AlertDialog.Builder(this);
                    builder.setTitle("授权");
                    builder.setMessage("需要允许授权才可使用");
                    builder.setPositiveButton("去授权", (dialog, id) -> {
                        if (mDialog != null && mDialog.isShowing()) {
                            mDialog.dismiss();
                        }
                        Intent intent = new Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS);
                        Uri uri = Uri.fromParts("package", getPackageName(), null);
                        intent.setData(uri);
                        //调起应用设置页面
                        startActivityForResult(intent, 2);
                    });
                    mDialog = builder.create();
                    mDialog.setCanceledOnTouchOutside(false);
                    mDialog.show();
                }
            }
        }
    }
~~~

## 4. 调起设置页面后回调

~~~java
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == 2) {
            //应用设置页面返回后再次判断权限
            requestPermission();
        }
    }
~~~



---

GitHub：[https://github.com/dkbnull/android-demo](https://github.com/dkbnull/android-demo)

Gitee：[https://gitee.com/dkbnull/android-demo](https://gitee.com/dkbnull/android-demo)

CSDN：[https://blog.csdn.net/dkbnull/article/details/104092233](https://blog.csdn.net/dkbnull/article/details/104092233)

微信：[https://mp.weixin.qq.com/s/EcNamBOpVV-5mlKhjqk3ww](https://mp.weixin.qq.com/s/EcNamBOpVV-5mlKhjqk3ww)

微博：[https://weibo.com/ttarticle/p/show?id=2309404465336207409222](https://weibo.com/ttarticle/p/show?id=2309404465336207409222)

知乎：[https://zhuanlan.zhihu.com/p/108509508](https://zhuanlan.zhihu.com/p/108509508)

---

