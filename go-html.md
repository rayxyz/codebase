# extract html text from html
```
abstract = bluemonday.StrictPolicy().Sanitize(post.Body)
runes := []rune(post.Abstract)
if len(runes) > 100 {
  post.Abstract = string(runes[:100]) + "..."
}
```

# change html content in fly
```
// ResizeImagesByURLInHTML :
func ResizeImagesByURLInHTML(content string, width int) (string, error) {
	if width <= 0 {
		width = 1000
	}
	doc, err := html.Parse(strings.NewReader(content))
	if err != nil {
		log.Error(err)
		return "", err
	}
	var f func(*html.Node)
	f = func(n *html.Node) {
		if n.Type == html.ElementNode && n.Data == "img" {
			for i, v := range n.Attr {
				if v.Key == "src" {
					if strings.Contains(v.Val, "https://xyz.com/file") || strings.Contains(v.Val, "http://localhost") {
						v.Val = v.Val + "?r=true&w=" + strconv.Itoa(width)
						n.Attr[i].Val = v.Val // !important if you wanna to change the value
						log.Println("resizing image => ", v.Val)
					}
					break
				}
			}
		}
		for c := n.FirstChild; c != nil; c = c.NextSibling {
			f(c)
		}
	}
	f(doc)
	buf := bytes.NewBuffer([]byte{})
	if err = html.Render(buf, doc); err != nil {
		return "", err
	}
	content = buf.String()

	return content, nil
}

```
