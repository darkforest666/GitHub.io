---
layout: post
title: 文件上传组件DropZone
date: 208-03-25
categories: blog
tags: [module]
description: dropzone
---
<h2>本期内容：上传组件dropzone</h2>
<p>
	最近项目上需要使用上传组件，鉴于之前使用的上传组件比较low，采用了比较好看和灵活的。
	组件的中文地址为：http://wxb.github.io/dropzonejs.com.zh-CN/dropzonezh-CN/
</p>
<h3>组件的使用</h3>
<p>首先获取这个组件，命令行指向npm install dropzone，没有按照nmpm的朋友可以通过官网下载</p>
<p>在下载下的dist目录下，取到需要的js和css，分别为：dropzone.js ，dropzone.css,然后引入我们的页面 </p>
<p>同样也支持commonjs的require写法，大家可以去官网查看</p>
<h3>组件的初始化</h3>
<p>该组件支持传统的form表单提交和简单的div操作，下面是实例：</p>

<h4>form表单的方式</h4>

<pre>
	<form action="/file-upload" class="dropzone">
	  <div class="fallback">
	    <input name="file" type="file" multiple />
	  </div>
	</form>
</pre>
	<p>此为官网上给出的实例，不知道为何我在此被困惑了一天左右的时间，原因是后台无法根据name获得iuput的文件流。</p>
<p>之后百度了很多版本，发现取消fallback这个class，将input设置为hidden，那么可以正常从后台提取到文件流。</p>
<p>看来网上的东西不能随便copy，需要自己去摸索！！！！</p>
<p>tips:
	form标签一定要添加这个属性：enctype="multipart/form-data"
</p>
<p>上面的属性中，action代表路由，class代表引用组件的class（严重影响美观)</p>

<h4>div的方式</h4>
<p>
	<pre>
	<div id="myId" class="dropzone" style="width: 800px; height: 300px;">点我上传</div>
	</pre>
	人没有尝试这种效果,百度了很久的ajax异步上传，没有成功
</p>
<p>需要注意的是此种方法需要在初始化时，指定URL属性</p>
</blockquote>
<h3>如何在js初始化</h3>
<p>
<blockquote>
	<pre>
	var myDropzone = new Dropzone("#dropz", {
        url: "/admin/upload",//文件提交地址
        previewTemplate：‘’，//为HTML添加一个上传预览字符串
        method:"post",  //也可用put
        paramName:"file", //默认为file
        maxFiles:1,//一次性上传的文件数量上限
        maxFilesize: 2, //文件大小，单位：MB
        acceptedFiles: ".jpg,.gif,.png,.jpeg", //上传的类型
        addRemoveLinks:true,
        thumbnail:function(){},//为上传文件添加缩略图
        parallelUploads: 1,//一次上传的文件数量
        //previewsContainer:"#preview",//上传图片的预览窗口
        dictDefaultMessage:'拖动文件至此或者点击上传',
        dictMaxFilesExceeded: "您最多只能上传1个文件！",
        dictResponseError: '文件上传失败!',
        dictInvalidFileType: "文件类型只能是*.jpg,*.gif,*.png,*.jpeg。",
        dictFallbackMessage:"浏览器不受支持",
        dictFileTooBig:"文件过大上传文件最大支持.",
        dictRemoveLinks: "删除",
        dictCancelUpload: "取消",
        init:function(){
            this.on("addedfile", function(file) {
                //上传文件时触发的事件
            });
            this.on("success",function(file,data){
                //上传成功触发的事件
                console.log('ok');
            });
            this.on("error",function (file,data) {
                //上传失败触发的事件
                console.log('fail');
                
            });
            this.on("removedfile",function(file){
                //失败时触发的回调函数
            });
        }
    });
</pre>
</blockquote>
</p>
<h3>为组件添加上传预览字符串</h3>
<p>
	<pre>
		<div class="dz-preview dz-file-preview">
		  <div class="dz-details">
		    <div class="dz-filename"><span data-dz-name></span></div>
		    <div class="dz-size" data-dz-size></div>
		    <img data-dz-thumbnail />
		  </div>
		  <div class="dz-progress"><span class="dz-upload" data-dz-uploadprogress></span></div>
		  <div class="dz-success-mark"><span>✔</span></div>
		  <div class="dz-error-mark"><span>✘</span></div>
		  <div class="dz-error-message"><span data-dz-errormessage></span></div>
		</div>
	</pre>
</p>
<h3>为上传文件生成缩略图</h3>
<p>
	在配置项的thumbnail属性，添加以下方法即可:
	<blockquote>
		<pre>
		thumbnail: function(file, dataUrl) {
		 	if (file.previewElement) {
		 $(file.previewElement).removeClass("dz-file-preview");
		var images = $(file.previewElement).find("[data-dz-thumbnail]").each(function() {
					var thumbnailElement = this;
					thumbnailElement.alt = file.name;
					thumbnailElement.src = dataUrl;
		});
		setTimeout(function() { $(file.previewElement).addClass("dz-image-preview"); }, 1);
		    }
		}
			</pre>
	</blockquote>
</p>
<h3>为上传文件过程添加进度条</h3>
<p>
	<blockquote>
		<pre>
		 var minSteps = 6,
			      maxSteps = 60,
			      timeBetweenSteps = 100,
			      bytesPerStep = 100000;
			  myDropzone.uploadFiles = function(files) {
			    var self = this;
			    for (var i = 0; i < files.length; i++) {
			      var file = files[i];
			          totalSteps = Math.round(Math.min(maxSteps, Math.max(minSteps, file.size / bytesPerStep)));
			      for (var step = 0; step < totalSteps; step++) {
			        var duration = timeBetweenSteps * (step + 1);
			        setTimeout(function(file, totalSteps, step) {
			          return function() {
			            file.upload = {
			              progress: 100 * (step + 1) / totalSteps,
			              total: file.size,
			              bytesSent: (step + 1) * file.size / totalSteps
			            };
			            self.emit('uploadprogress', file, file.upload.progress, file.upload.bytesSent);
			            if (file.upload.progress == 100) {
			              file.status = Dropzone.SUCCESS;
			              self.emit("success", file, 'success', null);
			              self.emit("complete", file);
			              self.processQueue();
			            }
			          };
			        }(file, totalSteps, step), duration);
			      }
			    }
			   }
			  </pre>
	</blockquote>
	myDropzone为初始化生成的对象
</p>
<p>以上为踩过的坑和使用关键点，欢迎大家使用</p>