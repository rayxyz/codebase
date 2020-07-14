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
async uploadFile() {
	if (!this.file) {
		return;
	}
	this.fileUploading = true;
	let formData = new FormData();
	// formData.append('files', this.file);

	// console.log('this.cropper.croppedImage => ', this.cropper.croppedImage);
	let blob = await fetch(this.cropper.croppedImage).then(r => r.blob());
	// console.log('cropped image blob => ', blob);
	let tmpFile = new File([blob], this.file.name);
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
}
```

# a comprehensive example
```
<!DOCTYPE html>
<html>

<head>
	<meta charset="utf-8">
	<title>博客写作</title>
	<meta name="viewport" content="width=device-width, initial-scale=1">
	<meta name="keywords" content="写博客,写文章,Ahezime博客写作,post articles,write articles">
	<meta name="description" content="Ahezime博客写作 | Write Ahezime blog article">
	<link rel="shortcut icon" href="/static/images/favicon.ico" type="image/x-icon">
	<link rel="icon" href="/static/images/favicon.ico" type="image/x-icon">

	<script src="https://cdn.jsdelivr.net/npm/bulma-tagsinput@2.0.0/dist/js/bulma-tagsinput.min.js"></script>

	<script src="https://cdn.jsdelivr.net/npm/showdown@1.9.1/dist/showdown.min.js"></script>

	<link rel="stylesheet" href="//cdn.jsdelivr.net/gh/highlightjs/cdn-release@9.16.2/build/styles/default.min.css">
	<script src="//cdn.jsdelivr.net/gh/highlightjs/cdn-release@9.16.2/build/highlight.min.js"></script>
	<script>hljs.initHighlightingOnLoad();</script>

</head>

<body>
{{ template "header.html" . }}

{{ $postId := 0 }}

{{ if .post }}
{{ $postId = .post.Id }}
{{ end }}
<div id="editor">
	<nav class="level breadcrumb">
		<!-- Left side -->
		<div class="level-left">
			<div class="level-item">
				<ul>
					<li><a href="/blog">博客</a></li>
					{{ if not .post }}
					<li class="is-active"><a aria-current="page">写文章</a></li>
					{{ else }}
					<li><a href="/blog/post/{{ .post.Id }}">文章</a></li>
					<li class="is-active"><a aria-current="page">编辑</a></li>
					{{ end }}
				</ul>
			</div>
		</div>

		<!-- Right side -->
		<div class="level-right">
			<p class="level-item">
				<label class="checkbox" v-cloak>
					<input type="checkbox" :checked="showSettings" @click="showSettings = !showSettings">
					显示相关设置
				</label>
			</p>
			{{ if not .post }}
			<p class="level-item">
				<a class="button is-info" @click="save" :disabled="saveButtonDisabled">
					<span class="icon">
						<i class="fas fa-save"></i>
					</span>
					<span>保存</span>
				</a>
			</p>
			{{ else }}
			{{ if eq .post.State 1 }}
			<p class="level-item">
				<a class="button is-info" @click="save" :disabled="saveButtonDisabled">
					<span class="icon">
						<i class="fas fa-save"></i>
					</span>
					<span>保存</span>
				</a>
			</p>
			{{ end }}
			{{ end }}
			{{ if not .post }}
			<p class="level-item">
				<a class="button is-info" disabled="true">
					<span class="icon">
						<i class="fas fa-paper-plane"></i>
					</span>
					<span>发布</span>
				</a>
			</p>
			{{ else }}
			{{ if or (eq .post.State 0) (eq .post.State 1) }}
			<p class="level-item">
				<a class="button is-info" @click="publish" :disabled="publishButtonDisabled">
					<span class="icon">
						<i class="fas fa-paper-plane"></i>
					</span>
					<span>发布</span>
				</a>
			</p>
			{{ end }}
			{{ end }}
			{{ if .post }}
			{{ if eq .post.State 2 }}
			<p class="level-item">
				<a class="button is-info" @click="updatePost" :disabled="updateButtonDisabled">
					<span class="icon">
						<i class="fas fa-arrow-up"></i>
					</span>
					<span>更新</span>
				</a>
			</p>
			{{ end }}
			{{ end }}
			{{ if .post }}
			<p class="level-item">
				<!-- <a class="button is-danger" @click="showDeleteModal = true">
					<span class="icon">
						<i class="fas fa-trash"></i>
					</span>
					<span>删除</span>
				</a> -->
				<a class="button is-danger" @click="deletePost">
					<span class="icon">
						<i class="fas fa-trash"></i>
					</span>
					<span>删除</span>
				</a>
			</p>
			{{ end }}
		</div>
	</nav>

	<article v-if="showSettings" class="message is-info">
		<div class="message-header">
			<p>相关设置</p>
			<!-- <button class="delete" aria-label="delete"></button> -->
		</div>

		<div class="message-body" v-cloak>
			<div class="tile is-ancestor">
				<div class="tile is-parent is-vertical">
					<div class="tile is-child">
						<div class="field">
							<p class="control">
								<input class="input" placeholder="文章标题" v-model="post.title">
							</p>
						</div>
						<div class="field">
							<p class="control">
								<textarea class="textarea" placeholder="请输入文章摘要" v-model="post.abstract"></textarea>
							</p>
						</div>

						<hr class="dropdown-divider">

						<div class="field">
							<label>标签</label>
							<p class="control">
							<div class="dropdown" v-bind:class="{'is-active': showTagsDropdown}">
								<div class="dropdown-trigger">
									<div class="field has-addons">
										<p class="control is-expanded has-icons-right">
											<input class="input" type="search" maxlength="20" placeholder="查询标签..."
												v-on:keyup="searchTags" v-model="tagSearchKeyword" />
											<!-- <span class="icon is-small is-right"><i
														class="fas fa-search"></i></span> -->
										</p>
										<button class="button is-primary" :disabled="createTagButtonDisabled"
											@click="newTag">创建</button>
									</div>
								</div>
								<div v-if="tagList" class="dropdown-menu" id="dropdown-menu" role="menu">
									<div class="dropdown-content">
										<a v-for="tag in tagList" class="dropdown-item" @click="addTag(tag)">
											${ tag.name }
										</a>
										<!-- <hr class="dropdown-divider"> -->
									</div>
								</div>

							</div>
							</p>
						</div>
						<div v-if="showTagsArea" class="field is-grouped is-grouped-multiline has-background-white"
							style="padding-top: 10px;padding-left: 10px;">
							<div v-for="(tag, index) in post.tag_list" :ref="'tag_' + tag.id + '_' + index"
								class="control">
								<div class="tags has-addons">
									<a class="tag is-link">${ tag.name }</a>
									<a class="tag is-delete" @click="removeTag(tag, index)"></a>
								</div>
							</div>
						</div>
					</div>
				</div>
				<!-- v-cloak: show when the vue instance initiated -->
				<div class="tile is-parent is-vertical">
					<div class="tile is-child" v-cloak>
						<div class="field level">
							<div class="file is-info has-name is-fullwidth level-left">
								<label class="file-label">
									<input ref="postCoverImageFile" class="file-input" type="file" name="files"
										accept=".jpg, .jpeg, .png" @change="handleFileUpload">
									<span class="file-cta">
										<span class="file-icon">
											<i class="fas fa-upload"></i>
										</span>
										<span class="file-label">
											封面图片
										</span>
									</span>
									<span v-if="file" class="file-name">
										${ file.name }
									</span>
									<span v-else class="file-name">
										设置封面图片或更换
									</span>
								</label>
							</div>
							<div class="buttons level-right">
								<button v-if="!fileUploading || !file" :disabled="!file" class="button is-link"
									@click="uploadFile">上传</button>
								<button v-if="fileUploading" class="button is-link-light"
									@click="cancelFileUpload">取消</button>
							</div>
						</div>
					</div>
					<div class="tile is-child columns" v-cloak>
						<!-- do not show the uploading status by default-->
						<div v-if="fileURL" class="column field" style="padding: 0px;">
							<firgure class="image is-128x128">
								<img :src="fileURL" @load="loadCoverImage" @error="loadCoverImageError"
									style="max-width: 128px;max-height: 128px;" />
								<a v-if="coverImageLoaded" class="delete is-medium"
									style="position: absolute; top: -10px; right: -10px;"
									@click="removeCoverImage"></a>
							</firgure>
						</div>
						<div v-if="cropper.croppedImage && !fileURL" class="column field" style="padding: 0px;">
							<firgure class="image is-128x128">
								<img :src="cropper.croppedImage" style="max-width: 128px;max-height: 128px;" />
								<a class="delete is-medium" style="position: absolute; top: -10px; right: -10px;"
									@click="removeCoverImage"></a>
							</firgure>
						</div>
						<div class="column">
							<!-- align items center -->
							<div v-if="fileURL && !coverImageLoaded" style="display: flex; align-items: center;">
								<i class=" fa fa-circle-notch fa-spin fa-3x fa-fw"></i>
								<span class="content is-small">正在加载封面图片...</span>
							</div>
							<div v-if="fileUploading" class="field column">
								<p>
									<strong>${ fileUploadedPercent }% completed</strong>
									<progress class="progress is-primary" :value="fileUploadedPercent"
										max="100"></progress>
								</p>
							</div>
						</div>
					</div>
					<div class="tile is-child columns" v-cloak>

					</div>
				</div>
			</div>
		</div>
	</article>

	<div class="ahz-editor columns">
		<div class="column ahz-blog-editor">
			<!-- <textarea ref="editorTextArea" :value="input" @input="update" @paste="onPaste"></textarea> -->
			<textarea ref="editorTextArea" placeholder="请输入markdown格式内容..." v-model="post.body" @input="update"
				value=""></textarea>
		</div>
		<div class="column content ahz-blog-preview" v-html="compiledMarkdown">

		</div>
	</div>

	<div class="modal" v-bind:class="{'is-active': showCropperModal}">
		<div class="modal-background"></div>
		<div class="modal-content">
			<div class="content" style="width: 800px; height: 600px;">
				<vue-cropper ref="cropper" :img="cropper.option.img" :output-size="cropper.option.size"
					:output-type="cropper.option.outputType" :info="true" :full="cropper.option.full"
					:fixed="cropper.fixed" :fixed-number="cropper.fixedNumber" :can-move="cropper.option.canMove"
					:can-move-box="cropper.option.canMoveBox" :fixed-box="cropper.option.fixedBox"
					:original="cropper.option.original" :auto-crop="cropper.option.autoCrop"
					:auto-crop-width="cropper.option.autoCropWidth"
					:auto-crop-height="cropper.option.autoCropHeight" :center-box="cropper.option.centerBox"
					:high="cropper.option.high" mode="cover" :maxImgSize="cropper.option.max">
				</vue-cropper>
			</div>
			<div class="is-centered" style="margin-top: 30px;">
				<button class="button is-primary" @click="crop('blob')" style="margin-right: 20px;">完成</button>
				<button class="button is-grey" @click="showCropperModal = !showCropperModal">取消</button>
			</div>
		</div>

		<button class="modal-close is-large" aria-label="close"
			@click="showCropperModal = !showCropperModal"></button>
	</div>
</div>
<!-- {% include "../footer.html" %} -->
</body>
<script src="//cdn.jsdelivr.net/npm/vue-cropper@0.5.4/dist/index.min.js"></script>

<script>
Vue.use(window['vue-cropper']);
let vm = new Vue({
	delimiters: ['${', '}'],
	el: '#editor',
	components: {
	},
	data: {
		converter: null,
		post: {
			id: parseInt(`{{ $postId }}`),
			title: '',
			abstract: '',
			body: '',
			pic: '',
			state: 0,
			type: 2, // blog post
			tag_list: []
		},
		file: '',
		fileURL: '',
		fileUploading: false,
		fileUploadedPercent: 0,
		coverImageLoaded: false,
		showSettings: true,
		saveButtonDisabled: false,
		publishButtonDisabled: false,
		updateButtonDisabled: false,
		showDeleteModal: false,
		deleteButtonDisabled: false,
		showTagsArea: false,
		showTagsDropdown: false,
		tagSearchKeyword: '',
		tagList: null,
		createTagButtonDisabled: true,
		showCropperModal: false,
		cropper: {
			previews: {},
			croppedImage: '',
			option: {
				img: '',
				size: 1,
				full: false,
				outputType: 'png',
				canMove: true,
				fixedBox: false,
				original: false,
				canMoveBox: true,
				autoCrop: true,
				// 只有自动截图开启 宽度高度才生效
				autoCropWidth: 400,
				autoCropHeight: 300,
				centerBox: false,
				high: true,
				max: 99999
			},
			show: true,
			fixed: true,
			fixedNumber: [16, 9]
		}
	},
	mounted() {
		this.init();
		this.loadPost();
		bulmaTagsinput.attach();
	},
	computed: {
		compiledMarkdown: function () {
			if (this.converter == null) {
				this.init();
			}
			html = this.converter.makeHtml(this.post.body);
			return html;
		}
	},
	methods: {
		init() {
			this.$nextTick(() => {
				this.$refs.editorTextArea.focus();
			});
			this.converter = new showdown.Converter();
		},
		update(e) {
			this.highlightCode();
		},
		highlightCode() {
			document.querySelectorAll('.ahz-blog-preview pre').forEach((block) => {
				hljs.highlightBlock(block);
			});
		},
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
		},
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
		async uploadFile() {
			if (!this.file) {
				return;
			}
			this.fileUploading = true;
			let formData = new FormData();
			// formData.append('files', this.file);

			// console.log('this.cropper.croppedImage => ', this.cropper.croppedImage);
			let blob = await fetch(this.cropper.croppedImage).then(r => r.blob());
			// console.log('cropped image blob => ', blob);
			let tmpFile = new File([blob], this.file.name);
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
		},
		cancelFileUpload() {
			this.file = '';
			this.fileUploading = false;
		},
		loadCoverImage(evt) {
			this.coverImageLoaded = true;
		},
		loadCoverImageError(err) {
			showDialog('错误', '加载图片出错: ' + err, () => {
				console.log('confirm button clicked.');
			});
		},
		removeCoverImage(evt) {
			this.file = '';
			this.fileURL = '';
			this.cropper.option.img = '';
			this.cropper.croppedImage = '';
		},
		loadPost() {
			if (this.post.id > 0) {
				let url = appconfigs.APIGATEWAY_ADDR + `/post/v1/details/` + this.post.id;
				axios.get(url, null).then(resp => {
					if (resp.data.status == 200) {
						this.post = resp.data.data.post;
						if (this.post.pic != '') {
							this.fileURL = this.post.pic;
						}
						if (this.post.tag_list && this.post.tag_list.length > 0) {
							this.showTagsArea = true;
						}
					}
					this.saveButtonDisabled = false;
				}).catch(err => {
					this.saveButtonDisabled = false;
					console.error(err);
				});
			}
		},
		validatePost() {
			if (this.post.title.trim().length == 0) {
				bulmaToast.toast({ message: '请输入标题', type: "is-warning" });
				return false;
			}

			if (this.post.body.trim().length == 0) {
				bulmaToast.toast({ message: '文章Markdown内容不能为空', type: 'is-warning', position: 'center' });
				return false;
			}

			if (this.fileURL != '') {
				this.post.pic = this.fileURL;
			}

			if (!this.post.tag_list || this.post.tag_list.length == 0) {
				bulmaToast.toast({ message: '文章至少要有一个标签', type: 'is-danger', position: 'center' });
				return false;
			}

			if (this.post.tag_list.length > 5) {
				bulmaToast.toast({ message: '文章不能有多于5个标签', type: 'is-danger', position: 'center' });
				return false;
			}

			return true;
		},
		save() {
			if (this.saveButtonDisabled) {
				return;
			}
			if (!this.validatePost()) {
				return;
			}

			this.saveButtonDisabled = true;

			// console.log('post to be saved => ', this.post);

			let url = appconfigs.APIGATEWAY_ADDR + `/post/v1/new_or_save`
			axios.post(url, this.post).then(resp => {
				if (resp.data.status == 200) {
					if (this.post.id == 0) {
						this.post.id = resp.data.data;
						window.location.href = '/blog/post/' + resp.data.data + '/edit';
					}
					bulmaToast.toast({ message: '保存成功', type: 'is-success', position: 'center' });
				} else {
					bulmaToast.toast({ message: '保存失败', type: 'is-danger', position: 'center' });
				}
				this.saveButtonDisabled = false;
			}).catch(err => {
				this.saveButtonDisabled = false;
				console.error(err);
			});
		},
		publish() {
			if (this.publishButtonDisabled) {
				return;
			}
			if (!this.validatePost()) {
				return;
			}

			this.publishButtonDisabled = true;

			let url = appconfigs.APIGATEWAY_ADDR + `/post/v1/publish`
			axios.post(url, this.post).then(resp => {
				if (resp.data.status == 200) {
					bulmaToast.toast({ message: '发布成功', type: 'is-success', position: 'center' });
					window.location.href = '/blog';
				} else {
					console.error(resp.data.msg);
					bulmaToast.toast({ message: '发布失败', type: 'is-danger', position: 'center' });
				}
				this.publishButtonDisabled = false;
			}).catch(err => {
				this.publishButtonDisabled = false;
				console.error(err);
			});
		},
		updatePost() {
			if (this.updateButtonDisabled) {
				return;
			}
			if (!this.validatePost()) {
				return;
			}

			this.updateButtonDisabled = true;

			let url = appconfigs.APIGATEWAY_ADDR + `/post/v1/update`
			axios.post(url, this.post).then(resp => {
				if (resp.data.status == 200) {
					bulmaToast.toast({ message: '更新成功', type: 'is-success', position: 'center' });
					window.location.reload();
				} else {
					bulmaToast.toast({ message: '更新文章出错: ' + resp.data.msg, type: 'is-danger', position: 'center' });
				}
				this.updateButtonDisabled = false;
			}).catch(err => {
				this.updateButtonDisabled = false;
				console.error(err);
			});
		},
		deletePost() {
			// `this` is not visible outside of the vue instance, should define a vue instance
			// and use it by the instance name. Thus you can access vue instance directly.
			showDialog('#editor', '删除提示', '确定要删除文章？ ', (node) => {
				if (vm.deleteButtonDisabled) {
					return;
				}

				vm.deleteButtonDisabled = true;

				let url = appconfigs.APIGATEWAY_ADDR + `/post/v1/delete`
				axios.delete(url, { data: [vm.post.id] }).then(resp => {
					if (resp.data.status == 200) {
						bulmaToast.toast({ message: '删除成功', type: 'is-success', position: 'center' });
						window.location.href = '/blog';
					} else {
						console.error(resp.data.msg);
						bulmaToast.toast({ message: '删除文章出错: ' + resp.data.msg, type: 'is-danger', position: 'center' });
					}
					vm.deleteButtonDisabled = false;
				}).catch(err => {
					vm.deleteButtonDisabled = false;
					console.error(err);
				});
			}, null, '删除');
		},
		searchTags() {
			if (!this.tagSearchKeyword) {
				this.createTagButtonDisabled = true;
				this.tagList = null;
				return;
			}
			let params = {
				keyword: this.tagSearchKeyword,
			}
			let url = appconfigs.APIGATEWAY_ADDR + `/post/v1/tag/list`;
			axios.get(url, { params: params }).then(resp => {
				if (resp.data.status == 200) {
					this.tagList = resp.data.data;
					if (this.tagList && this.tagList.length > 0) {
						this.showTagsDropdown = true;
					}
					if (!this.tagList && this.tagSearchKeyword && (this.tagSearchKeyword.trim().length > 0)) {
						this.createTagButtonDisabled = false;
					}
				}
			}).catch(err => {
				console.error(err);
			});
		},
		addTag(tag) {
			if (tag) {
				this.showTagsArea = true;
				if (!this.post.tag_list) {
					this.post.tag_list = [];
				}
				this.tagList = null;
				this.tagSearchKeyword = '';
				for (let i = 0; i < this.post.tag_list.length; i++) {
					if (this.post.tag_list[i].id == tag.id) {
						return;
					}
				}
				this.post.tag_list.push(tag)
			}
		},
		removeTag(tag, index) {
			let newIndex = this.post.tag_list.indexOf(tag)
			if (newIndex > -1) {
				this.post.tag_list.splice(newIndex, 1);
			}
			if (this.post.tag_list.length == 0) {
				this.post.tag_list = null;
				this.showTagsArea = false;
			}
		},
		newTag() {
			if (!this.tagSearchKeyword) {
				return;
			}
			this.createTagButtonDisabled = true;
			let url = appconfigs.APIGATEWAY_ADDR + `/post/v1/tag/new`;
			let tag = {
				name: this.tagSearchKeyword,
			};
			axios.post(url, tag).then(resp => {
				if (resp.data.status == 200) {
					this.showTagsArea = true;
					if (!this.post.tag_list) {
						this.post.tag_list = [];
					}
					this.post.tag_list.push(resp.data.data);
					this.tagList = null;
					this.tagSearchKeyword = '';
				} else {
					this.createTagButtonDisabled = false;
					bulmaToast.toast({ message: '创建标签出错', type: 'is-danger', position: 'center' });
					console.log(resp.data.msg)
				}
			}).catch(err => {
				this.createTagButtonDisabled = false;
				bulmaToast.toast({ message: '创建标签出错', type: 'is-danger', position: 'center' });
				console.error(err);
			})
		}
	}
});
</script>

<style>
#editor {
	margin: 10px;
	margin-left: 10px;
	margin-right: 10px;
	margin-bottom: 30px;
}

.ahz-editor {
	height: 82vh;
	display: flex;

	border-top: solid 1px;
	border-bottom: solid 1px;
	border-color: #f3f3f3;
}

.ahz-blog-editor {
	padding: 0px;

}

.ahz-blog-editor textarea {
	width: 100%;
	height: 100%;
	box-sizing: border-box;
	/* For IE and modern versions of Chrome */
	-moz-box-sizing: border-box;
	/* For Firefox                          */
	-webkit-box-sizing: border-box;
	/* For Safari                           */

	border: none;

	resize: none;
	padding: 10px;
	font-size: 16px;
	color: #333333;
	background-color: #fafafa;
}

.ahz-blog-editor textarea:focus,
input:focus {
	outline: 0;
}

.ahz-blog-preview {
	overflow-y: auto;
	overflow-x: auto;
}
</style>

</html>
```
