# unable to load script from assets 'index.android bundle'  ,make sure your bundle is packaged correctly or youu're runing a packager server

引入到 Android studio 中和 源生 Android 结合起来是，运行报错

> unable to load script from assets 'index.android bundle'  ,make sure your bundle is packaged correctly or youu're runing a packager server

*解决办法*：

- 在 root/app/src/main 下新建 assets 文件夹；
- 在 Terminal 执行：

    react-native bundle --platform android --dev false --entry-file index.android.js --bundle-output app/src/main/assets/index.android.bundle --assets-dest app/src/main/res/

- 重新运行程序。