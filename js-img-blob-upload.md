# vue-cropper demo code
```
crop(type) {
	if (type === 'blob') {
		this.$refs.cropper.getCropBlob((data) => {
			console.log(data);
			var img = window.URL.createObjectURL(data);
			this.cropper.croppedImage = img;
			// this.cropper.croppedImage = data;
		})
	} else {
		this.$refs.cropper.getCropData((data) => {
			this.cropper.croppedImage = data;
		});
	}
	this.showCropperModal = !this.showCropperModal;
}

handleFileUpload(e) {
	this.file = this.$refs.postCoverImageFile.files[0];
	console.log(this.file);
	this.showCropperModal = true;

	var reader = new FileReader();
	reader.onload = (e) => {
		console.log(e.target.result);
		console.log('typeof e.target.result => ', typeof e.target.result);
		let data;
		if (typeof e.target.result === 'object') {
			// 把Array Buffer转化为blob 如果是base64不需要
			data = window.URL.createObjectURL(new Blob([e.target.result]));
		} else {
			data = e.target.result;
		}
		this.cropper.option.img = data;
	}
	// 转化为base64
	// reader.readAsDataURL(file)
	// 转化为blob
	reader.readAsArrayBuffer(this.file)
},
```

# blob url/url to blob
> async
[https://stackoverflow.com/questions/52798723/append-created-image-file-in-formdata](https://stackoverflow.com/questions/52798723/append-created-image-file-in-formdata)
```
	fetch(this.cropper.croppedImage)
					.then(res => res.blob())
					.then(blob => {
            // so something.
					});
          
```

> sync 
[https://stackoverflow.com/questions/11876175/how-to-get-a-file-or-blob-from-an-object-url](https://stackoverflow.com/questions/11876175/how-to-get-a-file-or-blob-from-an-object-url)
```
let blob = await fetch(url).then(r => r.blob());
```

```
let formData = new FormData();
// formData.append('files', this.file);

let blob = await fetch(this.cropper.croppedImage).then(r => r.blob());
console.log('cropped image buffer => ', blob);
let tmpFile = new File(blog, this.file.name);
formData.append('files', tmpFile);

let that = this;
let config = {
  headers: {
    'Content-Type': 'multipart/form-data'
  },
  onUploadProgress: function (progressEvent) {
    that.fileUploadedPercent = Math.round((progressEvent.loaded * 100) / progressEvent.total);
    // console.log('this.fileUploadedPercent => ', that.fileUploadedPercent);
  }
};

let url = appconfigs.APIGATEWAY_ADDR + `/file/upload`;
axios.post(url, formData, config).then(resp => {
  // console.log(resp.data);
  if (resp.data.status == 200) {
    this.fileURL = resp.data.data[0].src;
    this.file = '';
    this.cropper.croppedImage = '';
  }
  this.fileUploading = false;
}).catch(err => {
  this.fileUploading = false;
  cosole.log('Upload cover image error => ' + err);
  bulmaToast.toast({ message: '上传封面图片出错', type: 'is-danger', position: 'center' });
});
```
