# Android Studio 中的assets和raw目录  
### 1. assets和res/raw的不同    
assets目录是Android的一种特殊目录，用于放置APP所需的固定文件，且该文件被打包到APK中时，不会被编码到二进制文件。  
Android还存在一种放置在res下的raw目录，该目录与assets目录不同。  
1.  assets目录不会被映射到R中，因此，资源无法通过R.id方式获取，必须要通过AssetManager进行操作与获取；res/raw目录下的资源会被映射到R中，可以通过getResource()方法获取资源。  
2.  多级目录：assets下可以有多级目录，res/raw下不可以有多级目录。  
3.  编码（都不会被编码）：assets目录下资源不会被二进制编码；res/raw应该也不会被编码。  

### 2. 添加二者的目录  
#### assets  
![](https://upload-images.jianshu.io/upload_images/1708416-00d2d31d0d716210.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/562)
#### res/raw  
![](https://upload-images.jianshu.io/upload_images/1708416-7f1786f6fafa2939.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/573)  
### 3.使用二者下的资源  
#### assets  
  
	AssetManager am = getAssets();  
	InputStream is = am.open("filename");   
获取到输入流后就可以进行操作了  

#### res/raw  

	InputStream is = getResources().openRawResource(R.id.fileNameID) ;
	//R.id.fileNameID
为需要访问的文件对应的资源ID 