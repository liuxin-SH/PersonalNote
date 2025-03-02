# java处理图片的当时
## 读取图片的
- 1. 方法一: 通过 java.awt.Toolkit 工具类来读取本地、网络 或 内存中 的 图片（支持 GIF、JPEG 或 PNG）

Image image = Toolkit.getDefaultToolkit().getImage(String filename);
Image image = Toolkit.getDefaultToolkit().getImage(URL url);
Image image = Toolkit.getDefaultToolkit().createImage(byte[] imageData);

- 2. 方法二: 通过 javax.imageio.ImageIO 工具类读取本地、网络 或 内存中 的 图片（BufferedImage 继承自 Image）

BufferedImage bufImage = ImageIO.read(File input);
BufferedImage bufImage = ImageIO.read(URL input);
BufferedImage bufImage = ImageIO.read(InputStream input);

// 创建空图片
BufferedImage bufImage = new BufferedImage(int width, int height, int imageType);

PS: 推荐使用第二种读取图片的方式，读取后获得的 BufferedImage 有更丰富的 API 对图片进行相关操作。

# 工具 开源库

## freemarker
```maven
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.29</version>
</dependency>
```
FreeMarker是一款模板引擎： 即一种基于模板和要改变的数据， 并用来生成输出文本（HTML网页、电子邮件、配置文件、源代码等）的通用工具。 它不是面向最终用户的，而是一个Java类库，是一款程序员可以嵌入他们所开发产品的组件。