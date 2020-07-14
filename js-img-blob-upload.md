# References
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
