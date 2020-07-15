# upload files with axios cancel token
```
FileUploader.prototype.upload2Server = function (files, ele, customEvt, callback) {
	for (let i = 0; i < files.length; i++) {
		files[i].percent = 0;
		const formData = new FormData();
		formData.append(this.inputFileName, files[i]);

		let source = axios.CancelToken.source();
		this.cancelTokenSourceList.push(source);
		axios.post(this.uploadURL, formData, {
			cancelToken: source.token,
			headers: {
				'Content-Type': 'multipart/form-data'
			},
			responseType: 'json',
			onUploadProgress: function (progressEvent) {
				let percent = Math.round((progressEvent.loaded * 100) / progressEvent.total);
				if (ele && customEvt) {
					files[i].percent = percent;
					customEvt.files = files;
					ele.dispatchEvent(customEvt);
				}
			}
		}).then(resp => {
			if (resp.data.status == 200) {
				// console.log(resp.data);
				callback(resp.data.data, null, files[i]);
			} else {
				callback(null, resp.data.msg, files[i]);
				console.error(resp.data);
			}
		}).catch(e => {
			if (axios.isCancel(e)) {
				console.log('Request canceled => ', e.message);
			} else {
				// handle error
				callback(null, e, files[i]);
				console.error(e);
			}
		});
	}
}
```
cancel the request/uploading:
```
let sourceList = this.cancelTokenSourceList;
				if (sourceList && sourceList.length > 0) {
					for (let i = 0; i < sourceList.length; i++) {
						sourceList[i].cancel('cancel cover image uploading');
					}
				}
```
