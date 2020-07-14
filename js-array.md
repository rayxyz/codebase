# remove an item from an array
```
removeCoverImage(evt, pic) {
    const index = this.post.pic_list.indexOf(pic);
    if (index > -1) {
        this.post.pic_list.splice(index, 1);
    }
},
```
# find items in array
```
let filteredObjs = this.post.pic_list.filter(obj => {
    return obj.id === pic.id;
});
console.log('found => ', filteredObjs);
```
