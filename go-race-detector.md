# Race detector usage
 ```
 ray@ray-pc:~/go_workspace/src/ahezime.com/ahezime/test$ go test -race
```

```
func randomDuration() time.Duration {
	return time.Duration(rand.Int63n(1e9))
}

func TestRaceDetector(t *testing.T) {
	start := time.Now()
	var tmr *time.Timer
	tmr = time.AfterFunc(randomDuration(), func() {
		fmt.Println(time.Now().Sub(start))
		tmr.Reset(randomDuration())
	})
	// _ = tmr.C
	time.Sleep(5 * time.Second)
}

// func TestNonRacing(t *testing.T) {
// 	start := time.Now()
// 	reset := make(chan bool)
// 	var tmr *time.Timer
// 	tmr = time.AfterFunc(randomDuration(), func() {
// 		fmt.Println(time.Now().Sub(start))
// 		reset <- true
// 	})
// 	for time.Since(start) < 10*time.Second {
// 		<-reset
// 		tmr.Reset(randomDuration())
// 	}
// }
```
