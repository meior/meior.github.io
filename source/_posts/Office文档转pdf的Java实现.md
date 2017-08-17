---
title: Office文档转pdf的Java实现
date: 2016-07-23 20:42:50
categories: Java
tags: [Office, PDF]
---
日前曾有在线预览Office文档的需求，简单搜了一下，发现这方面的需求还比较多，解决办法也是五花八门。考虑到服务器跨平台，放弃了调用Office提供的api，另外在后台文件格式转换方面，有转为html的也有转为flash的，但多少还有些不足之处，字体样式以及中文编码支持都不太完美。话说中文字体的许可证费用真的很贵么。

最后找到了[OpenOffice](http://www.openoffice.org/)，一个开源的跨平台办公软件套件，能编辑处理常见格式文档，兼容Microsoft Office文档。和Office不同的是，它不仅提供了图形编辑界面，还为开发者提供了应用程序接口。这里采用其提供的命令行接口以及`jodconverter`库实现office文档到pdf文件的转换，由于现代浏览器是可以直接打开pdf文档的，间接地实现了Office文档在线预览的效果，并且相比于flash以及图片等形式还可以复制文本内容。OpenOffice不需要额外的配置，只需要下载对应平台的安装包进行安装即可。下面来简单实现一下转换。
<!--more-->
首先是转换函数，输入源文件地址以目标文件地址，地址均为完整文件路径。调用一次api进行源文件的转换，就需要向OpenOffice服务进程新建一个连接，转换完毕后手动关闭连接。
{%codeblock lang:java%}
public static int office2PDF(String sourceFile, String destFile) {
        try {
            File inputFile = new File(sourceFile);
            if (!inputFile.exists()) {
                return -1;
            }

            // 如果目标路径不存在, 则新建该路径
            File outputFile = new File(destFile);
            if (!outputFile.getParentFile().exists()) {
                outputFile.getParentFile().mkdirs();
            }

            // 在本地8100端口创建新的连接实体
            OpenOfficeConnection connection = new SocketOpenOfficeConnection("127.0.0.1", 8100);
            connection.connect();

            // 转换
            DocumentConverter converter = new OpenOfficeDocumentConverter(connection);
            converter.convert(inputFile, outputFile);
            
            // 关闭连接
            connection.disconnect();

            return 1;

        } catch (IOException e) {
            e.printStackTrace();
        }

        return 1;
    }
{%endcodeblock%}
在main函数中启动OpenOffice服务进程，通过`Runtime.getRuntime().exec()`来创建命令行进程。这里有一个问题需要注意的是，由于OpenOffice在首次安装启动时需要进行注册才能正常使用其全部功能，仅仅是单纯的注册不需要繁琐步骤和费用。如果安装后没有启动OpenOffice注册就进行文档转换会报错，为了便利，2.0以上版本官方提供了`-nofirststartwizard`启动参数用于跳过首次启动注册。
{%codeblock lang:java%}
public static void main(String[] args) throws Exception {
    	// 启动OpenOffice的服务
        String command = "C:\\Program Files (x86)\\OpenOffice 4\\program\\soffice.exe -headless -accept=\"socket,host=127.0.0.1,port=8100;urp;\" -nofirststartwizard";
        Process pro = Runtime.getRuntime().exec(command);

        String source = "D:\\documents\\word\\test.docx";
        String dest = "D:\\documents\\pdf\\test_t.pdf";
    	
    	// 开始转换
        int result = office2PDF(source, dest);   
        if(result == 1){
        	System.out.println("转换成功!");
        }else{
        	System.out.println("转换失败!");
        }

        // 在这里我们也可以将服务进程关闭
        // pro.destroy();
        // 而实际在服务器上，为起到响应转换请求的效果，OpenOffice进程需要常驻，所以无需关闭
    }
{%endcodeblock%}
最新的`jodconverter`是09年作者发布的2.2.2版本，可以在[sourceforge](https://sourceforge.net/projects/jodconverter/files/JODConverter/2.2.2/)找到它，下载zip压缩包解压，将lib文件夹下所有jar包导入项目即可。在完整程序中需要导入`jodconverter`的以下包：
{%codeblock lang:java%}
import com.artofsolving.jodconverter.DocumentConverter;
import com.artofsolving.jodconverter.openoffice.connection.OpenOfficeConnection;
import com.artofsolving.jodconverter.openoffice.connection.SocketOpenOfficeConnection;
import com.artofsolving.jodconverter.openoffice.converter.OpenOfficeDocumentConverter;
{%endcodeblock%}
基本转换功能已经实现，后期将逐渐考虑使用MEAN处理前台页面以及业务逻辑，java作为OpenOffice服务器，在小众使用范围下，nodejs与java服务器暂用http协议通信，对于通信效率更高的`websocket`以及`thrift`不做考虑，后期有精力再继续学习踩坑。