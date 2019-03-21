# Delete items in for loop
```golang
for i := len(submittedList) - 1; i >= 0; i-- {
  v := submittedList[i]
  for _, vx := range approvedList {
    if vx.Id == v.Id {
      submittedList = append(submittedList[:i], submittedList[i+1:]...)
      break
    }
  }
}
```
