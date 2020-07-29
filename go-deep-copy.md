# copy
Check more in file go-chan-xxxx.md file, there's the QRCode generation example.
But you cannot copy a slice of structs by using the built-in copy function. The assignment of ResponseTime will 
change the ResponseTime of structs of source svcList.
```
svcList := registor.GetAll()
liteSvcList := make([]*registry.Service, len(svcList))
copy(liteSvcList, svcList)
for _, v := range liteSvcList {
  v.ResponseTime = nil
}
resp.WriteJSON(liteSvcList)
```
