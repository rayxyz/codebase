# copy
```
var liteSvcList []*registry.Service
svcList := registor.GetAll()
copy(liteSvcList, svcList)
for _, v := range liteSvcList {
  v.ResponseTime = nil
}
resp.WriteJSON(liteSvcList)
```
