# PDF文件转图片


> 项目中有将office文件第一页保存缩略图的需求，恰好工程中使用Tika解析文本用到了PDFBox，通过PDFBox将PDF转图片的实现

## 依赖

```xml
<dependency>
    <groupId>org.apache.pdfbox</groupId>
    <artifactId>pdfbox</artifactId>
    <version>2.0.16</version>
</dependency>
```


## 代码清单

```java
@Slf4j
public class PDFUtils {
    public static final float DEFAULT_DPI = 300;  // <1>
    public static final String DEFAULT_FORMAT = "jpg";
    
    public static void renderFirstPage2Img(String pdfRealPath, String filename){
        renderFirstPage2Img(pdfRealPath,filename,DEFAULT_DPI);
    }

    public static void renderFirstPage2Img(String pdfRealPath, String filename, float dpi){
        File outFile = new File(filename);
        outFile.getParentFile().mkdirs();
        FileInputStream in = null;
        FileOutputStream out = null;
        try {
            in = new FileInputStream(new File(pdfRealPath));
            out = new FileOutputStream(outFile);
            renderFirstPage2Img(in, out, dpi);
        } catch (IOException e) {
            log.error("首页转图片异常：{}", e.getMessage());
        } finally {
            if(in != null){
                try {
                    in.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if(out != null){
                try {
                    out.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void renderFirstPage2Img(InputStream in, OutputStream out){
        renderFirstPage2Img(in, out, DEFAULT_DPI);
    }

    public static void renderFirstPage2Img(InputStream in, OutputStream out, float dpi){
        System.setProperty("sun.java2d.cmm","sun.java2d.cmm.kcms.KcmsServiceProvider");
        try {
            PDDocument document = PDDocument.load(in);
            PDFRenderer pdfRenderer = new PDFRenderer(document);
            // 需要使用DPI为300
            BufferedImage image = pdfRenderer.renderImageWithDPI(0, dpi, ImageType.RGB);
            document.close();
            ImageIO.write(image, DEFAULT_FORMAT, out);
        } catch (IOException e) {
            log.error("首页转图片异常：{}", e.getMessage());
        }
    }

    public static String getBase64(InputStream in){
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        renderFirstPage2Img(in, out, DEFAULT_DPI);
        byte[] bytes = out.toByteArray();
        String base64String = Base64.encodeBase64String(bytes);
        log.info("文件:"+base64String);
        return base64String;
    }


    /**
     * 文档全部页
     */
    public static void renderAllPage2Img(String pdfRealPath, String outPath){ // <2>
        try {
            System.setProperty("sun.java2d.cmm","sun.java2d.cmm.kcms.KcmsServiceProvider");
            // 图像合并使用参数
            // 总宽度
            int width = 0;
            // 保存一张图片中的RGB数据
            int[] singleImgRGB;
            int shiftHeight = 0;
            //保存每张图片的像素值
            BufferedImage imageResult = null;
            //利用PdfBox生成图像
            PDDocument pdDocument = PDDocument.load(new File(pdfRealPath));
            PDFRenderer renderer = new PDFRenderer(pdDocument);
            //循环每个页码
            for (int i = 0, len = pdDocument.getNumberOfPages(); i < len; i++) {
                BufferedImage image = renderer.renderImageWithDPI(i, DEFAULT_DPI, ImageType.RGB);
                int imageHeight = image.getHeight();
                int imageWidth = image.getWidth();
                //计算高度和偏移量
                if (i == 0) {
                    //使用第一张图片宽度;
                    width = imageWidth;
                    //保存每页图片的像素值
                    imageResult = new BufferedImage(width, imageHeight * len, BufferedImage.TYPE_INT_RGB);
                } else {
                    // 计算偏移高度
                    shiftHeight += imageHeight;
                }
                singleImgRGB = image.getRGB(0, 0, width, imageHeight, null, 0, width);
                // 写入流中
                imageResult.setRGB(0, shiftHeight, width, imageHeight, singleImgRGB, 0, width);
            }
            pdDocument.close();
            // 写图片
            ImageIO.write(imageResult, DEFAULT_FORMAT, new File(outPath));
        } catch (Exception e) {
            log.error("PDF转图片失败：{}",e.getMessage());
        }
    }
}
```

<1> 经过测试,300显示效果较为清晰,体积稳定,dpi越高图片体积越大,一般电脑显示分辨率为96
<2> 遍历整篇文档的所有页面，转换为长图的实现

代码参考:
[bibliography]
- [[[CSDN刘富国]]] https://blog.csdn.net/u014155085/article/details/86077614[使用pdfBox实现pdf转图片，解决中文方块乱码等问题]