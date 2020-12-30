<script>
	import Vue from 'vue';
	import Links from 'static/utils/links.js';
	import Request from 'static/utils/request.js';
	export default {
		onLaunch: function() {
			let self = this;
			Vue.prototype.Url = Links.links;
			Vue.prototype.Get = Request.get;
			Vue.prototype.Post = Request.post;
			Vue.prototype.Config = Request.config;
			uni.getSystemInfo({
				success: function(e) {
					Vue.prototype.windowWidth = e.windowWidth;
					Vue.prototype.WindowHeight = e.windowHeight;
					Vue.prototype.IphoneX = e.model.search("iPhone X");
					// #ifdef MP-WEIXIN
					let custom = wx.getMenuButtonBoundingClientRect();
					Vue.prototype.Custom = custom;
					Vue.prototype.StatusBarHeight = e.statusBarHeight+custom.height;
					Vue.prototype.StatusBar = e.statusBarHeight;
					Vue.prototype.SafeHeight = e.safeArea.height;
					Vue.prototype.CustomBar = custom.bottom + custom.top - e.statusBarHeight;
					// #endif
				}
			});
			// #ifdef MP-WEIXIN
				self.$options.wechatLogin();
				self.$options.getShopInfo();
			// #endif
		},
		wechatLogin() {
			let self = this;
			let token = uni.getStorageSync('__yipinUserInfo').token;
			if(token){
				Vue.prototype.Get(Vue.prototype.Url.checkToken, {}).then(res => {
					if (res.code === 0) {
						self.globalData.user = true;
					}else{
						uni.removeStorageSync('__yipinUserInfo');
						self.wechatLogin();
					}
				});
			}else{
				uni.login({
				    success: function(wes) {
						provider: 'weixin',
						Vue.prototype.Get(Vue.prototype.Url.wechatLogin, {code: wes.code}).then(res => {
							if(res.code === 1e4){
								self.globalData.user = false;
								uni.removeStorageSync('__yipinUserInfo');
								return
							}
							if(res.code === 0){
								self.globalData.user = true;
								uni.setStorage({key: '__yipinUserInfo',data: res.data});
							}
						});
				    }
				});
			}
		},
		getShopInfo() {
			let self = this;
			Vue.prototype.Get(Vue.prototype.Url.shopInfo, {}).then(res => {
				if (res.code === 0) {
					uni.setStorage({key: '__yipinShopInfo',data: res.data});
				}else{
					uni.removeStorageSync('__yipinShopInfo');
				}
			});
		},
		globalData: {
			user: false
		}
	}
</script>

<style lang="less">
	page {
		font-size: 28rpx;
		height: 100%;
		.content {
			height: 100%;
		}
	}

	image {
		width: 100%;
		height: 100%;
		display: block;
	}

	::-webkit-scrollbar {
		width: 0;
		height: 0;
		color: transparent;
	}
	
	.lineBottom:after{
		position: absolute;
		content: '';
		width: 100%;
		left: 0;
		bottom: 0;
		height: 1px;
		background-color: #f5f5f5;
		transform: scale(1, 0.5);
		transform-origin: center bottom;
	}
	
	.loadingBox{
		position: fixed;
		width: 100%;
		height: 100%;
		top: 0;
		left: 0;
		z-index: 99;
		display: flex;
		align-items: center;
		justify-content: center;
		background-color: #fff;
		.loading {
		    width: 80rpx;
		    height: 80rpx;
		 }
	}
</style>
