---
layout: post
title: Java PDF转JPEG图片详解
categories:
- Java
---

//加载pdf文件

	File file = new File(pdfFile);

//以只读形式打开   RandomAccessFile 用法详见JDK API

	RandomAccessFile raf = new RandomAccessFile(file,"r");

//FileChannel类用于读取、写入、映射和操作文件的通道。
//其中RandomAccessFile含有getChannel()方法，用于返回与此文件关联的唯一FileChannel对象。详见JDK API，这一点类似于操作容器时，取数据的Iterator迭代器。说白了就是建立一个通道对文件进行查询、修改。

	FileChannel channel = raf.getChannel();

//channel.size()    返回此通道文件的当前大小。详见JDK API
//channel.map(FileChannel.MapMode mode , long position ,long size)   有三个参数，第一个参数为指定文件映射模式的类型安全枚举，一共有三项（PRIVATE、READ_ONLY、READ_WRITE）专用映射模式、只读映射模式和读取、写入映射模式。详见JDK API。。。从文件中的位置0开始到整个文件的大小结束，映射的范围。此处就相当于映射整个文件。将映射的文件读入ByteBuffer字节缓冲区。

	ByteBuffer buf = channel.map(FileChannel.MapMode.READ_ONLY, 0, channel.size());

//PDFFile为PDFRenderer包中的一个类。源代码详见：https://java.net/projects/pdf-renderer/sources/svn/show/trunk/src/com/sun/pdfview?rev=140
//源码中构造函数对PDFFile进行初始化，传入参数为ByteBuffer buf ，详见108行：https://java.net/projects/pdf-renderer/sources/svn/content/trunk/src/com/sun/pdfview/PDFFile.java?rev=140
//对pdf文件加载进来了。此处和File的加载不同，File加载只能对文件进行整体加载作为一个对象存在，不能做一些查询以及内容修改等操作。此处的加载时真正可以对pdf进行操作的加载。

	PDFFile pdffile = new PDFFile(buf );
//同样，PDFPage是对pdffile对象进行操作，将pdf文件的一页加载。

	PDFPage page = pdffile.getPage(1);
//Rentangle类可以指定坐标空间中的一个区域，通过坐标空间中Rentanle对象左上方的点（x,y）、宽度和高度来定义某个区域。

	Rectangle rect = new Rectangle(0, 0, (int)page.getBBox().getWidth(), (int)page.getBBox().getHeight());
//生成图片，PDFPage中存在getImage（int width,int height,Rentangle2D clip,ImageObserver observer,boolean drawbg,boolean wait）方法，详见PDFPage源代码201行：https://java.net/projects/pdf-renderer/sources/svn/content/trunk/src/com/sun/pdfview/PDFPage.java?rev=140，其中getImage有6个参数，第一二个参数不用多说了，指定生成图片的长宽，第二个参数为生成图片的内容，第三个不用管null，第四个true输出白色背景图片false输出黑色背景图片，最后一个不用管为true表示图片完全生成才返回。

	Image img = page .getImage(rect. width* n, rect. height* n, rect, null, true, true);
//其实到此处图片已经生成了。但是存放在缓冲区，且Image 为一个抽象类，对象不能实例化。
//BufferedImage继承Image，BufferedImage和Image主要就是将图片加载到内存。有三种构造方法可对其进行初始化，详见JDK API （很详细）,这里的TYPE_INT_RGB表示一个图像合成整数像素的8为RGB颜色分量。

	BufferedImage tag = new BufferedImage(rect. width* n, rect. height* n, BufferedImage. TYPE_INT_RGB);
//这里其实可以分两步写

	Graphics2D graph2d = tag.getGraphics();    tag.getGraphics()返回Grapics2D
	graph2d.drawImage(img,0,0,rect.width*n,rect.height*n,null);     

Graph2D类扩展Grapics类，提供对几何形状、坐标转换、颜色管理和文本布局以及更为复杂的控制。
drawImage()方法含有两种参数，这里的参数为 呈现一个图像，在绘制前进行图像空间到用户空间的转换。如果Image被完全加载并完全呈现，则返回true；否则为null，如果仍在加载返回false；
所以这句的作用就是确保，Image被完全加载才进行下一步。

	tag.getGraphics().drawImage(img , 0, 0, rect. width* n, rect.height *n , null);
//文件输出流，用于写入图像数据的原始字节流

	FileOutputStream out = new FileOutputStream( getPdfFilePath+ "\\" + i + ".bmp");
//创建一个和指定输出流关联的JPEGImageEncoder对象                 

	JPEGImageEncoder encoder = JPEGCodec.createJPEGEncoder(out); 
//设置默认参数，这里其实是真正将内存中的图片放到了文件上。后边的对param进行的操作都是在对图像进行调整以及优化。                    

	JPEGEncodeParam param = encoder.getDefaultJPEGEncodeParam( tag);
//设置图片质量1f-0.01f。

	param.setQuality(1f, true);
//设置图像为JPEG编码

	encoder.setJPEGEncodeParam( param);
	encoder.encode( tag);
	out.close();