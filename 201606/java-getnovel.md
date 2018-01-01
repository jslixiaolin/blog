# java爬虫之下载txt小说

最近迷上了天蚕土豆写的《大主宰》这本玄幻小说，无奈找不到下载链接。于是就萌生了自己爬取小说章节的想法，代码其实很简单，主要在于分析网页结构、正则匹配以及文件保存.

## 1. 分析网页结构
爬取小说主要需要爬取章节、正文，以及能保证爬取到所有的章节。以《大主宰》为例，其网页结构如下：

![](http://7xryc7.com1.z0.glb.clouddn.com/2.png)

可以看到小说正文包含在一个id为content的div里，这极大的帮助了我们的爬取.章节名称保存在一个名为readtitle的js变量中.下一页的url位于a标签的href属性中

## 2. 正则匹配
通过分析网页结构，可以得到如下正则表达式：
* 章节名： `readtitle = "(.+?)"`
* 正文： `<div id="content">(.+?)</div>`
* 下一章：`&rarr; <a href="(.+?)">`
如果当前章节是最新章节，那么其下一章的url是以 / 开头的，我们可以根据这个判断章节是否是最新章节.

## 3. 运行结果
以爬取《大主宰》为例，截至至6月30号，其最新章节为`第一千两百五十四章 最后的赢家`，爬取其小说内容，保存为txt文件，测试结果如下：

![](http://7xryc7.com1.z0.glb.clouddn.com/3.png)

## 4. 附录

你可以通过`jslinxiaoli@foxmail.com`联系我.
欢迎在[github](https://github.com/jslixiaolin)或者[知乎](http://www.zhihu.com/people/li-xiao-lin-4)上关注我 ^_^.
也可以访问个人网站: https://jslixiaolin.github.io

源代码如下：
```
package com.lxl.txt;

import java.io.*;
import java.net.URL;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

/**
 * 爬取小说txt文件
 *
 * @author lixiaolin
 * @createDate 2016-06-28 19:38
 */
public class NovelGet {
    public static void main(String[] args) {
        String baseUrl = "http://www.00ksw.com/html/1/1343/";
        String nextUrl = "597361.html";
        String destFilePath = "D:\\test.txt";
        System.out.println("开始爬取数据...");
        long startTime = System.currentTimeMillis();
        getNovel(baseUrl, nextUrl, destFilePath);
        long endTime = System.currentTimeMillis();
        System.out.println("用时 " + (endTime - startTime) / 1000 + "秒...");
    }

    public static void getNovel(String baseUrl, String nextUrl, String destFilePath) {
        try {
            File destFile = new File(destFilePath);
            // 目标文件存在则删除
            if (!destFile.exists()) {
                destFile.delete();
            }
            destFile.createNewFile();

            FileWriter fw = new FileWriter(destFile);
            String realUrl, resultContent;
            StringBuffer sb = new StringBuffer();
            BufferedReader br = null;
            // 正文正则匹配表达式
            Pattern contentPat = Pattern.compile("<div id=\"content\">(.+?)</div>");
            // 标题正则表达式
            Pattern titlePat = Pattern.compile("readtitle = \"(.+?)\"");
            // 下一章正则表达式
            Pattern nextPat = Pattern.compile("&rarr; <a href=\"(.+?)\">");
            Matcher matcher;
            // 下一张的url以 / 开头则是最新章节
            while (!nextUrl.startsWith("/")) {
                realUrl = baseUrl + nextUrl;
                // 获取目标url的response
                br = new BufferedReader(new InputStreamReader(new URL(realUrl).openStream()));
                String line;
                while ((line = br.readLine()) != null) {
                    sb.append(line);
                }
                // 替换空格和换行符
                resultContent = sb.toString().replaceAll("&nbsp;", "").replaceAll("<br /><br />", "*****");
                // 换行
                fw.write("\r\n\r\n");
                // 匹配标题
                matcher = titlePat.matcher(resultContent);
                if (matcher.find()) {
                    fw.write(matcher.group(1));
                }
                fw.write("\r\n");
                // 匹配正文
                matcher = contentPat.matcher(resultContent);
                if (matcher.find()) {
                    fw.write(matcher.group(1));
                }
                // 匹配下一页url
                matcher = nextPat.matcher(resultContent);
                if (matcher.find()) {
                    nextUrl = matcher.group(1);
                }
                // 清空
                sb.delete(0, sb.length());
            }
            if (br != null) {
                br.close();
            }
            fw.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```



