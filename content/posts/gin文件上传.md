---
title: "gin~文件上传"
date: 2023-03-31T10:01:23+08:00
tags : [                                    
     "Go Web",
 ]
categories : [                              
     "Go语言",
 ]
author: "博文"  
---

> 🙃上传文件是很常用的功能，oa，办公，财务，云平台，都会涉及上传表格，图片，镜像等信息。

# 单文件上传

```go
func TestUpload(t *testing.T) {
	r := gin.Default()
	//加载模板
	r.LoadHTMLFiles("upload.html")
	//请求上传文件页面
	r.GET("/upload", func(c *gin.Context) {
		c.HTML(http.StatusOK, "upload.html", nil)
	})

	r.POST("/upload", func(c *gin.Context) {
		file, err := c.FormFile("fl")
		if err != nil {
			c.JSON(http.StatusInternalServerError, gin.H{
				"message": err.Error(),
			})
			return
		}
		log.Println(file.Filename)
		//这里存到了本地，在实际项目中，更多的是存到了对象存储，例如mino，阿里云之类的;
		dst := fmt.Sprintf("D:/BaiduNetdiskDownload/%s", file.Filename)
		// 上传文件到指定目录
		c.SaveUploadedFile(file, dst)
		c.JSON(http.StatusOK, gin.H{
			"message": fmt.Sprintf("'%s' uploaded!", file.Filename),
		})
	})
	r.Run()
}
```

这个demo是我去年学习Go的时候，参考李文周的视频和博客写的，注意点

- html里面`<input type="file" name="fl">` , 是fl，不是f1，不要弄混了。
- 不要加`multiple`,这是单文件上
- postman无法测试，这个要通过浏览器测试

![](/文件上传/20230401102220.png)

# 多文件上传

```go
func TestUploadMulti(t *testing.T) {
	r := gin.Default()

	// 加载模板
	r.LoadHTMLFiles("upload.html")
	// 上传文件
	r.GET("/upload", func(c *gin.Context) {
		c.HTML(http.StatusOK, "upload.html", nil)
	})

	r.POST("/upload", func(c *gin.Context) {
		// 多文件上传
		form, _ := c.MultipartForm()
		files := form.File["file"]

		for index, file := range files {
			log.Println(file.Filename)
			dst := fmt.Sprintf("D:/BaiduNetdiskDownload/%s_%d", file.Filename, index)
			// 上传文件到指定目录
			c.SaveUploadedFile(file, dst)
		}
		c.JSON(http.StatusOK, gin.H{
			"message": fmt.Sprintf("%d files uploaded!", len(files)),
		})
	})
	r.Run()
}

```

多文件上传，注意

- html里`<input type="file" name="fl  multiple`" 要有多选这个选项
- 可以用postman测试, 注意key写file

![](/文件上传/20230401104116.png)

# 小结

- 文件上传是基本且重要的功能，要结合后面实践使用
- 需要学习一下源码，是如何设计的