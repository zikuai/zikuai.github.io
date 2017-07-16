---
layout: post
title: nodejs renameSync 重命名报错
categories: blog
description: nodejs调用fs.renameSync
keywords: node file upload unlinkSync
---

## nodejs 文件上传 调用fs.renameSync 报错

报错信息如下：

    Error: EXDEV: cross-device link not permitted, rename 'C:\Users\ADMINI~1\AppData\Local\Temp\upload_e9d50a899c1ff90c476f93de18bf0b3e' -> 'D:\WebStorm-project\node-file-upload-xzk\tmp\test.png'
    at Object.fs.renameSync (fs.js:733:18)
    at form.parse (D:\WebStorm-project\node-file-upload-xzk\requestHandlers.js:32:12)
    at IncomingForm.<anonymous> (D:\WebStorm-project\node-file-upload-xzk\node_modules\formidable\lib\incoming_form.js:105:9)
    at emitNone (events.js:86:13)
    at IncomingForm.emit (events.js:185:7)
    at IncomingForm._maybeEnd (D:\WebStorm-project\node-file-upload-xzk\node_modules\formidable\lib\incoming_form.js:553:8)
    at D:\WebStorm-project\node-file-upload-xzk\node_modules\formidable\lib\incoming_form.js:230:12
    at WriteStream.<anonymous> (D:\WebStorm-project\node-file-upload-xzk\node_modules\formidable\lib\file.js:74:5)
    at WriteStream.g (events.js:291:16)
    at emitNone (events.js:91:20)


出错代码：

    function upload(response, request) {
	    let form = new formidable.IncomingForm();
	
	    form.parse(request, (error, fields, files) => {
	
	        fs.renameSync(files.upload.path, "./tmp/test.png");
	
	        response.writeHead(200, {
	            "Content-Type": "text/html"
	        });
	        response.write("received image:<br/>");
	        response.write("<img src='/show' />");
	        response.end();
	    });
    }

大概意思就是跨磁盘分区重命名无权限，网上查了一下信息，1.0版本之前是可以的

## 解决方法

#### 1.上传前定义一个临时目录form.uploadDir='tmp'

    function upload(response, request) {
	    let form = new formidable.IncomingForm();
	
	    //解决重命名不能夸磁盘分区移动问题
	    form.uploadDir='tmp';
	
	    form.parse(request, (error, fields, files) => {
	
	        fs.renameSync(files.upload.path, "./tmp/test.png");
	
	        response.writeHead(200, {
	            "Content-Type": "text/html"
	        });
	        response.write("received image:<br/>");
	        response.write("<img src='/show' />");
	        response.end();
	    });
    }


#### 2.读取文件流的形式，主要利用fs的createReadStream、createWriteSream和unlinkSync方法

    function upload(response, request) {
    	let form = new formidable.IncomingForm();
	
	    form.parse(request, (error, fields, files) => {
	
	        //解决重命名不能夸磁盘分区移动问题
	        let readStream = fs.createReadStream(files.upload.path);
	        let writeStream = fs.createWriteStream('./tmp/test.png');
	
	        readStream.pipe(writeStream);
	
	        readStream.on('end', () => {
	            fs.unlinkSync(files.upload.path);
	        });
	
	        response.writeHead(200, {
	            "Content-Type": "text/html"
	        });
	        response.write("received image:<br/>");
	        response.write("<img src='/show' />");
	        response.end();
	    });
    }

    