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
But you cannot copy a slice of structs by using the built-in copy function. The assignment of ResponseTime will 
change the ResponseTime of structs of source svcList. Use the copier library to solve the problem

The revised version, it works:
```
var liteSvcList []*registry.Service
svcList := registor.GetAll()
// liteSvcList := make([]*registry.Service, len(svcList))
// copy(liteSvcList, svcList)
copier.Copy(&liteSvcList, &svcList)
for _, v := range liteSvcList {
  v.ResponseTime = nil
}
resp.WriteJSON(liteSvcList)
``
