# http

> simple

Get: 
```go 

url := "xyz.com/abc"
r, err := http.Get(url)
if err != nil {
	resp.Status = httpx.StatusQueryDataError
	resp.Message = "query service from registry server error"
	return nil
}
defer r.Body.Close()

if r.StatusCode != http.StatusOK {
	io.Copy(ioutil.Discard, r.Body) // !important
	resp.Status = httpx.StatusQueryDataError
	resp.Message = "registry server response status => " + r.Status
	return nil
}
```

Post:
```go
resp, err := http.Post(registerAddr, "application/json", bytes.NewBuffer(svcData))
if err != nil {
	log.Fatalf("connect to registry server '%s' error\n", registerAddr)
	return err
}
defer resp.Body.Close()

if resp.StatusCode == http.StatusOK {
	buf, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(err)
		return err
	}
	if err = json.Unmarshal(buf, &s); err != nil {
		log.Fatal(err)
		return err
	}
	log.Infof("register service %s success, key => %s\n", s.Name, s.Key)
} else {
	bbs, _ := ioutil.ReadAll(resp.Body)
	errmsg := fmt.Sprintf("status => %s, message => %s", resp.Status, string(bbs))
	log.Fatal(errmsg)
	return errors.New(errmsg)
}
```

> complex
```go
type PalmLog struct {
	SchoolId int64  `json:"school_id,string"`
	CampusId int64  `json:"campus_id,string"`
	UserId   int64  `json:"user_id,string"`
	RoleId   int32  `json:"role_id, string"`
	Data     string `json:"data"`
}

func sendLogToServer(palmLog *PalmLog) {
	palmLogBytes, err := json.Marshal(palmLog)
	if err != nil {
		return
	}
	req, err := http.NewRequest("POST", "http://www.ray-xyz.com:8080/savePalmLog", bytes.NewBuffer(palmLogBytes))
	req.Header.Set("X-Custom-Header", "palm-running-log")
	req.Header.Set("Content-Type", "application/json")

	client := &http.Client{}
	resp, err := client.Do(req)
	if err != nil && resp != nil {
		defer resp.Body.Close()
	}
	if err != nil {
		log.Debug("Connection to log server error.")
		return
	}
	if resp.StatusCode == 200 {
		log.Info("Send log to logserver ok!")
	}
}

func GetEasemobHXToken() (*EasemobHXToken, error) {
	reqBodyBytes, err := json.Marshal(GetEasemobHXTokenReq{
		GrantType:    grantType,
		ClientId:     hxinfo.ClientId,
		ClientSecret: hxinfo.ClientSecret,
	})
	if err != nil {
		return nil, err
	}

	req, err := http.NewRequest("POST", hxinfo.BaseUrl+"/token", bytes.NewBuffer([]byte(reqBodyBytes)))
	if err != nil {
		return nil, err
	}
	req.Header.Set("Accept", "application/json")
	req.Header.Set("Content-Type", "application/json")

	client := &http.Client{}
	resp, err := client.Do(req)
	if err != nil && resp != nil {
		defer resp.Body.Close()
	}
	if err != nil {
		return nil, err
	}
	if resp.StatusCode == http.StatusOK {
		body, err := ioutil.ReadAll(resp.Body)
		hxtoken := new(EasemobHXToken)
		if err = json.Unmarshal(body, hxtoken); err != nil {
			return nil, err
		}
		// log.Printf("I got the hx token => %#v", hxtoken)
		return hxtoken, nil
	}

	return nil, errors.New("attaining hx token error")
}
```

> tcp/client
```go
package main

import (
	"bufio"
	"encoding/json"
	"fmt"
	"log"
	"mqtt/control"
	"net"
)

// Client : MQTT Client
type Client struct {
	Conn net.Conn
}

func main() {
	fmt.Println("connecting to the MQTT server")
	client := &Client{}
	client.Connect()
}

// Connect to the server
func (c *Client) Connect() {
	header := new(control.ConnectHeader)
	payload := &control.ConnectPayload{
		ClientID:  "client_id看",
		WillTopic: "willtopic",
		WillMsg:   "will message fdsfsdfsdfsdfsdfsdfsdfsdfsdfsee",
		UserName:  "ray",
		Password:  "ray123",
	}
	payloadBytes, err := json.Marshal(payload)
	if err != nil {
		panic("marsh payload error")
	}
	headerBytes, err := header.Marshal(len(payloadBytes))
	if err != nil {
		panic("marshal header error")
	}
	var pack []byte
	pack = append(pack, headerBytes...)
	pack = append(pack, payloadBytes...)

	log.Println(pack)

	conn, err := net.Dial("tcp", "localhost:8080")
	if err != nil {
		panic("error of listening")
	}
	c.Conn = conn

	fmt.Fprintf(conn, string(pack))
	data, err := bufio.NewReader(conn).ReadString('\n')
	if err != nil {
		log.Println("error of reading content")
	}
	fmt.Println(data)
}
```

> tcp/server
```go
package main

import (
	"fmt"
	"log"
	"mqtt/control"
	"mqtt/store"
	"net"
	"strings"
)

func main() {
	fmt.Println("I am the primitive MQTT server, and I am alive...")
	fmt.Println("Server running on port => ", 8080)
	ln, err := net.Listen("tcp", ":8080")
	if err != nil {
		panic("error of creating mqtt server")
	}
	for {
		conn, err := ln.Accept()
		if err != nil {
			log.Println("error of accept connection")
		}
		go handleConn(conn)
	}
}

func init() {
	fmt.Println("I am fucking init....")
	store.ClientIDMap = make(map[string]string, 10)
}

func handleConn(conn net.Conn) {
	// defer conn.Close()
	b, err := control.ReadPacket(conn)
	if err != nil {
		log.Println(err)
		return
	}
	log.Println(" data lenth => ", len(b), "data received => ", b)
	if b[0]>>4 == control.CONNECT {
		fmt.Println("Connection packet detected.")
		packet, err := control.ParseConnectPacket(b)
		if !strings.EqualFold(store.ClientIDMap[packet.Payload.ClientID], "") {
			log.Println("Repeated connection request, disconnecting....")
			conn.Close()
			delete(store.ClientIDMap, packet.Payload.ClientID)
			log.Println("Disconnected client => ", packet.Payload.ClientID)
			return
		}
		store.ClientIDMap[packet.Payload.ClientID] = packet.Payload.ClientID
		if err != nil {
			log.Println(err)
			return
		}
		conn.Write([]byte("header => " + (packet.Header.String()) + "  |||||| payload => " + fmt.Sprintf("%#v", packet.Payload) + "\n"))
	}
}

// ReadPacket : Read MQTT packet from stream
func ReadPacket(conn net.Conn) ([]byte, error) {
	buf := make([]byte, 0, 4096)
	tmp := make([]byte, 128)
	i := 0
	packLen := 0
	for {
		n, err := conn.Read(tmp)
		if err != nil {
			if err != io.EOF {
				log.Println("read data error")
			}
			break
		}
		if i == 0 {
			packLenParsed, err := GetPackLen(tmp[1:5])
			if err != nil {
				log.Println(err)
				return nil, err
			}
			packLen = packLenParsed
			i++
		}
		buf = append(buf, tmp[:n]...)
		if len(buf) >= packLen {
			break
		}
	}
	return buf, nil
}

```
