最近在用uniapp开发一款APP,因为需要兼容旧的H5页面,所以在uniapp中用到了webview,但是发现webview与uniapp的通信还是有一些局限性:  
1.开发文档中说过了,webview只有在plusready后,才能对plus.storage进行操作,或者调用uniapp的一些代码,但是webview加载plus的时间.实在是太慢了...快的时候,四五秒,慢的时候...二三十秒都有在测试中出现过  
2.虽然webview中能调用一些uniapp的代码.但也仍然不是全部,所以交互功能应该也还是受限的  
然后最近几天一直在看文档发现5+App 里网页可直接调用 5+Runtime 的相关原生能力,然后我又去看了5+Runtime的webview的api:[https://www.html5plus.org/doc/zh_cn/webview.html](https://www.html5plus.org/doc/zh_cn/webview.html),终于找到了一个比较好的方法:  
[https://www.html5plus.org/doc/zh_cn/webview.html#plus.webview.WebviewObject.overrideUrlLoading](https://www.html5plus.org/doc/zh_cn/webview.html#plus.webview.WebviewObject.overrideUrlLoading)  
## overrideUrlLoading:   
拦截Webview窗口的URL请求  
void wobj.overrideUrlLoading(options, callback);  
说明：  
拦截URL请求后，Webview窗口将不会跳转到新的URL地址，此时将通过callback回调方法返回拦截的URL地址（可新开Webview窗口加载URL页面等）。 此方法只能拦截窗口的网络超链接跳转（包括调用loadURL方法触发的跳转），不可拦截页面请求资源请求（如加载css/js/png等资源的请求）。 注意：多次调用overrideUrlLoading时仅以最后一次调用设置的参数值生效。  
这个地方有两个参数:  
### options:  
拦截Webview窗口URL请求的属性  
属性：  
#### effect:   
(String 类型 )拦截URL请求生效时机  
可取值： "instant" - 表示立即生效，即调用overrideUrlLoading方法后立即生效； "touchstart" - 表示用户操作Webview窗口（触发touchstart事件）后生效，如果用户没有操作Webview窗口则不对URL请求操作进行拦截处理。 默认值为"instant"。  
#### mode: 
(String 类型 )拦截模式  
可取值： "allow"表示满足match属性定义的条件时不拦截url继续加载，不满足match属性定义的条件时拦截url跳转并触发callback回调； "reject"表示满足match属性定义的提交时拦截url跳转并触发callback回调，不满足match属性定义的条件时不拦截url继续加载。 默认值为"reject"。  
#### match: 
(String 类型 )区配是否需要处理的URL请求  
支持正则表达式，默认值为对所有URL地址生效（相当于正则表达式“.*”）。  
如果mode值为"allow"则允许区配的URL请求跳转，mode值为"reject"则拦截区配的URL请求。  
#### exclude: 
(String 类型 )排除拦截处理请求类型  
不拦截处理指定类型的URL请求，直接使用系统默认处理逻辑。 可取值： "none"表示不排除任何URL请求（即拦截处理所有URL请求）； "redirect"表示排除拦截处理301/302跳转的请求（谨慎使用，非a标签的href触发的URL请求可能会误判断为302跳转）。 默认值为"none"。  
### OverrideUrlLoadingCallback  
Webview窗口拦截URL链接跳转的回调函数  
参数：  
#### event:   
( Event ) 必选 Webview窗口拦截URL跳转事件数据  
可通过event的url属性获取拦截的URL地址。
通过这个到底能做什么呢?我们先来看一张下面的效果图:  
![image](https://raw.githubusercontent.com/xieyushi/blog/master/blogimg/bloggif8.gif)  
图中顶部文字在uniapp中,中间的页面为webview,页面中并没有加载plus的东西.只是一个简单的链接,而我们点击链接的时候,页面并没有跳转,但是数据已经被uniapp拿到了.  
整个页面代码如下:  

```
<template>
	<view class="content">
		<view class="param">{{param}}</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				webview: null,
				param:''
			}
		},
		onLoad() {
			let curwebview = this.$mp.page.$getAppWebview();
			this.webview = plus.webview.open('https://blog.coder666.cn/webviewtest/index.html', '',{top:'246px',bottom:'100px'});
			curwebview.append(this.webview);
			this.webview.show();
			this.webview.overrideUrlLoading({mode:'reject'}, (e)=>{
				//根据参数的不同,做不同的操作!url中,把双引号用别的字符串替换.就OK了.再替换回来可以转json
				console.log(e);
				this.param ='参数为:'+ e.url;
				uni.showToast({
					icon:'none',
					title:'参数为:'+e.url
				})
			});
		},
		methods: {

		}
	}
</script>

<style>
	.content {
		text-align: center;
		height: 400upx;
	}

</style>
```
代码关键部分就只有onLoad中了.这里的mode我设置为了reject,即页面的所有的url都不跳转,然后拿到url,大家可以看到url上带了参数,所以webview可以通过这种方式来进行数据传输,不过唯一的美中不足就是url有最大长度限制,至于多长...请大家自行百度...我百度到的,是字符串不能超过1855,不过也够你放很多东西了吧...一般说来是够的.  
通讯已经搞定了.接下来还要再说一点的是,overrideUrlLoading这个方法,在reject模式下,或者是allow模式下,都可以有match配置,即,符合设置的字符串的才进行拦截或者通过.
match的配置,示例:  
'match:'.*(/member/index|/back\.html)'  
上面的match, 表示网址以/member/index结尾或者是/back.html结尾就会匹配.
然后去判断e.url,比如是/member/index就去用户中心,如果是back.html,就返回到上一个uniapp的页面,等等操作.webview无需等待plus加载.
代码已经放在了github上.  
github地址:[https://github.com/xieyushi/uniappwebviewconnect](https://github.com/xieyushi/uniappwebviewconnect)
