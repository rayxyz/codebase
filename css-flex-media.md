# flex row with wrap
```css
display: flex; flex-direction: row; flex-wrap: wrap;
```
```html
<div v-if="post.pic_list && post.pic_list.length > 0"
  style="display: flex; flex-direction: row; flex-wrap: wrap;" v-cloak>
  <div v-for="(pic, index) in post.pic_list" :key="pic.id" class="content"
    style="padding: 0px; margin-right:20px;">
    <firgure v-if="pic.pic && pic.pic != ''" class="image is-128x128">
      <img :src="pic.pic" @load="loadCoverImage($event, pic)"
        @error="loadCoverImageError($event, pic)"
        style="max-width: 128px;max-height: 128px;" />
      <a :key="pic.id" class="delete is-medium"
        style="position: absolute; top: -10px; right: -10px;"
        @click="removeCoverImage($event, pic)"></a>
    </firgure>
  </div>
</div>
```

# media

```
@media all and (max-width: 760px) {
    body {
	      background-image: none;
	      background-color: lightgray;
    }
}
```
