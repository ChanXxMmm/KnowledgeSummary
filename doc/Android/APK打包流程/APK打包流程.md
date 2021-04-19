![image](https://user-images.githubusercontent.com/61224872/115194660-5579c080-a120-11eb-92da-4ca40e59d8cf.png)


首先我们要了解下apk的打包流程
* 首先通过aapt工具将资源生成R.java
* R.java，自己的代码和通过aidl生成的java代码通过javac编译成.class文件
* 将.class文件和第三方资源库通过dex生成.dex文件
* 将资源文件通过aapt打包成.ap_文件，其他资源和.dex文件通过apkbuilder打包生成.apk
* 将.apk与签名文件通过Jarsigner打包成签名的.apk
* 最后签名的.apk通过zipalign进行四字节对齐
  * zipalign: 运行更快(对齐后mmap读取)，节约RAM内存
