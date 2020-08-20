# 卡通渲染技术
## 1. 传统赛璐璐风
实现要素
1. 梯度化漫反射(Ramped Diffuse)，常用二值化手段分为明暗两个颜色
2. 轮廓勾边
3. 风格化的高光
### 1.1 光照
**方法1.** 通过漫反射Lambert模型NdotL结果通过阈值，分出明暗两个颜色（崩坏3是三个颜色有两层Shadow Color）。使用Blinn-Phong配合高光阈值增加高光部分，也可通过这个方法产生边缘光。  
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/144012610230289615.jpg?raw=true)  
(二之国)  

**方法2.** 通过Ramp贴图方式控明暗。  
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/161090be316143318499fa58fc667a65.011.1433145945.png?raw=true)  

### 1.2 轮廓勾边
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/144012610806291655.jpg?raw=true)  
《大神》Capcom  

#### 1.2.1 图像检测法
基于Sobel、Laplace、Canny等边缘检测算法勾勒轮廓线  
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/250px-Cel_shading_no_outlines.png?raw=true)![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/250px-Cel_shading_depth.png?raw=true)![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/250px-Cel_shading_normals.png?raw=true)  
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/250px-Cel_shading_edge_detection.png?raw=true)  
基于图像检测的方法，比较适合用于场景的勾线。这种方法还有一个好处，不会随着细节的复杂度增加，而降低运行效率。  
#### 1.2.2 Backface方法
Backface方法是一种比较传统的方法：对每个物体渲染两次，第一次画背面，使用线框模式或按法线方向放大一点；第二次则正常画物体。  
Backface方法，是比较高效的方法，而且通过顶点色可以实现各种风格化的控制。比如通常头发丝对勾线逐渐变细，美术就可再在mask来刷线表达想要效果，暗处的色彩饱和度也会提高
#### 1.2.3 基于笔刷的勾线方法
首先是轮廓线的提取，然后是连接轮廓线，根据模型的关系，将相邻的轮廓边连接成尽可能长的轮廓线，但有可能会出现轮廓线交叉的问题，这点也是需要处理的。然后是轮廓线的分段，在步骤二的基础上，将轮廓线在曲律上分开。然后是笔触，这种方法可以达到更为风格化的表现，笔触会更加明显。  
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/20200818173407.png?raw=true)  

### 1.3 《罪恶装备》的卡通渲染实现
原文 https://www.4gamer.net/games/216/G021678/20140703095/  
#### 1.3.1 Toon Shader
Lighting的结果和给予阴影的Shading，基本上都是，当超过某个阈值时是[明]，低于阈值的是[阴]（※），采用了极端普通的分二个等级的Tong Shader的配置。例如，Lighitng的结果是采用0~255的值，128以上是[明]，不到127的按[阴]来处理。  
阴影颜色采用阴影状态颜色贴图来决定阴影处的颜色。具体来说，角色皮肤上出现的阴影会带上一些红色，衣服上的阴影会带上衣服色彩的饱和度  
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/081-500x501.jpg.jpeg?raw=true)  
不使用阴影状态颜色纹理  
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/082-500x501.jpg.jpeg?raw=true)  
使用阴影状态颜色纹理  
#### 1.3.2 高光
赛璐璐风格的2D动画里，高光的存在感并不明显。《罪恶工具：X标记》对于高光部分，通过手绘附加高光的法则，将之加
入了光照中。  
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/447601_56ee50c2e44c9.png?raw=true)  
#### 1.3.3 阴影
为了让实现赛璐璐动画那样的明暗效果，在3D模型上下了多种多样的功夫。其一就是顶点颜色。具体来说就是在顶点颜色中的R通道加入了阴影权重参数。这个可以设置凹陷和被周围遮蔽地方的强度。在主人公索尔的场合，阴影权重参数为下巴和头附近的自投影效果做出了很大的贡献。  
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/061255576409604.jpg?raw=true)![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/061255585938446.jpg?raw=true)![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/061255597349232.jpg?raw=true)  
#### 1.3.4 法线调整
《罪恶装备》对于面部阴影出现的方式花了大量功夫。脸部和颈部是由艺术家手动进
行法线调整的。比如把脸颊附近的法线调整到和鬓角处接近，在鬓角变暗的时候，脸颊附近也会变暗。通过法线调整实现了让高精度模型拥有低精度模型的概括阴影。  
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/-447601_56ee50a3bb3f1.png?raw=true)  
一方面，衣服和头发使用了Gator。通过转写简单形状模型的法线分布，也能让复杂形状的模型出现概括的阴影。索尔的脚步有复杂的形状。为了实现概括的阴影，开发者准备了大小相近的圆筒模型。通过转写圆筒模型的法线进行法线调整。  
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/-447601_56ee50a359634.png?raw=true)  
#### 1.3.5 轮廓线
为了实现漫画里描线的效果，本作中采用了2个方法。  
一. 最基本的线描就是3D模型的轮廓线，这里采用了背面法。  
《罪恶装备》的3D模型在顶点颜色中加入了“进行线描时粗细的控制值”。通过这个就可以调整线条的强弱  
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/087-500x552.jpg.jpeg?raw=true)
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/086-500x552.jpg.jpeg?raw=true)
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/088-500x552.jpg.jpeg?raw=true)  

二. 在衣服和饰品的沟，针口和肌肉隆起等地方还用了另外的一个手法。3D构造上是沟的地方，通过背面法会有无法出现轮廓线的地方。这些地方只能用纹理贴图的方式加入。  
锯齿是纹理像素在多边形像素化时产生的。如果纹理像素是矩形排列的，锯齿就不容易出现。但即便纹理像素矩形排列，纹理也会出现斜方向的情况，这样还是会出现锯齿。也就是说，如果纹理像素都是水平或者垂直状态，加上虚化的效果，就可能避免出现锯齿出现。于是本村制作了完全是垂直线和水平线的轮廓线纹理。通过UV MAPPING的调整，这个贴图也可以适用于各种斜线和曲线。(注：也就是说不是用纹理去适应UV网格，而是让UV网格去适应纹理，从而实现平滑无锯齿的轮廓线条。)
### 1.4 《崩坏3》的卡通渲染实现
《崩3》的实现参考了《罪恶装备》的做法，但是阴影色处理进行了一些简化，它用两重阴影色替代了阴影状态颜色纹理。  
## 2. 其他卡通渲染
### 2.1 《Team Fortress2》
采用Warped diffuse+边缘光的方式  
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/20200818194141.png?raw=true)  
和传统的赛璐璐风相比保持风格化的同时会更立体。  
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/20200818194729.png?raw=true)
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/20200818194803.png?raw=true)  
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/20200818194834.png?raw=true)
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/20200818194940.png?raw=true)  
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/20200818195012.png?raw=true) 
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/20200818195120.png?raw=true)  
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/20200818195146.png?raw=true)
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/20200818195208.png?raw=true)  
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/20200818195242.png?raw=true)  

### 2.2 《琪亚娜·极乐净土》、《八重樱 · 桃源恋歌》
在《军团要塞2》的1D单通道Ramp贴图基础上改为2d多通道Ramp
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/20200819181100.png?raw=true) 
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/20200819181317.png?raw=true) 
头发用的是Scheuermann模型，眼球渲染基于解剖学类似UE4中实现
## 3. 基于PBR的卡通风格
写实的渲染方式，卡通化的造型。卡通化概念比较模糊，如果能找到标的物，就可以量化了。
### 3.1 手办风格
《决战!平安京》模型卡通化以手办为参考对象
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/3aeabd3c68b84943b28421e82ece6365.jpeg?raw=true) 
### 3.2 SD娃娃风格
![image](https://github.com/yangjunhua/CartoonRendering/blob/master/image/O1CN01gsiI1H2EJBRJQwi43_!!181558723.jpg?raw=true)  
