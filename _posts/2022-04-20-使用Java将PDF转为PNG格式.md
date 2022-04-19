---
layout: post
title: "使用Java将PDF转为PNG格式"
subtitle: convert pdf to png by java
categories: java
tags: [java , png, pdf]
banner: "/assets/images/cover/2022-04-20-glow.png"
---

在某些情况下，我们常常需要将PDF格式的文件转为图片格式，如PNG，这时候使用pdfbox就很容易实现这一功能。

<!--more-->

> 以下代码主要是使用了pdfbox这个第三方库，关于PDF转为图片，网页上有很多免费的网站，但是这些网站往往都需要上传文件，有泄漏文件内容的风险，WPS内部也有PDF转图片的选项，但是需要付费才能使用，因此使用下面的代码，实现PDF的无损转换。
>
> 其实对于PDF文件，如果对转换后的图片没有太高的质量要求，完全可以使用截图软件进行截图

## 1. Gradle引入依赖

```gradle
//PDF box
implementation 'org.apache.pdfbox:pdfbox:2.0.25'
```

## 2. 实现代码

```java
package com.flywinter;
import org.apache.pdfbox.pdmodel.PDDocument;
import org.apache.pdfbox.rendering.PDFRenderer;
import javax.imageio.ImageIO;
import java.io.File;
import java.io.IOException;

/**
 * Created by IntelliJ IDEA
 * User:Zhang Xingkun
 * Date:2022/4/15 17:58
 * Description:
 */
public class PdfToImage {
    public static void main(String[] args) throws IOException {
        final var filePath = "E:\\study\\Code\\Demo\\water-heat-pipe-pressure-loss-2.pdf";
        convertToPNG(filePath);
    }

    private static void convertToPNG(String filePath) throws IOException {
        final var targetFile = new File(filePath);
        final var imageType = "png";
        try (final var document = PDDocument.load(targetFile);
        ) {
            final var renderer = new PDFRenderer(document);
            final var pageSize = document.getNumberOfPages();
            for (int i = 0; i < pageSize; i++) {
                final var image = renderer.renderImage(i);
                final File outputPath = getOutputPath(targetFile, imageType, i + 1);
                ImageIO.write(image, imageType, outputPath);
            }
        }
    }

    private static File getOutputPath(File file, String imageType, int page) {
        final var fullName = file.getName();
        final var preName = new StringBuilder(fullName).substring(0, fullName.length() - 4);
        final var parentPath = file.getParent();
        return new File("%s%s%s_%d.%s".formatted(parentPath, File.separator, preName, page, imageType));
    }

}
```

关于上述代码，filePath为pdf文件的路径，运行该程序后，将会将PDF的每一页都转为`原文件名_页码.png`的格式，并存放在同级目录下。

上述代码只能转换一个pdf文件，可以根据自己需要进行修改，比如批量转换等
