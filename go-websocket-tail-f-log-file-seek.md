# use websocket & TailFile to tail log 
```
import (
	"github.com/hpcloud/tail"
	log "github.com/sirupsen/logrus"
	"github.com/gorilla/websocket"
)

upgrader.CheckOrigin = func(r *http.Request) bool {
		return true
	}
	c, err := upgrader.Upgrade(resp.GetWriter(), req.GetRequest(), nil)
	if err != nil {
		log.Print("upgrade:", err)
		return nil
	}
	defer c.Close()

	t, err := tail.TailFile(logFileName, tail.Config{Follow: true, Location: &tail.SeekInfo{
		Offset: 200,
		Whence: os.SEEK_END,
	}})
	if err != nil {
		log.Error(err)
	}
	for line := range t.Lines {
		err = c.WriteMessage(websocket.TextMessage, []byte(line.Text))
		if err != nil {
			c.Close()
			t.Stop()
			log.Println("write:", err)
		}
	}

```
