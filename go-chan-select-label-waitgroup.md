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

```
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
``` 
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




