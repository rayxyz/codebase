# copy
Check more in file go-chan-xxxx.md file, there's the QRCode generation example.
```
svcList := registor.GetAll()
liteSvcList := make([]*registry.Service, len(svcList))
copy(liteSvcList, svcList)
for _, v := range liteSvcList {
  v.ResponseTime = nil
}
resp.WriteJSON(liteSvcList)
```
