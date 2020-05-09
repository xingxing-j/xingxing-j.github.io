---
title: Servlet之文件上传
date: 2020-04-20 19:33:06
categories: 后端学习
tags:
- [Servlet]
- [文件上传]
---

用Servlet进行文件上传的两种方式

<!-- more -->

#### 文件上传三要素

1. 请求方式：Post
2. 页面必须有**\<input type="file"/>**
3. 请求的enctype必须为multipaart/form-data

#### 文件上传方式一

该方式依赖两个包，commons -io.jar  commons-fileupload.jar

示例代码 

```java
@WebServlet("/fileUpload")
public class FileUploadServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        req.setCharacterEncoding("utf-8");
        DiskFileItemFactory factory = new DiskFileItemFactory();
        ServletFileUpload fileUpload = new ServletFileUpload(factory);

        // 拿真实路径？
        String realPath = req.getServletContext().getRealPath("/upload");
        System.out.println(realPath);
        // 在服务器里创建文件夹？
        File file = new File(realPath);
        if (!file.exists()) {
            file.mkdirs();
        }

        try {
            List<FileItem> list = fileUpload.parseRequest(req);
            for (FileItem item : list) {
//                System.out.println(item.getString());
//                System.out.println(item.getName());
				// isFormField判断是否为文件，是返false,否返true
                if (!item.isFormField()) {
                    item.write(new File(realPath, item.getName()));
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

#### 文件上传方式二

该方式依赖于Servlet3.0，并结合了ajax

##### 前端代码

```html
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>异步上传</title>
    <script src="jquery-1.12.4.min.js"></script>
</head>
<body>
    <img src="" alt=""/>
    <br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/>
    <form>
        <input type="file" name="xxx" id="img"/>
    </form>

    <button>上传</button>
    <script>
        $(function () {
            $("button").on("click", function () {
                // 获取表单数据对象
                var formData = new FormData();
                // 向表单数据对象中添加上传的文件对象
                formData.append("img", $("#img")[0].files[0]);
                // 用ajax进行异步提交
                $.ajax({
                    url: "/fileUpload2",
                    type: "post",
                    data: formData,
                    contentType: false,
                    processData: false,
                    success: function (resp) {
                        $("img").attr("src", resp);
                    }
                })
            })
        })
    </script>
</body>
</html>
```

##### 后端代码

```java
@WebServlet("/fileUpload2")
@MultipartConfig// 需要此注解
public class FileUploadServlet2 extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        // 1.解决编码问题
        req.setCharacterEncoding("utf-8");
        resp.setContentType("text/html;charset=utf-8");
        // 2.将上传内容封装到part对象中
        // ***此处getPart()的 参数 为formData.append()里添加的 属性名***
        Part part = req.getPart("img");
        // 3.拿真实路径？选择文件存放位置？
        String realPath = req.getServletContext().getRealPath("/upload");
        // 4.获取上传的文件名
        String fileName = part.getSubmittedFileName();
        System.out.println(fileName);
        // 5.获取文件后缀名
        String suffixName = fileName.substring(fileName.lastIndexOf("."));
        // 6.拼接UUID获取后缀得新文件名
        String newFileName = UUID.randomUUID().toString() + suffixName;
        System.out.println(newFileName);
        // 7.通过part写出
        part.write(realPath + "/" + newFileName);
        // 8.通过响应向前端返回上传文件的位置
        resp.getWriter().write("/upload/" + newFileName);
    }
}
```

