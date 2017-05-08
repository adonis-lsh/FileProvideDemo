# Android7.0适配之FileProvide(拍照,裁剪,应用安装)

现在Android7.0在份额在不断的增加(个人感觉Android升级给用户带来最多的并不是更流畅而是更安全),许多应用都已经开始或者已经适配了Android7.0,如果你还不了解Android7.0有哪些新特性的小伙伴们,请[移步](https://developer.android.com/about/versions/nougat/android-7.0-samples.html),查看详细的介绍.
如果你的App是一个轻量级的应用,那么你很有可能会去调用系统的一些功能如拍照,裁剪,应用下载,安装,而不是依靠第三方库,那么你肯定用的到这个FileProvide.要了解一个你不熟悉的东西,最好的方式就是看[官网的API文档(教材)](https://developer.android.com/reference/android/support/v4/content/FileProvider.html),那么我们就来翻译一下大致内容(英语水平有限,大神勿喷,不好的地方还请指正):

FileProvider是ContentProvider(有助于App共享文件更加安全的组件)的一个特殊的子类,通过 content:// Uri获取一个文件而不是file:/// Uri.
当你创建一个包含有content URI的Intent的时候,Content URI赋予你临时的读写权限.为了可以给一个目标app(原文client app,这里应该是目标App)发送一个特定的content URI,你也可以通过调用Intent.setFlags()去添加权限,这些权限一直有效,只要接收的Activity在栈中还存活着.要是跳转到Service,只要这个Service在运行,权限就有效.
相比较而言,用file:/// Uri来获取文件,你不得不修改系统底层的文件权限.在你改变文件权限之前,你授予权限,对于任何app都是可用的,所以授予这个级别的权限从根本上是不安全的.(就是liunx的rwx)
content URI提高了文件安全访问的级别,使FileProvider成为Android的安全体系重要组成部分。

FileProvider 可大致分为下面几个部分

1. 自定义一个FileProvider
2. 指定可用文件
3. 给文件生成content URI
4. 授予URI一个临时权限
5. 给另一个App提供URI


## 自定义一个FileProvider
因为FileProvider的默认功能已经包含了将一个文件生成Content URI,所以,你不需要再在你的代码里去定义一个子类.相应的,你完全可以在你App的XML中通过 <provider>元素去指定一个FileProvider,包括指定FileProvider的组件,给 android.support.v4.content.FileProvider设置 android:name属性,基于你的控制域给 android:authorities设置一个属性.例如,如果你控制mydomain.com域,你应该用com.mydomain.fileprovider授权.设置 android:exported属性为false; FileProvider不需要成为一个共有的类,设置android:grantUriPermissions这个属性为true,则允许你给文件临时授权.例子

    <manifest>
    ...
    <application>
        ...
        <provider
            android:name="android.support.v4.content.FileProvider"
            android:authorities="com.mydomain.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            ...
        </provider>
        ...
    </application>
	</manifest>

如果你想要重写FileProvider中默认的方法,请继承FileProvider,并且要在<provider> 元素下的android:name写上全类名

## 指定可用文件
一个FileProvide仅仅只能为你生成一个预先在目录中指定的文件的content URI,在XML中,用<paths>的子元素去指定一个目录,并指定它的存储区域和路径.例如,下面这个paths元素告诉FileProvider,你打算去访问你的images/子目录下面私有文件的content URI

    <paths xmlns:android="http://schemas.android.com/apk/res/android">
    <files-path name="my_images" path="images/"/>
    ...
	</paths>

<paths>元素必须有一个或多个下面类似的子标签
    <files-path name="name" path="path" />
它表示你的app的内部存储区根目录的文件,该子目录的根路径等同于Context.getFileDir

    <cache-path name="name" path="path" />
它表示你的app的内部缓存区根目录的文件,该子目录的根路径等同于getCacheDir()

    <external-path name="name" path="path" />
它表示你app的外部缓存区根目录的文件,该子目录的根路径等同于Environment.getExternalStorageDirectory().

    <external-files-path name="name" path="path" />
它表示你的app的外部存储区根目录的文件,该子目录的根路径等同于Context.getExternalFilesDir(null).

    <external-cache-path name="name" path="path" />
它表示你的app的外部缓存区根目录的文件,该子目录的根路径等同于Context.getExternalCacheDir().

这些子元素都有相同的属性

    name="name"

 一个URI路径段,为了确保安全,这个值隐藏了你将要分享的子目录的名字,这个子目录的名字被包含在path的属性值中.

    path ="path"

你将要分享的子目录,虽然name属性是一个URI路径片段,但是path的值是一个真正的紫木槿的名字.注意这个值只能引用一个子目录,不能是一个或多个文件,你不能通过文件名单独的分享一个文件,也不能用通配符来指定某一类型的文件

你必须为你想要content URI文件的每个目录指定<path>子元素.例如,这写xnl元素指定了两个目录

    <paths xmlns:android="http://schemas.android.com/apk/res/android">
    	<files-path name="my_images" path="images/"/>
    	<files-path name="my_docs" path="docs/"/>
	</paths>

把<paths>元素和它的子元素放到你项目中的一个xml中,例如,你可以把它们添加到一个叫res/xml/file_paths.xml文件中,为了让它和FileProvider联系起来,在定义好FileProvider的<provider>元素下添加一个<meta-data>做为子元素,设置<meta-data>元素的"android:name"属性为 android.support.FILE_PROVIDER_PATHS,设置"android:resource"元素的属性为@xml/file_paths(注意不要指定.xml扩展名).例如:

	<provider
	    android:name="android.support.v4.content.FileProvider"
	    android:authorities="com.mydomain.fileprovider"
	    android:exported="false"
	    android:grantUriPermissions="true">
	    <meta-data
	        android:name="android.support.FILE_PROVIDER_PATHS"
	        android:resource="@xml/file_paths" />
	</provider>

## 给文件生成content URI
为了给另一个app通过content URI分享文件,因此,你的app不得不生成一个content URI.为了生成content URI,首先要给文件new File,然后传递File 到getUriForFile(),你可以通过 把getUriForFile()返回的content URI放在Intent中传递给另一个app,目标app收到这个content URI就能打开这个文件并且通过调用 ContentResolver.openFileDescriptor得到ParcelFileDescriptor进而读取它的内容.

例如,假设你的app正要通过拥有 com.mydomain.fileprovider权限的FileProvider给另一个app提供文件,你可以通过下面的代码去得到一个在images/ 子目录下的default_image.jpg的content URI:

    File imagePath = new File(Context.getFilesDir(), "images");
    File newFile = new File(imagePath, "default_image.jpg");
    Uri contentUri = getUriForFile(getContext(), "com.mydomain.fileprovider", newFile);

由于先前片段截取的结果,getUriForFile()返回这样一个结果:
content://com.mydomain.fileprovider/my_images/default_image.jpg

## 授予URI一个临时权限
想要授予从getUriForFile()获取content URI一个访问权限,可以通过下面任意一种方式:
1. 把content:// Uri传入 Context.grantUriPermission(package, Uri, mode_flags) 方法中,用你想要的模式标签,这将授予content URI指定的包临时访问权限,你可以把mode_flags参数的值设置为 FLAG_GRANT_READ_URI_PERMISSION,FLAG_GRANT_WRITE_URI_PERMISSION ,或者都设置上.这个授权将在你调用revokeUriPermission()或者重启设置之前一直有效.
2. 通过Intent调用setData()把content URI放进去
3. 接下来,调用Intent.setFlags()设置为FLAG_GRANT_READ_URI_PERMISSION,FLAG_GRANT_WRITE_URI_PERMISSION,至少一个.
4. 最后,发送Intent到另一个app,大多数情况下,你可通过调用setResult()来实现它.
5. 在接受方Activity在栈中存活的情况下,被赋予Intent权限中仍然有效.当栈被销毁的时候,权限将自动被移除.权限将自动扩展到该app的其他组件,并授权给其中的一个Activy.


##给另一个App提供URI
有各种各样的方式去吧一个文件的content URI提供给目标app.一个通用的方式是通过调用startActivityResult()有目标app启动你的app,目标app给你的app发送一个Intent去启动你的一个Activity,对此,你的app可以及时的给目标app返回一个content URI或者提供一个界面供用户选择文件.在最后一种情况下,一旦用户选择了文件,你的app就能返回它的content URI.以上联众情况,你的app都可以通过setREsult()发送包含有content URI的Intent

你也可以把content URI放到ClipData对象中,然后把对象添加到Intent中发送给你的目标app.要做到这一点,调用Intent.setClipData().当你用这个方法的时候,你能添加多种ClipData对象给Intent,每一个都有自己的content URI,当你调用通过Intent调用Intent.setFlags()设置临时权限的时候,所有的content URI都将有相同的权限.

注：该Intent.setClipData()方法只适用于平台版本16（安卓4.1）及更高版本。如果你想保持与以前版本的兼容性，你应该在一个时间发送一个内容URI Intent。设置行动ACTION_SEND，并通过调用把URI数据 setData()。

# 举个例子
![这里写图片描述](http://opgkgu3ek.bkt.clouddn.com/17-5-8/21600260-file_1494215797284_133b6.gif)

## 裁剪
先看一个正常的在系统相册选取图片然后裁剪的例子
![](http://opgkgu3ek.bkt.clouddn.com/17-5-8/76828988-file_1494208163394_41ce.png)

我这面path里面"",说明是根路径,这里我只是为了省事一点,当然你可以写上要共享目录,整体xml的意思的就是我要共享外部储存的路径
![](http://opgkgu3ek.bkt.clouddn.com/17-5-8/49348729-file_1494208230355_fe73.png)

下面是正常的裁剪:

	@AfterPermissionGranted(RC_STORAGE_PREM)
    public void cropImage(View view) {
        mPerms = new String[]{Manifest.permission.READ_EXTERNAL_STORAGE,Manifest.permission.WRITE_EXTERNAL_STORAGE};
        if (EasyPermissions.hasPermissions(this, mPerms)) {
            Intent intent = new Intent(Intent.ACTION_PICK);
            intent.setType("image/*");
            startActivityForResult(intent,SelectPhoto);
        } else {
            // Ask for one permission
            EasyPermissions.requestPermissions(this, getString(R.string.tishi),
                    RC_STORAGE_PREM, mPerms);
        }

    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (data.getData() != null) {
            switch (requestCode) {
                case SelectPhoto:
                    Uri uri = data.getData();
                    Log.e(TAG, uri.toString());
                    crop(uri,CropPhoto , 1, 1);
                    break;
                case CropPhoto:
                    Bitmap bitmap = BitmapFactory.decodeFile(getPath(data.getData()));
                    Log.e(TAG, data.getData().toString());
                    mImageView.setImageBitmap(bitmap);
                    break;
            }
        }
    }

    private void crop(Uri uri, int tag, int aspectX, int aspectY) {
        // 裁剪图片意图
        Intent intent = new Intent("com.android.camera.action.CROP");
        intent.setDataAndType(uri, "image/*");
        intent.putExtra("crop", "true");
        // 裁剪框的比例，1：1
        intent.putExtra("aspectX", aspectX);
        intent.putExtra("aspectY", aspectY);
        // 裁剪后输出图片的尺寸大小
        intent.putExtra("outputX", 250);
        intent.putExtra("outputY", 250);
        intent.putExtra("outputFormat", "JPEG");// 图片格式
        intent.putExtra("noFaceDetection", true);// 取消人脸识别
        intent.putExtra("return-data", false);
        intent.putExtra(MediaStore.EXTRA_OUTPUT, getCachePath()+"Cache");
        // 开启一个带有返回值的Activity，请求码为PHOTO_REQUEST_CUT
        startActivityForResult(intent, tag);
    }

    public String getCachePath() {
        File cacheDir;
        if (android.os.Environment.getExternalStorageState().equals(android.os.Environment.MEDIA_MOUNTED))
            cacheDir = getExternalCacheDir();
        else
            cacheDir = getCacheDir();
        if (!cacheDir.exists())
            cacheDir.mkdirs();
        return cacheDir.getAbsolutePath();
    }

    private String getPath(Uri uri) {
        String[]  data = { MediaStore.Images.Media.DATA };
        CursorLoader loader = new CursorLoader(this, uri, data, null, null, null);
        Cursor cursor = loader.loadInBackground();
        int column_index = cursor.getColumnIndexOrThrow(MediaStore.Images.Media.DATA);
        cursor.moveToFirst();
        return cursor.getString(column_index);
    }
需要注意的地方:
![这里写图片描述](http://opgkgu3ek.bkt.clouddn.com/17-5-8/4994057-file_1494212343452_648.png)
如果return-data,你写的是true,就是你返回的是数据不是路径,可能会报一个错误
android.os.TransactionTooLargeException: data parcel size 548172 bytes,原因是Intent携带的数据太大,不过正常的国内rom没有这个问题,为了安全起见,还是传路径好一点.

你以为这样就完了,当然不是:如果你调用的不是系统相册选取的图片,就会遇到问题,看刚才打印的log;
![这里写图片描述](http://opgkgu3ek.bkt.clouddn.com/17-5-8/4867054-file_1494212904468_3657.png)
系统系统选取图片返回的是content开头的Uri,API24下是不会报错的,如果获取的图片路径是File开头的,此时用系统裁剪改怎么做呢?
![这里写图片描述](http://opgkgu3ek.bkt.clouddn.com/17-5-8/57686936-file_1494214680509_10e2a.png)

这样就完了吗?并不是,看我们前面的xml里面external-path,也就是外部路径,我们知道,现在的Android手机都把内置了储存空间,也是相当于一个SD卡,如果你有第二块SD呢,如三星的机型,如果传入的SD卡中的图片路径就会报错,why?因为我们我们前面写的external-path,表示的外部储存的路径,也就是你的机身储存那么路径又要怎么表示?前面我们翻译了文档,发现上面并没有说明这类情况,那么我们该怎么解决?看FileProvide的源码.
![这里写图片描述](http://opgkgu3ek.bkt.clouddn.com/17-5-8/31473754-file_1494215027894_12dc.png)
发现里面除了文档上面说的那五类中路径没还有一个就是root-path,也就是整个手机的根路径,那就好办了
![这里写图片描述](http://opgkgu3ek.bkt.clouddn.com/17-5-8/22363625-file_1494215392429_108a4.png)
好了,关于系统剪裁就介绍到这里.[代码地址](https://github.com/adonis-lsh/FileProvideDemo)
