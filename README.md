### If you don't know, please click here : [CmonBaby](https://www.cmonbaby.com)

## Compress ![Build Status](https://travis-ci.org/greenrobot/EventBus.svg?branch=master)

## About coding
```java
public class MainActivity extends AppCompatActivity {

    private String cameraCachePath; // Photo source file path
    private final static String TAG = "simon >>> ";
    private Disposable subscribe;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Permission request sample
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            String[] perms = {Manifest.permission.CAMERA,
                    Manifest.permission.WRITE_EXTERNAL_STORAGE};
            if (checkSelfPermission(perms[0]) == PackageManager.PERMISSION_DENIED ||
                    checkSelfPermission(perms[1]) == PackageManager.PERMISSION_DENIED) {
                requestPermissions(perms, 200);
            }
        }

        // compress(compressMore()); // Batch Compress
        // supportRxJava(compressMore()); // Support RxJava
    }

    private void aboutCoding() {
        Compressor.with(this)
                .load("/storage/emulated/0/DCIM/Camera/IMG_20190322_142010.jpg") // need`s compress photos
                // Cache image path after compression, Default: Constants.COMPRESS_CACHE
                .targetDir("")
                .unCompressMinPixel(1000) // Minimum pixel uncompressed, Default：1000
                .unCompressNormalPixel(2000) // Normal pixel uncompressed, Default：2000
                .maxPixel(1200) // Max pixel, Default：1200
                .maxSize(200 * 1024) // Max size, Default：200 * 1024 = 200KB
                .enablePixelCompress(true) // Enable pixel compress, Default：true
                .enableQualityCompress(true) // Enable quality compress, Default：true
                .enableReserveRaw(true) // Enable reserveRaw, Default：true
                .showCompressDialog(new DialogListener() { // dialog listener
                    @Override
                    public void show() {
                        Log.e(TAG, "dialog show");
                    }

                    @Override
                    public void dismiss() {
                        Log.e(TAG, "dialog dismiss");
                    }
                })
                .filter(new CompressFilter() { // open filter
                    @Override
                    public boolean apply(String path) {
                        return !(TextUtils.isEmpty(path) || path.toLowerCase().endsWith(".gif"));
                    }
                })
                .setCompressListener(new CompressListener() { // compress listener
                    @Override
                    public void onCompressStart() {
                        Log.e(TAG, "start compress");
                    }

                    @Override
                    public void onCompressSuccess(List<PhotoBean> photoBeans) {
                        if (!photoBeans.isEmpty()) {
                            for (PhotoBean bean : photoBeans) {
                                Log.e(TAG, bean.getCompressPath());
                            }
                        }
                    }

                    @Override
                    public void onCompressFailed(CompressException e, String... errorPhotoPath) {
                        if (errorPhotoPath.length > 0) {
                            Log.e(TAG, errorPhotoPath[0] + e.getDetailMessage());
                        }
                    }
                }).compress();
    }

    private void supportRxJava(final List<PhotoBean> photoBeans) {
        /**
         * Function<T, R>
         * T：String、File、Uri、PhotoBean、List<String>、List<File>、List<Uri>、List<PhotoBean>
         * R：List<String>、List<File>、List<PhotoBean>
         */
        subscribe = Flowable.just(photoBeans)
                .subscribeOn(Schedulers.io())
                .map(new Function<List<PhotoBean>, List<PhotoBean>>() {
                    @Override
                    public List<PhotoBean> apply(@NonNull List<PhotoBean> strings) throws Exception {
                        return Compressor.with(MainActivity.this).load(photoBeans).get();
                    }
                })
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<List<PhotoBean>>() {
                    @Override
                    public void accept(List<PhotoBean> photoBeans) {
                        if (!photoBeans.isEmpty()) {
                            for (PhotoBean file : photoBeans) {
                                Log.e(TAG, file.getCompressPath());
                            }
                        }
                    }
                });
    }

    // Compress more
    private List<PhotoBean> compressMore() {
        List<PhotoBean> photoBeans = new ArrayList<>();
        photoBeans.add(PhotoBean.create("/storage/emulated/0/DCIM/Camera/IMG_20190322_142010.jpg"));
        photoBeans.add(PhotoBean.create("/storage/emulated/0/DCIM/Camera/IMG_20190414_190611.jpg"));
        photoBeans.add(PhotoBean.create("/storage/emulated/0/DCIM/Camera/IMG_20190322_141920.jpg"));
        return photoBeans;
    }

    // click camera
    public void camera(View view) {
        try {
            cameraCachePath = IntentUitls.openCamera(this, Constants.CAMERA_CODE);
        } catch (CompressException e) {
            e.printStackTrace();
        }
    }

    // click album
    public void album(View view) {
        IntentUitls.openAlbum(this, Constants.ALBUM_CODE);
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        // camera callback
        if (requestCode == Constants.CAMERA_CODE && resultCode == RESULT_OK) {
            compress(cameraCachePath);
        }

        // album callback
        if (requestCode == Constants.ALBUM_CODE && resultCode == RESULT_OK) {
            if (data != null) {
                Uri uri = data.getData();
                if (uri != null) {
                    String path = UriParseUtils.getPath(this, uri);
                    compress(path);
                }
            }
        }
    }

    // compress single photo
    private void compress(String path) {
        Compressor.with(this)
                .load(path)
                .targetDir("")
                .showCompressDialog(new SimpleDialogListener(MainActivity.this))
                .setCompressListener(new SimpleCompressListener() { // simple impl
                    @Override
                    public void onCompressSuccess(List<PhotoBean> photoBeans) {
                        if (!photoBeans.isEmpty()) {
                            for (PhotoBean bean : photoBeans) {
                                Log.e(TAG, bean.getCompressPath());
                            }
                        }
                    }
                }).compress();
    }

    // compress photos
    private void compress(List<PhotoBean> photoBeans) {
        Compressor.with(this)
                .load(photoBeans)
                .targetDir("")
                .showCompressDialog(new SimpleDialogListener(MainActivity.this, "title"))
                .setCompressListener(new SimpleCompressListener() { // simple impl
                    @Override
                    public void onCompressSuccess(List<PhotoBean> photoBeans) {
                        if (!photoBeans.isEmpty()) {
                            for (PhotoBean bean : photoBeans) {
                                Log.e(TAG, bean.getCompressPath());
                            }
                        }
                    }
                }).compress();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        subscribe.dispose();
    }
}
```

## Add Compress to your project
<a href="https://www.cmonbaby.com">
<img src="https://img.shields.io/bintray/v/cmonbaby/simon/compress_core?label=maven-central"></a>

Via Gradle:
```gradle
implementation 'com.cmonbaby.compress.core:compress_core:1.0.1'
```

Via Maven:
```xml
<dependency>
    <groupId>com.cmonbaby.compress.core</groupId>
    <artifactId>compress_core</artifactId>
    <version>1.0.1</version>
</dependency>
```

## License

Copyright (C) 2013-2020 Markus Junginger, Simon (https://www.cmonbaby.com)  
Compress binaries and source code can be used according to the [Apache License, Version 2.0](LICENSE).
