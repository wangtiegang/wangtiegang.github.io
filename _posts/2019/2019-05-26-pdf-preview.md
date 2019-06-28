---
layout:     post
title:      "实现pdf在线预览"
subtitle:   " \"jquery，react预览pdf，office\""
date:       2019-05-26 17:00:00
author:     "wangtiegang"
header-img: ""
catalog: true
tags:
    - 前端
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

#### 前端引入PDFObject.js文件。

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

#### 后端返回pdf文件流。

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

> 使用此种方式最好是每次打开一个新的标签页，这样方便用户同时打开几个pdf文档，效果显示如下：

![post-pdf-preview](/img/in-post/2019-05/post-pdf-preview-1.png)


### React项目pdf预览

#### 使用react-pdf组件

  > 本项目使用react + ant design pro + springboot开发，前后端分离。由于刚接触react，不知道怎么集成PDFObject.js，隐约感觉也不太适合，于是github找到了一个react的开源组件： [react-pdf](https://github.com/wojtekmaj/react-pdf) ，需要注意的是，还有一个star数更多的同名库，那个是在线创建pdf的组件，不能搞错了。

  > 按照官方教程进程demo测试，使用方式非常简单，显示也很正常，但是问题来了，只要对pdf的内容进行选中，就会发现选中的文字是错位的，无法忍受。

  ![image-20190526185023094](/img/in-post/2019-05/post-pdf-preview-2.png)

  

  > 各种查询之后，发现组件最新发布的版本Bug fixs里写了已经修复了这个bug
  >
  > * Fixed text layer vertically misaligned in some cases ([#332](https://github.com/wojtekmaj/react-pdf/issues/332)).

  > 由于react-pdf是移植于pdf.js项目的，所以在将pdf渲染成canvas后还无法选取内容，这个时候在canvas之上再渲染一个透明的text layer层实现选取复制。遗憾的是，我使用的是最新版本，但依然存在这个问题，进入组件的官网demo，发现存在相同的问题。由于解决不了，只能放弃使用这个组件，目前还没找到合适的react组件。

#### 使用``` <embed> ```标签实现pdf预览

  > 本前端龙套经过各种挣扎之后，突然意识到为什么不直接在react中使用``` <embed> ```、``` <frame> ``` 、``` <object> ``` 标签呢？于是尝试了一下。

##### 前端直接使用 ``` <embed> ``` 标签

  ```react
    render() {
      return ({detail.previewStatus === 'playable' && detail.src ? (
        <embed
          src={encodeURI(`/api/knowledge/preview?src=${detail.src}`)}
          width="100%"
          height={height}
        />) : (
          <Alert message="文档不支持预览，请下载后查看" type="info" showIcon />
      )});
    }
  ```

##### 后端直接返回pdf对象

  ```java
    @GetMapping(value = "/knowledge/preview", produces = "application/pdf")
    public FileSystemResource preview(String src) {
    	return new FileSystemResource(src);
    }
  ```

##### 实现效果

  ![image-20190526194236017](/img/in-post/2019-05/post-pdf-preview-3.png)

  > **这个地方可以看到pdf的标题是乱码，各种折腾之后发现是个坑**

##### pdf标题乱码

  > 在发现标题乱码的问题之后，第一反应是因为浏览器直接编码的问题导致，我后端直接返回spring 的 FileSystemResource 对象，没地方设置，于是改成了Jquery下读取文件流写入response的方式进行pdf对象返回，但是不起效果，各种尝试之后都解决不了。

  > 另外一种方式是将预览的标题给隐藏掉，这个不是必须的，但是查看了W3C文档后发现``` <embed> ``` 标签没有这个属性，换成 ``` <object> ``` 或 ``` <frame> ``` 标签也不行，但是找到可以隐藏整个工具栏的方法，在src属性链接的结尾加上参数可以对预览样式进行控制。
  >
  > ```html
  > <!-- #后面的参数控制pdf预览效果 -->
  > <embed src="http://URL_TO_PDF.com/pdf.pdf#toolbar=0&navpanes=0&scrollbar=0" width="425" height="425">
  > ```
  >
  > 更多参数参考 [Adobe参数官方文档](https://www.adobe.com/content/dam/acom/en/devnet/acrobat/pdfs/pdf_open_parameters.pdf)

  > 在纠结隐藏工具栏会导致跳页，旋转，打印按钮无法使用的时候，突然发现文档名称被我修改成英文之后，还是显示乱码，清除浏览器缓存后也还是乱码。于是下载pdf文档用软件打开，点击属性，发现被坑了，预览显示的是pdf的标题属性，并不是文件名，撤销隐藏工具栏，皆大欢喜。
  >
  > ![image-20190526202045234](/img/in-post/2019-05/post-pdf-preview-4.png)

### 关于使用OpenOffice将文档转成pdf

  > Word，Excel，Ppt预览的方案从一开始觉得最好是能在线打开，能跟本地打开一样查看。网上查询之后，大概了解了几种方式。

  > 第一种是直接使用微软或google的免费服务，将文档的url地址传入就行了，但前提是文档必须公网可访问的，由于文档的私密性，所以这种方案直接排除。

  > 第二种是使用各种第三方收费的服务，效果类似第一种，但是由于收费和文档托管的问题，也直接排除

  > 最后一种就是使用openoffice转pdf进行预览了，这种方式最符合要求，缺点就是需要自己安装服务，转换后的格式可能有点误差，由于具体实现不是我，此处只讲讲大概思路。
  >
  > * 服务器安装openoffice服务。
  > * 在用户上传office文档的时候，立即对文档进行pdf转换，如果要对并发进行控制，可以使用消息队列进行限制。
  > * 用户进行预览时，先读取文件转换状态，如果已经转化成功，则直接返回pdf文档，否则提示用户下载查看。

