> The goroutines block on sending to the unbuffered channel. A minimal change unblocks the goroutines is to create a buffered channel with capacity 

> go channel selection

```go
type ReadOp struct {
	key int32
}

type WriteOp struct {
	key int32
}

func testChan() {
	reads := make(chan *ReadOp)
	writes := make(chan *WriteOp)
	go func() {
		for {
			select {
			case read := <-reads:
				fmt.Println("read, read.key => ", read.key)
			case write := <-writes:
				fmt.Println("write, write.key => ", write.key)
			}
		}
	}()
	for i := 0; i < 100; i++ {
		readOp := &ReadOp{
			key: int32(i),
		}
		reads <- readOp

		writeOp := &WriteOp{
			key: int32(i),
		}
		writes <- writeOp
	}
}
```

> label
```go
func testLabel() {
loop:
	for i := 0; i < 100; i++ {
		switch i {
		case 49:
			fmt.Println("i == 49")
			// break
			break loop
		}
		if i == 49 {
			fmt.Println("You didn't get out of the place.")
		}
	}
}
```

> chan & waitgroup
``` go
func cacheAllData() {
	log.Println("Begin caching all data...")
	wg := new(sync.WaitGroup)
	wg.Add(11)
	done := make(chan struct{})

	go func(done <-chan struct{}, wg *sync.WaitGroup) {
		defer wg.Done()
		userList, err := api.GetAllUsers(context.Background())
		if err != nil {
			return
		}
		log.Println(">>> Caching users...")
		CacheAllUsers(userList)
		<-done
	}(done, wg)

	go func(done <-chan struct{}, wg *sync.WaitGroup) {
		defer wg.Done()
		schoolList, err := sapi.GetAllSchools(context.Background())
		if err != nil {
			return
		}
		log.Println(">>> Caching schools...")
		CacheAllSchools(schoolList)
		<-done
	}(done, wg)

	go func(done <-chan struct{}, wg *sync.WaitGroup) {
		defer wg.Done()
		campusList, err := sapi.GetAllCampuses(context.Background())
		if err != nil {
			return
		}
		log.Println(">>> Caching cmpuses...")
		CacheAllCampuses(campusList)
		<-done
	}(done, wg)

	close(done)
	wg.Wait()

	log.Println("Cache all data complete.")
}

func checkForUpdates() {
	if !PingDB() {
		log.Println(">>> Checking for updates failed.")
		return
	}

	log.Printf(">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>\n")
	log.Println(">>> Checking for updates...")
	conn := NewConn()
	lastCheckTime, err := redis.String(conn.Do("GET", strings.Join([]string{constants.CachePrefix, "last_check_time"}, constants.SeparatorUnderscore)))
	if conn != nil {
		defer conn.Close()
	}
	if err != nil {
		log.Println(err.Error())
		// TODO: Reinit the cache.
		return
	}
	log.Println(">>> Last check time: ", lastCheckTime)
	synch := make(chan int)
	go func() {
		go updateUserInfo(checkUser(lastCheckTime))
		go updateSchoolInfo(checkSchool(lastCheckTime))
		go updateCampusInfo(checkCampus(lastCheckTime))
		synch <- 1
	}()
	<-synch
	log.Println(">>> Checking for updates complete.")
	log.Printf("<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<\n\n")
	dbtime := util.GetDBTime()
	log.Println(">>> Update `last_check_time` to: ", dbtime)
	conn.Do("SET", strings.Join([]string{constants.CachePrefix, "last_check_time"}, constants.SeparatorUnderscore), dbtime)
}
```

> Nonblocking chan
```go
func testNonBlockingChan() {
	timeout := time.After(5 * time.Second)
	ch := make(chan int)
	go assignCh(ch)
loop:
	for {
		log.Println("HAHAHAHAH")
		select {
		case <-timeout:
			{
				fmt.Println("Timeout.")
			}
		case <-ch:
			{
				fmt.Println("Channel case.")
				break loop
			}
		}
	}
	fmt.Println("Hello, I am out !")
}

func assignCh(ch chan int) chan int {
	time.Sleep(12 * time.Second)
	ch <- 1
	return ch
}
```

> Real world example
```go
type qrCode struct {
	Key    string `json:"key"`
	QRCode string `json:"qr_code"`
}

type genQRCodeRawData struct {
	Key     string  `json:"key"`
	IntKey  int     `json:"int_key"`
	Value   string  `json:"value"`
	Bgcolor []uint8 `json:"bgcolor"`
	Fgcolor []uint8 `json:"fgcolor"`
	Size    int     `json:"size,string"`
}

type genQRCodeListParams struct {
	RawData []*genQRCodeRawData `json:"raw_data"`
}

// Generate QRCode list
func genQRCodeListHandler(ctx context.Context, req *httpx.Request, resp *httpx.Response) error {
	params := new(genQRCodeListParams)
	if err := req.MapJson(params); err != nil {
		return err
	}
	if params.RawData == nil || len(params.RawData) == 0 {
		resp.StatusCode = exception.CodeParseParamsError
		resp.Message = "error params！"
		return nil
	}

	// var qrcodeList []*qrCode
	// for _, v := range params.RawData {
	// 	q, err := qrcode.New(v.Value, qrcode.Medium)
	// 	q.BackgroundColor = color.RGBA{v.Bgcolor[0], v.Bgcolor[1], v.Bgcolor[2], 255}
	// 	q.ForegroundColor = color.RGBA{v.Fgcolor[0], v.Fgcolor[1], v.Fgcolor[2], 255}
	// 	buf, err := q.PNG(v.Size)
	// 	if err != nil {
	// 		resp.StatusCode = exception.CodeRunningError
	// 		resp.Message = "generating QR code error！"
	// 		return nil
	// 	}
	// 	base64QRCode := base64.StdEncoding.EncodeToString([]byte(buf))
	// 	base64QRCodeImg := "data:image/png;base64," + base64QRCode
	// 	qrcodeList = append(qrcodeList, &qrCode{
	// 		Key:    v.Key,
	// 		QRCode: base64QRCodeImg,
	// 	})
	// }

	batchCount := 0
	batchMap := make(map[int][]*genQRCodeRawData)
	var rawDataList []*genQRCodeRawData
	for i, v := range params.RawData {
		if strings.EqualFold(v.Key, dict.Blank) || strings.EqualFold(v.Value, dict.Blank) ||
			len(v.Bgcolor) < 3 || len(v.Fgcolor) < 3 {
			resp.StatusCode = exception.CodeParseParamsError
			resp.Message = "data item empty：key => " + v.Key + ", value => " + v.Value
			return nil
		}
		v.Key = strconv.Itoa(i)
		v.IntKey = i
		log.Println(i)
		v.Value = v.Key + v.Value
		rawDataList = append(rawDataList, v)
		if len(rawDataList) >= 50 {
			batchCount++
			// need to depp copy rawDataList
			cpy := make([]*genQRCodeRawData, len(rawDataList))
			copy(cpy, rawDataList)
			batchMap[batchCount] = cpy
			rawDataList = append(rawDataList[:0])
		}
	}
	batchCount++
	cpy := make([]*genQRCodeRawData, len(rawDataList))
	copy(cpy, rawDataList)
	batchMap[batchCount] = cpy
	log.Println("length(batchMap) => ", len(batchMap))
	rawDataList = append(rawDataList[:0])

	synch := make(chan struct{})
	wg := new(sync.WaitGroup)
	var qrcodeList []*qrCode
	for i, dataList := range batchMap {
		rawDataList = append(rawDataList, batchMap[i]...)
		wg.Add(1)
		go func(synch <-chan struct{}, wg *sync.WaitGroup, dataList []*genQRCodeRawData) {
			defer wg.Done()
			var list []*qrCode
			for _, v := range dataList {
				q, err := qrcode.New(v.Value, qrcode.Medium)
				q.BackgroundColor = color.RGBA{v.Bgcolor[0], v.Bgcolor[1], v.Bgcolor[2], 255}
				q.ForegroundColor = color.RGBA{v.Fgcolor[0], v.Fgcolor[1], v.Fgcolor[2], 255}
				buf, err := q.PNG(v.Size)
				if err != nil {
					log.Println("error of generating QR code！")
				}
				base64QRCode := base64.StdEncoding.EncodeToString([]byte(buf))
				base64QRCodeImg := "data:image/png;base64," + base64QRCode
				list = append(list, &qrCode{
					Key:    v.Key,
					QRCode: base64QRCodeImg,
				})
			}
			qrcodeList = append(qrcodeList, list...)
			<-synch
		}(synch, wg, dataList)
		log.Println("executed goroutine ", i, " finished.")
	}

	close(synch)
	wg.Wait()

	resp.WriteKeyvals("data", qrcodeList)

	return nil
}
```



