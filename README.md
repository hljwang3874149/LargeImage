# LargeImage
Android 加载大图  可以高清显示10000*10000像素的图片

#实现原理
#监听View的显示区域的变化，然后加载显示区域内应该显示的图片区域，然后绘制到View上 #
  

###1.UpdateView负责监听显示区域的变化的View，子类通过重写onUpdateWindow(Rect visiableRect)监听显示区域，大部分代码源于SurfaceView监听代码  ###

### 2.ImageManager负责加载显示区域的图片块。   ###
###3.LargeImageView负责绘制图片块   ###

###实现细节：  ###
每次LargeImageView的onDraw方法都会调用ImageManagerd的getDrawData(float imageScale, Rect imageRect)方法，imageRect为在View上图片显示的区域(需要加载的图片区域)，imageScale 假设等于4的话，就是View上显示1像素，image要加载4个像素的区域（缩小4倍的图片）  
getDrawData(float imageScale, Rect imageRect)实现细节：  
手势移动，图片显示区域会变化。比如显示区域是800*800，向右移动2像素，难道要重新加载800*800的图片区域？
所以我采用了图片切块的操作,分块的优化  



1. 比如图片显示比例是1，那么要横向分多少份才，纵向分多少分，才合理？图片显示比例是4，横向分多少份才，纵向分多少分，才合理。
-  
 
所以我采用了基准块（图片比例是1，一个图片块的宽高的合理sise） 
BASE_BLOCKSIZE = context.getResources().getDisplayMetrics().widthPixels / 2;  
图片缩放比例为1的话，图片块宽高是BASE_BLOCKSIZE  
图片缩放比例为4的话，图片块宽高是4*BASE_BLOCKSIZE  
图片没被位移，那么屏幕上显示横向2列，纵向getDisplayMetrics().heightPixels/BASE_BLOCKSIZE行  


2.因为手势放大缩小操作要加载不同清晰度的图片区域，比如之前的图片缩放是4，现在缩放是4.2，难道要重新加载？ 
-  
通过public int getNearScale(float imageScale)方法计算趋于2的指数次方的值（1，2，4，8，16）  
比如3.9和4.2和4比较接近，就直接加载图片显示比例为4的图片块  

3.之前没加载的区域，难道要空白显示么？
- 
为了避免加载出现白色块，我会缓存当前比例的加载的图片块，以及2倍比例的图片块（之前加载过，并且当前还属于当前显示区域的，如果不是的话也不缓存它）
所以发现没有的话去拿其他比例的图片区去显示

4.难道只加载显示区域？
- 
当然，还会去加载旁边一部分没显示的区域的图片块


5.onDraw方法是UI线程,调用getDrawData(float imageScale, Rect imageRect)加载图片块的方法怎么不卡住
-   

getDrawData只返回之前加载过的图片块，而没有加载的是通过LoadHandler.sendMessage去加载
LoadHandler的Loop是通过HandlerThread线程创建的Loop，也就是开个线程加载. 
   
每加载一个图片块通过	onImageLoadListenner.onBlockImageLoadFinished();onDraw重绘  
onDraw又调用getDrawData加载，直至需要显示的图片块加载完成



# 说明   
其中 android-gesture-detectors-lib 手势类库  
源地址https://github.com/Almeros/android-gesture-detectors   

# 联系方式和问题建议

* 微博:http://weibo.com/u/3181073384
* QQ 群: 开源项目使用交流，问题解答: 549284336（开源盛世） 

License
=======

    Copyright 2015 shizhefei（LuckyJayce）

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
