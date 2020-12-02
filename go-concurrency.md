# A comprehensive example 
```go
package concurrency

import (
	"errors"
	"fmt"
	"strings"
	"testing"
	"time"

	log "github.com/sirupsen/logrus"
)

type mysqlDataObject struct {
	ID         int64
	Name       string
	CreateTime string
	UpdateTime string
}

func insertData(obj *mysqlDataObject) error {
	fmt.Println("inserting data...")
	time.Sleep(3 * time.Second)
	obj.CreateTime = time.Now().Format("2006-01-02 15:04:05")
	fmt.Println("name: ", obj.Name, ", insert time => ", obj.CreateTime)
	fmt.Println("new record inserted")

	return nil
}

func updateData(obj *mysqlDataObject) error {
	fmt.Println("updating data...")
	time.Sleep(2 * time.Second)
	obj.UpdateTime = time.Now().Format("2006-01-02 15:04:05")
	fmt.Println("name: ", obj.Name, ", update time => ", obj.UpdateTime)
	if !strings.EqualFold(obj.UpdateTime, "") {
		return errors.New("error")
	}
	fmt.Println("record updated")

	return nil
}

func genMySQLDataObject() *mysqlDataObject {
	return &mysqlDataObject{
		Name: "Ray_" + fmt.Sprintf("%d", time.Now().UnixNano()),
	}
}

func doDirtyWork(obj *mysqlDataObject) {
	fmt.Println(">>>>>>>>>>>>>>>> name in doDirtyWork => ", obj.Name)
	// synch := make(chan int, 1)
	errch := make(chan error, 1)
	go func() {
		err := insertData(obj)
		if err != nil {
			errch <- err
			return
		}
		err = updateData(obj)
		if err != nil {
			errch <- err
			return
		}
		errch <- nil
		fmt.Println("Okay")
		// synch <- 1
	}()
	// select {
	// case err := <-errch:
	// 	if err != nil {
	// 		log.Error(err)
	// 	}
	// case <-synch:
	// 	fmt.Println("Okey.")
	// }

	err := <-errch
	if err != nil {
		log.Error(err)
	}

	fmt.Println("<<<<<<<<<<<<<<<<<<<<<<< name in doDirtyWork => ", obj.Name)
}

func TestMockMySQLConcur(t *testing.T) {
	for i := 0; i < 5; i++ {
		obj := genMySQLDataObject()
		fmt.Println("index => ", i, ", name: ", obj.Name)
		doDirtyWork(obj)
		fmt.Println()
	}
}

```
