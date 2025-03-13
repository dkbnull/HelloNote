实现程序自动更新，简单来说就是两步：

1、把自动更新apk程序下载到本地；

2、安装下载的apk程序

话不多说，先看效果：

./assets/01_Android中使用okhttp实现程序自动更新.assets/SVID_20200126_172655_1.mp4

<br>

*关于实现程序自动更新服务端相关操作可参考[Spring Boot实现文件上传与下载](https://blog.csdn.net/dkbnull/article/details/88858717)*

## 0. 开发环境

- IDE：Android Studio
- JDK：1.8
- Gradle Plugin：3.5.3
- Gradle：5.4.1

## 1. 检查更新服务

~~~java
public class UpdateService {

    private static OkHttpClient okHttpClient;

    public static void download(final String fileName, final UpdateCallback callback) {
        String url = "http://127.0.0.1:8090/springbootdemo/log/download/" + fileName;
        Request request = new Request.Builder()
                .addHeader("Accept-Encoding", "identity")
                .url(url).build();

        if (okHttpClient == null) {
            okHttpClient = new OkHttpClient();
        }

        okHttpClient.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                callback.onFailure();
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                if (response.body() == null) {
                    callback.onFailure();
                    return;
                }

                File filePath = new File(CommonConstants.DOWNLOAD_PATH);
                if (!filePath.exists()) {
                    filePath.mkdirs();
                }

                long contentLength = response.body().contentLength();
                byte[] buffer = new byte[1024];
                File file = new File(filePath.getCanonicalPath(), fileName);
                try (InputStream is = response.body().byteStream();
                     FileOutputStream fos = new FileOutputStream(file)) {
                    LoggerUtils.getLogger().info("保存路径：" + file);

                    int length;
                    long sum = 0;
                    while ((length = is.read(buffer)) != -1) {
                        fos.write(buffer, 0, length);
                        sum += length;
                        int progress = (int) (sum * 1.0f / contentLength * 100);
                        callback.onProgress(progress);
                    }
                    fos.flush();

                    callback.onSuccess();
                } catch (Exception e) {
                    callback.onFailure();
                }
            }
        });
    }

    public interface UpdateCallback {

        void onSuccess();

        void onProgress(int progress);

        void onFailure();
    }
}
~~~

## 2. 检查更新界面

~~~java
@ActivityLayoutInject(R.layout.activity_update)
public class UpdateActivity extends BaseActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
    }

    @OnClick(R.id.menuBtnCheckUpdate)
    public void onClickCheckUpdate() {
        final ProgressDialog dialog = new ProgressDialog(this);
        dialog.setProgressStyle(ProgressDialog.STYLE_HORIZONTAL);
        dialog.setCanceledOnTouchOutside(false);
        dialog.setCancelable(true);
        dialog.setTitle("正在下载");
        dialog.setMessage("请稍后...");
        dialog.setProgress(0);
        dialog.setMax(100);
        dialog.show();

        String fileName = "AndroidDemo-v1.0.0.0.apk";
        UpdateService.download(fileName, new UpdateService.UpdateCallback() {
            @Override
            public void onSuccess() {
                dialog.dismiss();

                if (!Environment.getExternalStorageState().equals(Environment.MEDIA_MOUNTED)) {
                    return;
                }

                File file = new File(CommonConstants.DOWNLOAD_PATH + fileName);
                try {
                    LoggerUtils.getLogger().info("安装文件目录：" + file);
                    LoggerUtils.getLogger().info("准备安装");
                    installApk(file);
                } catch (Exception e) {
                    LoggerUtils.getLogger().info("获取打开方式错误", e);
                }
            }

            @Override
            public void onProgress(int progress) {
                dialog.setProgress(progress);
            }

            @Override
            public void onFailure() {
                dialog.dismiss();
            }
        });
    }

    private void installApk(File file) {
        Intent intent = new Intent(Intent.ACTION_VIEW);
        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        Uri fileUri;

        //Android7.0以上
        if (Build.VERSION.SDK_INT >= 24) {
            fileUri = FileProvider.getUriForFile(this,
                    BuildConfig.APPLICATION_ID + ".provider", file);
            intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
        } else {
            fileUri = Uri.fromFile(file);
        }

        intent.setDataAndType(fileUri, "application/vnd.android.package-archive");
        startActivity(intent);
    }
}
~~~

<br>

如果你的Android机器是7.0以下，到这里就可以了，但是如果是7.0以上，还需如下操作。

## 3. 新建file_paths.xml文件

res\xml文件夹下新建file_paths.xml，path就是下载的apk存放目录

```xml
<?xml version="1.0" encoding="utf-8"?>
<paths>
    <external-path
        name="download"
        path="AndroidDemo/download/" />
</paths>
```

## 4. 注册FileProvider

在AndroidManifest.xml的application下注册

~~~xml
        <provider
            android:name="androidx.core.content.FileProvider"
            android:authorities="cn.wbnull.androiddemo.provider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths" />
        </provider>
~~~

## 5. 注册读写权限

在AndroidManifest.xml中注册读写和网络权限

~~~xml
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
~~~



好了，到这里我们就实现最上视频中所示效果了。



---

GitHub：[https://github.com/dkbnull/android-demo](https://github.com/dkbnull/android-demo)

Gitee：[https://gitee.com/dkbnull/android-demo](https://gitee.com/dkbnull/android-demo)

CSDN：[https://blog.csdn.net/dkbnull/article/details/104088585](https://blog.csdn.net/dkbnull/article/details/104088585)

微信：[https://mp.weixin.qq.com/s/jkK7rR6rRmbeMekqh3h6WQ](https://mp.weixin.qq.com/s/jkK7rR6rRmbeMekqh3h6WQ)

微博：[https://weibo.com/ttarticle/p/show?id=2309404465065775464739](https://weibo.com/ttarticle/p/show?id=2309404465065775464739)

知乎：[https://zhuanlan.zhihu.com/p/104902195](https://zhuanlan.zhihu.com/p/104902195)

---

