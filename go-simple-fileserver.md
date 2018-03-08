```
// 上传文件列表
func uploadFileListHandler(ctx context.Context, req *httpx.Request, resp *httpx.Response) error {
	var fileList []*pb.File
	request := req.GetRequest()
	err := request.ParseMultipartForm(dict.MaxUploadMemory)
	if err != nil {
		return err
	}
	mpf := request.MultipartForm
	files := mpf.File["files"]
	for _, v := range files {
		log.Println(" file_name => ", v.Filename, " file_size => ", v.Size)
		if dict.MaxFileSize < v.Size {
			resp.StatusCode = exception.CodeDataTooLarge
			resp.Message = "文件 '" + v.Filename + "' 太大（" + util.FileSizeInString(v.Size) + "）, 已超过" +
				util.FileSizeInString(dict.MaxFileSize)
			return nil
		}
		file, errx := v.Open()
		if errx != nil {
			resp.StatusCode = exception.CodeRunningError
			resp.Message = "解析文件出错！"
			return nil
		}
		defer file.Close()

		fileRelativePath, fileAbsPath, errx := genFilePaths(v.Filename)
		if errx != nil {
			return errx
		}
		newFile, errx := os.Create(fileAbsPath)
		if errx != nil {
			resp.StatusCode = http.StatusInternalServerError
			resp.Message = "创建文件出错"
			return nil
		}
		defer newFile.Close()

		dataLen, errx := io.Copy(newFile, file)
		if errx != nil {
			resp.StatusCode = http.StatusInternalServerError
			resp.Message = "上传文件出错"
			return nil
		}

		fileModel := &pb.File{
			Name:    v.Filename,
			Src:     strings.Join([]string{"http://", config.FileServerAddr, ":", strconv.Itoa(int(config.FileServerPort)), prefix, "/file"}, ""),
			Size:    int32(dataLen),
			Path:    fileRelativePath,
			IsLocal: config.DeployLocal,
		}

		fileList = append(fileList, fileModel)
	}

	fileList, err = api.SaveFileList(ctx, fileList)
	if err != nil {
		resp.StatusCode = exception.CodeSaveDataError
		resp.Message = "保存文件信息出错"
		return nil
	}

	for _, v := range fileList {
		v.Src = strings.Join([]string{v.Src, big.NewInt(v.Id).String()}, "/")
	}

	success, err := api.UpdateFileSrc(ctx, fileList)
	if err != nil || !success {
		resp.StatusCode = exception.CodeUpdateDataError
		resp.Message = "更新文件信息出错"
		return nil
	}

	resp.WriteKeyvals("data", fileList)

	return nil
}
```

```
var (
	// homedir = os.Getenv("HOME")
	homedir = "/root"
)

// 生成文件在服务器上的路劲
func genFilePaths(fileName string) (fileRelativePath string, fileAbsPath string, err error) {
	fileNameHash := util.GenerateMD5HashCode(strings.Join([]string{fileName, time.Now().Format("2006-01-02 15:04:05")}, ""))
	log.Println("file name hash: ", fileNameHash)

	fileRelativePath += time.Now().Format("2006/01/02") + "/" + fileNameHash[0:2]
	fileRelativePath = path.Join(dict.FileSotreDir, fileRelativePath)
	log.Println("file relative path before concat: ", fileRelativePath)
	pathToCreate := path.Join(homedir, fileRelativePath)
	if err := os.MkdirAll(pathToCreate, 0777); err != nil {
		log.Println("Making direcitories error.")
		return dict.Blank, dict.Blank, err
	}
	log.Println("making directories complete.")

	fileRelativePath = path.Join(fileRelativePath, string(fileNameHash[2:len(fileNameHash)]))
	fileNameParts := strings.Split(fileName, ".")
	fileRelativePath = strings.Join([]string{fileRelativePath, "." + fileNameParts[len(fileNameParts)-1]}, "")
	fileAbsPath = strings.Join([]string{homedir, fileRelativePath}, "/")

	log.Println("fileRelativePath => ", fileRelativePath)
	log.Println("fileAbsPath => ", fileAbsPath)

	return fileRelativePath, fileAbsPath, nil
}
```
