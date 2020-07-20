# custom template funcs
```
package template

import (
	"reflect"
	"text/template"

	"ahezime.com/ahezime/pkg/util"

	log "github.com/sirupsen/logrus"
)

// var (
// 	// Context :
// 	Context map[string]interface{}
// )

// Data :
type Data map[string]interface{}

func add(i, j int) int {
	return i + j
}

func sub(i, j int) int {
	return i - j
}

func mul(i, j int) int {
	return i * j
}

func divide(i, j int) int {
	return i / j
}

func mod(i, j int) int {
	return i % j
}

func slicy(number int) []int {
	var s []int
	for i := 0; i < number; i++ {
		s = append(s, i)
	}
	return s
}

func hasField(v interface{}, name string) bool {
	rv := reflect.ValueOf(v)
	if rv.Kind() == reflect.Ptr {
		rv = rv.Elem()
	}
	if rv.Kind() != reflect.Struct {
		return false
	}
	return rv.FieldByName(name).IsValid()
}

func truncate(s string, i int, dots bool) string {
	return util.Truncate(s, i, dots)
}

func extractHTMLText(html string) string {
	return util.ExtractHTMLText(html, 0)
}

func formatNumber(number interface{}) string {
	// number.(int64)
	var numberReal int64
	switch number.(type) {
	case int:
		numberReal = int64(number.(int))
	case int8:
		numberReal = int64(number.(int8))
	case int16:
		numberReal = int64(number.(int16))
	case int32:
		numberReal = int64(number.(int32))
	case int64:
		numberReal = number.(int64)
	case uint:
		numberReal = int64(number.(uint))
	case uint8:
		numberReal = int64(number.(uint8))
	case uint16:
		numberReal = int64(number.(uint16))
	case uint32:
		numberReal = int64(number.(uint32))
	case uint64:
		numberReal = int64(number.(uint64))
	case float32:
		numberReal = int64(number.(float32))
	case float64:
		numberReal = int64(number.(float64))
	default:
		log.Error("Unsupported type => " + reflect.TypeOf(number).Kind().String())
	}

	return util.FormatNumber2String(numberReal)
}

func funcMap() map[string]interface{} {
	return template.FuncMap{
		// The name "inc" is what the function will be called in the template text.
		"add":             add,
		"sub":             sub,
		"divide":          divide,
		"mul":             mul,
		"mod":             mod,
		"slicy":           slicy,
		"hasField":        hasField,
		"truncate":        truncate,
		"formatNumber":    formatNumber,
		"extractHTMLText": extractHTMLText,
	}
}

// New :
func New(filenames ...string) (*template.Template, error) {
	return template.New("tmpl").Funcs(funcMap()).ParseFiles(filenames...)
}
```

# backend handler
```
// Get the index page
func indexPageHandler(ctx context.Context, req *httpx.Request, resp *httpx.Response) error {
	resp.Stream = true
	
	...
	
	tmpl, err := template.New(
		filepath.Join(staticResourcePathWebapp, "index.html"),
		filepath.Join(staticResourcePathWebapp, "pages/header.html"),
		filepath.Join(staticResourcePathWebapp, "pages/footer.html"),
	)
	if err != nil {
		return err
	}
	err = tmpl.ExecuteTemplate(resp.GetWriter(), "index.html", template.Data{
		"page":              "home",
		"user":              user,
		"topBlogPostList":   topBlogPostComList,
		"blogPostList":      blogPostComList,
		"communityPostList": communityPostComList,
	})
	if err != nil {
		return err
	}

	return nil
}
```

# html comprehensive example
```
<!DOCTYPE html>
<html>

<head>
	<meta charset="utf-8">
	<title>Ahezime: 一个开放的博客和社区平台网站。| Ahezime: An open blog & community platform.</title>
	<meta name="viewport" content="width=device-width, initial-scale=1">
	<meta name="keywords" content="Ahezime.com,博客,社区,blog,open blog platform,博客网站,开放式博客平台,社区网站平台">
	<meta name="description" content="Ahezime.com是一个用户可以在上面写博客和发帖分享、传播知识的博客和社区平台网站。">
	<link rel="shortcut icon" href="/static/images/favicon.ico" type="image/x-icon">
	<link rel="icon" href="/static/images/favicon.ico" type="image/x-icon">

	<link rel="stylesheet" href="/static/css/index-main.css">
</head>

<body>
	{{ template "header.html" .}}
	<div id="app">
		{{ if not .user}}
		<section v-if="showTopMsg" class="hero is-primary" v-cloak>
			<a class="delete is-medium" style="position: relative; top: 8px; right: calc(-100% + 30px) ;"
				@click="showTopMsg = !showTopMsg"></a>
			<div class="hero-body">
				<div class="container">
					<h2 class="subtitle">
						没有Ahezime账号？
					</h2>
					<h1 class="title">
						<span style="cursor: pointer;" @click="window.location.href='/register'">立即注册 <i
								class="fa fa-arrow-right"></i></span>
					</h1>
				</div>
			</div>
		</section>
		{{ end }}

		<div class="container index-container">
			<div class="tile is-ancestor">
				<div class="tile is-parent">
					<div class="tile is-6 is-child notification">
						<img src="/static/images/ahezime-dashboard.png">
					</div>
					<div class="tile is-child notification">
						<div class="content">
							<p style="font-size: 17px; color: #333333;">
								Ahezime.com是一个用户可以在上面写博客和发帖分享、传播知识的博客和社区平台网站。
							</p>
						</div>
						<div>
							<a class="button is-info is-large" href="/about">
								<span>了解更多</span>
								<span class="icon is-small">
									<i class="fas fa-book"></i>
								</span>
							</a>
						</div>
					</div>
				</div>
			</div>
		</div>

		{{ if .topBlogPostList }}
		{{ if gt (len .topBlogPostList) 0 }}
		<div class="container index-container">
			<div class="tile is-ancestor is-vertical">
				<div class="tile">
					<p class="is-child subtitle" style="margin: 10px;">
						<span><i class="fa fa-blog has-text-grey"></i>&nbsp;&nbsp;<strong>博客Top10</strong></span>
					</p>
				</div>
				{{ $topBlogPostList := .topBlogPostList }}
				{{ $topBlogPostCount := len .topBlogPostList}}
				{{ $topBlogLoopCount := (divide $topBlogPostCount 3) }}
				{{ $topBlogLoopCountRemainder := (mod $topBlogPostCount 3) }}
				{{ if gt $topBlogLoopCountRemainder 0 }}
				{{ $topBlogLoopCount = (add $topBlogLoopCount 1) }}
				{{ end }}

				{{ range $loopIndex := slicy $topBlogLoopCount }}
				<div class="tile post-tile">
					<!-- slice func is officially supported -->
					{{ $start := mul $loopIndex 3 }}
					{{ $end := add (mul $loopIndex 3) 3 }}
					{{ if gt $end $topBlogPostCount }}
					{{ $end = $topBlogPostCount}}
					{{ end }}
					{{ range $i, $v := slice $topBlogPostList $start $end }}
					<div class="tile is-parent is-4" @click="loadPostDetails({{ .Post.Id }}, 2)" style="height: auto;">
						<div class="tile is-child has-background-white-ter rows">
							<div class="columns media">
								{{ if not (eq $v.Post.Pic "") }}
								<div class="column is-5" style="height: 100%;">
									<p class="image is-16by9">
										<img src="{{ .Post.Pic }}" />
									</p>
								</div>
								{{ end }}
								<div class="column is-7" style="margin-left: 10px;height: 100%;">
									<div class="content is-small">
										<span class="ahezime-title">{{ $v.Post.Title }}</span>
										<p class="ahezime-abstract">{{ truncate $v.Post.Abstract 100 true}}</p>
									</div>
								</div>
							</div>

							<div class="sticky-at-bottom-bar">
								<a class="has-text-grey" style="margin-right: 10px;">
									<span class="icon is-small">
										<i class="fas fa-eye"></i>
									</span>
									{{ if gt $v.PageviewCount 0 }}
									{{ formatNumber $v.PageviewCount }}
									{{ end }}
								</a>
								<a class="has-text-grey" style="margin-right: 10px;">
									<span class="icon is-small">
										<i class="fas fa-thumbs-up"></i>
									</span>
									{{ if gt $v.CountUpvote 0 }}
									{{ formatNumber $v.CountUpvote }}
									{{ end }}
								</a>
								<a class="has-text-grey" style="margin-right: 10px;">
									<span class="icon is-small">
										<i class="fas fa-comment"></i>
									</span>
									{{ if gt .CountComments 0 }}
									{{ formatNumber $v.CountComments }}
									{{ end }}
								</a>
								<span class="content"
									style="float: right;font-size: 13px; margin-right: 20px; color: #808080;">
									{{ $v.UserInfo.UserName }} · ${ getDateFromYMDHMSTime("{{ $v.Post.PublishTime }}")}
								</span>
							</div>
						</div>
					</div>
					{{ end }}
				</div>
				{{ end }}
			</div>
		</div>
		{{ end }}
		{{ end }}

		{{ if .blogPostList }}
		{{ if gt (len .blogPostList) 0 }}
		<div class="container index-container">
			<div class="tile is-ancestor is-vertical post-tile">
				<div class="tile">
					<p class="is-child subtitle" style="margin: 10px;">
						<span><i class="fa fa-blog has-text-grey"></i>&nbsp;&nbsp;<strong>最新博客文章</strong></span>
					</p>
				</div>

				{{ $blogPostList := .blogPostList }}
				{{ $blogPostCount := len .blogPostList}}
				{{ $blogPostLoopCount := (divide $blogPostCount 3) }}
				{{ $blogPostLoopCountRemainder := (mod $blogPostCount 3) }}
				{{ if gt $blogPostLoopCountRemainder 0 }}
				{{ $blogPostLoopCount = (add $blogPostLoopCount 1) }}
				{{ end }}

				{{ range $loopIndex := slicy $blogPostLoopCount }}
				<div class="tile">
					{{ $start := mul $loopIndex 3 }}
					{{ $end := add (mul $loopIndex 3) 3 }}
					{{ if gt $end $blogPostCount }}
					{{ $end = $blogPostCount}}
					{{ end }}
					{{ range $i, $v := slice $blogPostList $start $end }}
					<div class="tile is-parent is-4" @click="loadPostDetails({{ .Post.Id }}, 2)" style="height: auto;">
						<div class="tile is-child has-background-white-ter rows">
							<div class="columns media">
								{{ if not (eq $v.Post.Pic "") }}
								<div class="column is-5" style="height: 100%;">
									<p class="image is-16by9">
										<img src="{{ .Post.Pic }}" />
									</p>
								</div>
								{{ end }}
								<div class="column is-7" style="margin-left: 10px;height: 100%;">
									<div class="content is-small">
										<span class="ahezime-title">{{ $v.Post.Title }}</span>
										<p class="ahezime-abstract">{{ truncate $v.Post.Abstract 100 true}}</p>
									</div>
								</div>
							</div>

							<div class="sticky-at-bottom-bar">
								<a class="has-text-grey" style="margin-right: 10px;">
									<span class="icon is-small">
										<i class="fas fa-eye"></i>
									</span>
									{{ if gt $v.PageviewCount 0 }}
									{{ formatNumber $v.PageviewCount }}
									{{ end }}
								</a>
								<a class="has-text-grey" style="margin-right: 10px;">
									<span class="icon is-small">
										<i class="fas fa-thumbs-up"></i>
									</span>
									{{ if gt $v.CountUpvote 0 }}
									{{ formatNumber $v.CountUpvote }}
									{{ end }}
								</a>
								<a class="has-text-grey" style="margin-right: 10px;">
									<span class="icon is-small">
										<i class="fas fa-comment"></i>
									</span>
									{{ if gt .CountComments 0 }}
									{{ formatNumber $v.CountComments }}
									{{ end }}
								</a>
								<span class="content"
									style="float: right;font-size: 13px; margin-right: 20px; color: #808080;">
									{{ $v.UserInfo.UserName }} · ${ getDateFromYMDHMSTime("{{ $v.Post.PublishTime }}")}
								</span>
							</div>
						</div>
					</div>
					{{ end }}
				</div>
				{{ end }}

				{{ if eq (len $blogPostList) 8}}
				<div class="tile" style="margin: 10px;">
					<div class="is-parent">
						<a class="is-child is-small" href="/blog" style="color: #009AFF;">
							<span>
								<strong>更多</strong>
								<i class="fa fa-arrow-right"></i>
							</span>
						</a>
					</div>
				</div>
				{{ end }}
			</div>
		</div>
		{{ end }}
		{{ end }}

		<!-- <div class="is-divider"></div> -->

		{{ if .communityPostList }}
		{{ if gt (len .communityPostList) 0 }}
		<div class="container index-container">
			<div class="tile is-ancestor is-vertical post-tile">
				<div class="tile">
					<p class="is-child subtitle" style="margin: 10px;">
						<span><i class="fa fa-user-friends has-text-grey"></i>&nbsp;&nbsp;<strong>最热社区帖子</strong></span>
					</p>
				</div>
				{{ $communityPostList := .communityPostList }}
				{{ $communityPostCount := len .communityPostList}}
				{{ $communityPostLoopCount := (divide $communityPostCount 4) }}
				{{ $communityPostLoopCountRemainder := (mod $communityPostCount 4) }}
				{{ if gt $communityPostLoopCountRemainder 0 }}
				{{ $communityPostLoopCount = (add $communityPostLoopCount 1) }}
				{{ end }}

				{{ range $loopIndex := slicy $communityPostLoopCount }}
				<div class="tile post-tile">
					{{ $start := mul $loopIndex 4 }}
					{{ $end := add (mul $loopIndex 4) 4 }}
					{{ if gt $end $communityPostCount }}
					{{ $end = $communityPostCount}}
					{{ end }}
					{{ range $i, $v := slice $communityPostList $start $end }}
					<div class="tile is-parent is-3" @click="loadPostDetails({{ .Post.Id }}, 1)" style="height: auto;">
						<div class="tile is-child has-background-white-ter rows">
							<div class="">
								<!-- <span class="content"
									style="float: right;font-size: 13px; margin-right: 20px; color: #808080;">
									{{ $v.UserInfo.UserName }} · ${ getDateFromYMDHMSTime("{{ $v.Post.PublishTime }}")}
								</span> -->
								<div style="float: left; margin-right: 10px;">
									{{ if $v.UserInfo.Avatar }}
									{{ if not (eq $v.UserInfo.Avatar "") }}
									<img class="ahezime-avatar" src="{{ $v.UserInfo.Avatar }}" />
									{{ end }}
									{{ end }}

								</div>
								<div class="content is-small">
									<span><a target="_blank"
											href="/u/profile?uid={{ $v.UserInfo.ID }}">{{ $v.UserInfo.UserName }}</a></span><br />
									<span style="font-size: 11; color: lightgray;">${
										getDateFromYMDHMSTime("{{ $v.Post.PublishTime }}") }</span>
								</div>
							</div>

							<div class="under-image-section">
								<div class="content is-small">
									<!-- <span class="ahezime-title">{{ $v.Post.Title }}</span> -->
									<!-- <p class="ahezime-abstract">{{ truncate $v.Post.Body 150 true}}</p> -->
									<p class="ahezime-abstract">{{ $v.Post.Abstract}}</p>
								</div>
							</div>

							<!-- <div class="content">
								{{ if not (eq $v.Post.Pic "") }}
								<p class="image is-16by9">
									<img src="{{ $v.Post.Pic }}" width="100%" />
								</p>
								{{ else }}
								<div class="image is-16by9"
									style="background-color: lightgray; border-top-left-radius: 4px; border-top-right-radius: 4px;">
								</div>
								{{ end }}
							</div> -->
							{{ if $v.Post.PicList }}
							<div class="content">
								{{ range $picIndex, $pic := $v.Post.PicList }}
								<img src="{{ $pic.Pic }}" width="60" height="60"
									style="object-fit: cover; margin-right: 10px;" />
								{{ end }}
							</div>
							{{ end }}

							<div class="sticky-at-bottom-bar">
								<a class="has-text-grey" style="margin-right: 10px;">
									<span class="icon is-small">
										<i class="fas fa-eye"></i>
									</span>
									{{ if gt $v.PageviewCount 0 }}
									{{ formatNumber $v.PageviewCount }}
									{{ end }}
								</a>
								<a class="has-text-grey" style="margin-right: 10px;">
									<span class="icon is-small">
										<i class="fas fa-thumbs-up"></i>
									</span>
									{{ if gt $v.CountUpvote 0 }}
									{{ formatNumber $v.CountUpvote }}
									{{ end }}
								</a>
								<a class="has-text-grey" style="margin-right: 10px;">
									<span class="icon is-small">
										<i class="fas fa-comment"></i>
									</span>
									{{ if gt .CountComments 0 }}
									{{ formatNumber $v.CountComments }}
									{{ end }}
								</a>
							</div>
						</div>
					</div>
					{{ end }}
				</div>
				{{ end }}

				{{ if eq (len .communityPostList) 9}}
				<div class="tile" style="margin: 10px;">
					<div class="is-parent">
						<a class="is-child is-small" href="/community" style="color: #009AFF;">
							<span>
								<strong>更多</strong>
								<i class="fa fa-arrow-right"></i>
							</span>
						</a>
					</div>
				</div>
				{{ end }}
			</div>
		</div>
		{{ end }}
		{{ end }}

	</div>
	{{ template "footer.html" }}
</body>

<script>
	new Vue({
		delimiters: ['${', '}'],
		el: '#app',
		data: {
			showTopMsg: true,
		},
		created: function () {
			// console.log('Vue is created.');
		},
		mounted: function () {
			// console.log('Vue is mounted.');
			this.setPageView();
		},
		methods: {
			setPageView() {
				setPageView("homepage");
			},
			loadPostDetails(postId, type) {
				if (type == 1) {
					window.open(appconfigs.APIGATEWAY_ADDR + '/community/post/' + postId);
				}
				if (type == 2) {
					window.open(appconfigs.APIGATEWAY_ADDR + '/blog/post/' + postId);
				}
			}
		}
	});
</script>

<style>
	#app {
		margin-bottom: 30vh;
	}

	.index-container {
		padding: 12px;
		margin-top: 3vh;
	}

	.post-tile .is-child {
		border-radius: 5px;
		padding: 10px;
		color: #555555;
		cursor: pointer !important;
	}

	.post-tile .is-child .media {
		max-height: 14vh;
		overflow: hidden;
		word-break: break-word;
	}

	@media all and (max-width: 760px) {
		.post-tile .is-child .media {
			min-height: 16vh;
		}

		/* .post-tile .is-child .ahezime-abstract {
			display: none;
		} */
	}

	.post-tile .is-child .media img {
		border-radius: 4px;
	}

	.rows {
		display: flex;
		flex-direction: column;
	}

	.sticky-at-bottom-bar {
		height: 20px;
		margin-top: auto;
		display: flex;
	}

	.sticky-at-bottom-bar span {
		font-size: 14px;
	}
</style>

</html>
```
