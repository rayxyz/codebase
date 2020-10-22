# Race detector usage
 ```shell
 ray@ray-pc:~/go_workspace/src/ahezime.com/ahezime/test$ go test -race
```

```go
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

Race condition detected: 
```shell
948.079392ms
==================
WARNING: DATA RACE
Read at 0x00c00011e028 by goroutine 17:
  ahezime.com/ahezime/test.TestRaceDetector.func1()
      /home/ray/go_workspace/src/ahezime.com/ahezime/test/common_test.go:169 +0x121

Previous write at 0x00c00011e028 by goroutine 15:
  ahezime.com/ahezime/test.TestRaceDetector()
      /home/ray/go_workspace/src/ahezime.com/ahezime/test/common_test.go:167 +0x18d
  testing.tRunner()
      /usr/local/go/src/testing/testing.go:1039 +0x1eb

Goroutine 17 (running) created at:
  time.goFunc()
      /usr/local/go/src/time/sleep.go:168 +0x51

Goroutine 15 (running) created at:
  testing.(*T).Run()
      /usr/local/go/src/testing/testing.go:1090 +0x700
  testing.runTests.func1()
      /usr/local/go/src/testing/testing.go:1334 +0xa6
  testing.tRunner()
      /usr/local/go/src/testing/testing.go:1039 +0x1eb
  testing.runTests()
      /usr/local/go/src/testing/testing.go:1332 +0x527
  testing.(*M).Run()
      /usr/local/go/src/testing/testing.go:1249 +0x43f
  main.main()
      _testmain.go:62 +0x223
==================
1.031103808s
1.69807842s
1.933737491s
2.221416044s
2.770879572s
3.40418194s
3.736197786s
3.919836732s
4.400745368s
--- FAIL: TestRaceDetector (5.00s)
    testing.go:954: race detected during execution of test
Getting name
FAIL
exit status 1
```
