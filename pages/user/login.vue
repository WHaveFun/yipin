<template>
	<view class="content">
	    <view class="login">
	        <image class="logo" :src="shopData.shop_logo" />
	        <view class="title" :style="'color:'+template.color.color3">{{shopData.shop_name}}</view>
	        <view class="text" :style="'color:'+template.color.color5">首次进入需要授权才能使用哦</view>
			<view class="loginBox">
				<button class="userlogin" :style="'background-color:'+template.color.color1" open-type="getUserInfo" @getuserinfo="userlogin">授权登录</button>
				<button class="userlogin navigateBack" @click="navigateBack()" :style="'color:'+template.color.color5">暂不登陆</button>
			</view>
	    </view>
	</view>
</template>

<script>
	import Icon from "@/components/icon/icon.vue";
	const app = getApp();
	export default {
		components: {
			Icon
		},
		data() {
			return {
				template: uni.getStorageSync('__yipinTemplateConfig'),
				shopData: uni.getStorageSync('__yipinShopInfo'),
				wxCode: ''
			}
		},
		onLoad() {
			let self = this;
			self.getWechatLogin();
		},
		methods: {
			getWechatLogin() {
				let self = this;
				uni.login({
					provider: 'weixin',
					success: function(wxs) {
						self.wxCode = wxs.code;
					}
				})
			},
			userlogin(e) {
				let self = this;
				let user = e.detail;
				if(user.iv){
					self.Post(self.Url.wechatDetail, {
						iv: user.iv,
						code: self.wxCode,
						raw_data: user.rawData,
						signature: user.signature,
						encrypted_data: user.encryptedData
					}).then(res => {
						if(res.code === 0){
							app.globalData.user = true;
							uni.showToast({title: '授权登陆成功'});
							uni.setStorage({key: '__yipinUserInfo',data: res.data});
							setTimeout(function () {
								uni.navigateBack();
							}, 500);
						}else{
							uni.showModal({
							  title: '授权失败',
							  content: '获取授权信息失败，请重新授权登录',
							  showCancel: false
							});
						}
					});
				}else{
					self.getWechatLogin();
					uni.showModal({
					  title: '授权失败',
					  content: '获取授权信息失败，请重新授权登录',
					  showCancel: false
					});
				}
			},
			navigateBack() {
				uni.navigateBack();
			}
		}
	}
</script>

<style lang="less">
	page{
		background-color: #FFFFFF;
	}
	.login{
	    height:60vh;
	    width:100%;
	    display:flex;
	    align-items:center;
	    justify-content:center;
	    flex-direction:column;
	    .logo{
	        width:120rpx;
	        height:120rpx;
	        border-radius:100%;
	    }
	    .title{
	        color:#27323f;
	        font-size:32rpx;
	        padding-top:20rpx;
	    }
	    .text{
	        color:#27323f;
	        padding-top:20rpx;
	        font-size:26rpx;
	    }
		.loginBox{
			padding-top: 50rpx;
			.input{
				width: 350rpx;
				height: 70rpx;
				border: 1px solid #f5f5f5;
				border-radius: 6rpx;
				text-align: center;
			}
			.send{
				color: #FFFFFF;
				margin-top: 20rpx;
				height: 70rpx;
				display: flex;
				align-items: center;
				justify-content: center;
				font-size: 26rpx;
				border-radius: 6rpx;
			}
			.userlogin{
			    background-color:#f59072;
			    width:250rpx;
			    height:70rpx;
			    font-size:28rpx;
			    display:flex;
			    align-items:center;
			    justify-content:center;
			    padding:0;
			    color:#fff;
			    border-radius:6rpx;
			    margin-bottom:30rpx;
			}
			.navigateBack{
			    background-color:#f8f8f8;
			    color:#979d9f;
			    margin-top:10rpx;
			}
			button::after{
			    border: 0;
			} 
		}
	}
</style>
