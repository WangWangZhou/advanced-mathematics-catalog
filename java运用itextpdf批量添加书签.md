
java运用itextpdf批量添加书签
带太多书不方便，但是电子书又普遍没有目录，这里开发了一个小工具
用于生成书签。这个过程参考了很多大神们的资料。
这里运用的工具是itextpdf,这个项目可以在windos下使用，也可以在Linux下使用。

# 目录文件
```
目录 7
第一章 概率论的基本概念 1
随机试验 1
样本空间、随机事件 2
频率与概率 5
等可能概型(古典概型) 9
条件概率 14
独立性 23
小结 23
习题 24

第二章 随机变量及其分布 30
第三章 多维随机变量及其分布 60
第四章 随机变量的数字特征 90
第五章 大数定律及中心极限定理 119
第六章 样本及抽样分布 128
第七章 参数估计 149
第八章 假设检验 178
第九章 方差分析及回归分析 224
第十章 bootstrap方法 270
第十一章 在数理统计中应用Excel软件 282
第十二章 随机过程及其统计描述 300
第十三章 马尔可夫链 319
第十四章 平稳随机过程 335
第十五章 选做习题 363


```
# 代码文件

```java
package mark;

import com.itextpdf.text.Document;
import com.itextpdf.text.DocumentException;
import com.itextpdf.text.pdf.*;
import java.io.BufferedReader;
import java.io.FileOutputStream;
import java.io.FileReader;
import java.io.IOException;
import java.util.*;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class GenerateBookmarkUtil {

    private Pattern spacePattern = Pattern.compile("\\s*");

    public static void main(String[] args) {
        GenerateBookmarkUtil zcc = new GenerateBookmarkUtil();
        String sourceTextPath = "/media/11/目录.txt";//生成的目录文本路径
        String sourceDocPath = "/media/11/概率论与数理统计.pdf"; //电子书路径
        String desFilename = "/media/11/概率论与数理统计目录.pdf"; // 生成的带有目录的文件的路径（问价是复制，不是覆盖）
			
        try {
            zcc.createPdf(sourceTextPath, sourceDocPath, desFilename,10);
        	//zcc.getPdf(7, 10, sourceDocPath, desFilename);
        } catch (DocumentException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    /**
     * 获取目录pdf文件用于OCR识别
     * @param startPage 开始页码
     * @param endPage 结束页码
     * @param sPdfPath 需要生成书签的文件
     * @param filename 生成书签后的文件路径
     * @throws DocumentException
     * @throws IOException
     */
    public void getPdf(int startPage,int endPage,String sPdfPath, String filename) throws DocumentException, IOException {
        //可以创建新的文件，也可以直接操作源文件，添加书签
        // step 1
        Document document = new Document();
        // step 2 输出文件
        PdfCopy writer = new PdfCopy(document, new FileOutputStream(filename)); // step 3
        writer.setViewerPreferences(PdfWriter.PageModeUseOutlines);//设置打开pdf文件时显示书签
        document.open();
        // step 4 逐页读入pdf文件并写入输出文件
        PdfReader reader = new PdfReader(sPdfPath);
        int n = reader.getNumberOfPages();
        for (int page = startPage; page <= endPage; page++) {
            writer.addPage(writer.getImportedPage(reader, page));
        }
        writer.freeReader(reader);
        document.close();
        
        System.out.println("生成成功");
    }
	
	/**
	 * 根据txt目录文件生成带目录的pdf文件
	*@param sTxtPath 书签文件的路径
	*@param sPdfPath 需要生成书签的文件
	*@param filename  生成书签后的文件路径
	*@param startPage  正文的实际页数-1
	*/
    public void createPdf(String sTxtPath, String sPdfPath, String filename,int startPage) throws DocumentException, IOException {
		
		//if(startPage>0)
		//	startPage--;
		
        //可以创建新的文件，也可以直接操作源文件，添加书签
        // step 1
        Document document = new Document();
        // step 2 输出文件
        PdfCopy writer = new PdfCopy(document, new FileOutputStream(filename)); // step 3
        writer.setViewerPreferences(PdfWriter.PageModeUseOutlines);//设置打开pdf文件时显示书签
        document.open();
        // step 4 逐页读入pdf文件并写入输出文件
        PdfReader reader = new PdfReader(sPdfPath);
        int n = reader.getNumberOfPages();
        for (int page = 1; page <= n; page++) {
            writer.addPage(writer.getImportedPage(reader, page));
        }
        writer.freeReader(reader);
        // step 5 添加书签
        PdfOutline root = writer.getRootOutline();
        PdfAction action;//标识书签点击后的跳转动作，通过它设置跳转的页码


        try {
            //读入保存书签的TXT文件，分拆为书签名及跳转页码；
            BufferedReader bufRead = new BufferedReader(new FileReader(sTxtPath));
            String str;

            String[] ss = null;
            int skipPage = startPage; //正文书签跳过的页数

            /*数据的存储结构
             *
             * page 页码
             *title 标题
             * kids 子书签   page,title ,kids
             */
            List<Map<String, Object>> outlines = new ArrayList<>();//存放解析的数据

            String pageNumStr = null;
            int pageNum = 0;
			
			//遍历读取目录文件，每次读取一行，并解析出该行的标题和页码值	
            while ((str = bufRead.readLine()) != null) {
                String title = "";
                int parentPage = 0;
                String childTitle = "";
                int childPage = 0;
                str = str.trim();

				//过滤空行
                Matcher matcher = spacePattern.matcher(str);
                if (matcher.matches()) {
                    continue;
                }
                
				//获取页码
                ss = str.split(" ");
                pageNumStr = ss[ss.length - 1].trim();

                //不包含页码的标题
                for (int j = 0; j < ss.length - 1; j++) {
                    title += ss[j] + "  ";
                }
                
                if (pageNumStr.matches("\\d+")) {//存在页码
                    try {
                        pageNum = Integer.valueOf(pageNumStr);
                        pageNum += skipPage;
                    } catch (Exception e) {
                    }
                    String category="目录";
                    String sectionNumberStr = ss[0]; //获取标题的序号
                    if (sectionNumberStr.contains("章")) {//章节
                        List<Map<String, Object>> kids = null;
                        kids = new ArrayList();
                        Map<String, Object> chapterMap = new HashMap<>();
                        chapterMap.put("title", title);
                        chapterMap.put("page", pageNum);
                        chapterMap.put("kids", kids);
                        outlines.add(chapterMap);
                    }else if("目录".equals(sectionNumberStr)){
                        //目录书签
                        new PdfOutline(root,
                                PdfAction.gotoLocalPage(pageNum-skipPage, new PdfDestination(PdfDestination.FIT), writer), "目   录");
                    }else {//小节
                        List<Map<String, Object>> kids = (List<Map<String, Object>>) outlines.get(outlines.size() - 1).get("kids");//将小节添追加到自书签列表
                        Map<String, Object> littleChapter = new HashMap<>();
                        littleChapter.put("title", title);
                        littleChapter.put("page", pageNum);
                        littleChapter.put("kids", new ArrayList<>());
                        kids.add(littleChapter);
                    }
                } else {//不存在页码，可能是大标题 或者目录   例如：第一部分 核心实现
                    Map<String, Object> bigSection = new HashMap<>();
                    bigSection.put("title", title);
                    bigSection.put("page", -1);
                    bigSection.put("kids", new ArrayList<>());
                    outlines.add(bigSection);
                }

            }


            //数据遍历，添加书签
            System.out.println(outlines);
            outlines.forEach(map -> {
                String sectionTitle = (String) map.get("title");
                int page = (int) map.get("page");
                List<Map<String, Object>> kids = (List<Map<String, Object>>) map.get("kids");
                PdfOutline sectionOutline = null;
                if (page < 0) {//该部分是大类别
                    int parentPage = (int) kids.get(0).get("page") - 1; //获取大分类的页数，一般就是子目录页数减一
                    PdfAction action1 = PdfAction.gotoLocalPage(parentPage,
                            new PdfDestination(PdfDestination.FIT), writer);//设置书签动作
                    sectionOutline = new PdfOutline(root, action1, sectionTitle, false); //大分类的
                } else {
                    PdfAction action1 = PdfAction.gotoLocalPage(page,
                            new PdfDestination(PdfDestination.FIT), writer);//设置书签动作
                    sectionOutline = new PdfOutline(root, action1, sectionTitle, false); //一级章节标题;
                }
                for (Map<String, Object> kid : kids) { //一级章节
                    String firstTitle = (String) kid.get("title");
                    int firstPage = (int) kid.get("page");
                    List<Map<String, Object>> firstKids = (List<Map<String, Object>>) kid.get("kids");
                    PdfAction action2 = PdfAction.gotoLocalPage(firstPage,
                            new PdfDestination(PdfDestination.FIT), writer);//设置书签动作
                    PdfOutline firstOutline = new PdfOutline(sectionOutline, action2, firstTitle, false); //一级标题的outline

                    for (Map<String, Object> secondKid : firstKids) {//二级章节
                        String secondTitle = (String) kid.get("title");
                        int secondPage = (int) kid.get("page");
                        List<Map<String, Object>> secondKids = (List<Map<String, Object>>) kid.get("kids");
                        PdfAction action3 = PdfAction.gotoLocalPage(secondPage,
                                new PdfDestination(PdfDestination.FIT), writer);//设置书签动作
                        PdfOutline secondOutline = new PdfOutline(firstOutline, action2, secondTitle, false); //二级标题的outline
                    }
                }
            });

        } catch (IOException ioe) {
        }
        document.close();
        System.out.println("添加书签成功");
    }


}
```


