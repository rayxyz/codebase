> int64 to string 

```
stringDigits := strconv.FormatInt(78, 10)
```

> Interface{}/map[string]interface{} to specific type
```
// BuriedDataCollectParams :
type BuriedDataCollectParams struct {
	DataType string      `json:"data_type"`
	Data     interface{} `json:"data"`
}
```

```
bdcp := new(model.BuriedDataCollectParams)
if err := req.ParseBody(bdcp); err != nil {
  return err
}
```

```
dataBytes, err := json.Marshal(bdcp.Data)
if err != nil {
  log.Error(err)
}
fmt.Println("dataBytes => ", string(dataBytes))

var minerInfo *model.MinerInfo
if err := json.Unmarshal(dataBytes, &minerInfo); err != nil {
  log.Error(err)
  return err
}
```
