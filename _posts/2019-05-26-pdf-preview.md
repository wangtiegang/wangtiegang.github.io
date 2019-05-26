---
layout:     post
title:      "实现pdf在线预览"
subtitle:   " \"实现pdf在线预览\""
date:       2019-05-26 17:00:00
author:     "wangtiegang"
header-img: ""
catalog: true
tags:
    - 前端
    - react
---

> 最近两个项目都有Pdf、Word、Excel、Ppt在线预览的需求，其中Office相关的文件也是转换成Pdf的方式进行预览。一个项目前端使用jquery技术栈，另外一个使用react技术栈，在这上面花了不少时间，做些记录，方便下次查询。



### jquery项目pdf预览

> **在网上了解了一下之后，发现有如下几种方式可以实现Pdf预览**

<table>
  <tr>
    <th>方式</th><th>简单说明</th>
  </tr>
  <tr>
    <td>使用PDFObject.js框架</td>
    <td>使用简单，内部使用 <code>&lt;embed&gt;</code> 标签实现pdf预览，需要浏览器支持。</td>
  </tr>
  <tr>
    <td>使用PDF.js框架</td>
    <td>将pdf解析渲染成canvas，兼容性强，在pc和移动端都能实现预览，但是加载速度慢。</td>
  </tr>
  <tr>
    <td>使用 <code>&lt;embed&gt;</code> 标签</td>
    <td>使用简单，主流浏览器都支持，不同浏览器效果有差异。</td>
  </tr>
  <tr>
    <td>使用 <code>&lt;iframe&gt;</code> 标签</td>
    <td>使用简单，主流浏览器都支持，不同浏览器效果有差异。</td>
  </tr>
  <tr>
    <td>使用 <code>&lt;object&gt;</code> 标签</td>
    <td>使用简单，主流浏览器都支持，不同浏览器效果有差异。</td>
  </tr>
</table>



> **由于是内部Web项目，用户少且容忍度高，不需要考虑各浏览器兼容及移动端浏览等情况，所以简单了解后选定PDFObject.js方式，指定使用Chrome浏览器。**

* 1. 前端引入PDFObject.js文件。

     ```html
     <!-- 引入pdfobject -->
     <script src="${base.contextPath}/lib/pdf-object/pdfobject.min.js"></script>
     
     <body>
         <div id="pdf"></div>
         <script>
             // 加载pdf
             PDFObject.embed("/file/preview?fileId=" + fileId, "#pdf");
         </script>
     
         <style>
          /** 自定义预览样式 */
          .pdfobject-container { 
               overflow: auto; 
               position: absolute; 
               top: 0; 
               right: 0; 
               bottom: 0; 
               left: 0; 
               width: 100%; 
               height: 100%; 
               overflow-x: hidden; 
               overflow-y: hidden;
           }
         </style>
     </body>
     ```

     

* 2. 后端返回pdf文件流。

     ```java
     public void preview(HttpServletResponse response, Long fileId, String userAgent){
       // do something
       File file = new File(filePath);
       if(file.exists()){
         InputStream in = null;
         OutputStream out = null;
         	try{
           	response.reset();
             if(StringUtils.contains(userAgent, "Mozilla")){
             	//google,火狐浏览器
               fileName = new String(fileName.getBytes("gb2312"), "ISO-8859-1");
             }else{
               //其他浏览器
               String fileNameEncode = URLEncoder.encode(fileName,"UTF8");
               fileName = new String((fileNameEncode).getBytes("UTF-8"), "UTF8");
             }
             //inline 表示回复中的消息体会以页面的一部分或者整个页面的形式展示
             //attachment 意味着消息体应该被下载到本地；大多数浏览器会呈现一个“保存为”的对话框
             response.setHeader("Content-Disposition", "inline;filename=\"" 
                                + fileName + "\"");
             response.setContentType("application/pdf;charset=UTF-8");
             in = new FileInputStream(file.getAbsolutePath());
             out = response.getOutputStream();
             response.setContentLength(in.available());
             byte[] b = new byte[1024];
             int len;
             while((len = in.read(b))!=-1){
                out.write(b, 0, len);
             }
             out.flush();
             } catch (IOException e) {
                     // log
             }
       }
     }
     ```
