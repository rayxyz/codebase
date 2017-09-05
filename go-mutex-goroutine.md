> sync.Mutex

```go
type LockedObj struct {
	count int
	sync.Mutex
}

func testLock() {
	lockObj := new(LockedObj)
	lockObj.count = 10
	doTestLock(lockObj)
	go func() {
		fmt.Println("person_02: lockObj.count => ", lockObj.count)
		lockObj.count = lockObj.count + 2
		fmt.Println("person_02: plused lockObj.count => ", lockObj.count)
	}()
	time.Sleep(time.Second * 5)
}

func doTestLock(lockObj *LockedObj) {
	fmt.Println("Before lock.")
	lockObj.Lock()
	time.Sleep(time.Second * 3)
	lockObj.count = lockObj.count + 1
	lockObj.Unlock()
	fmt.Println("After unlocked.", " lockObj.count => ", lockObj.count)
}

func main() {
	testLock()
}
```
