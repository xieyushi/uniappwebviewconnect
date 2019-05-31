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
