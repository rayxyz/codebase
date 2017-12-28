```
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
```

```
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
