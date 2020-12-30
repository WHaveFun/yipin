<template>
	<view class="content">
		<block v-if="brandInfo">
			<view class="headerBox" :style="'background-color:'+brandInfo.mini_color">
				<view class="logo">
					<image v-if="brandInfo.mini_logo" :src="brandInfo.mini_logo"></image>
					<view v-if="brandInfo.mini_title" class="title">{{brandInfo.mini_title}}</view>
				</view>
				<view class="text" v-if="brandInfo.mini_intro"><text></text><span>{{brandInfo.mini_intro}}</span></view>
			</view>
			<view class="infoBox">
				<view class="title">开通&定制专属小程序</view>
				<view class="text">请联系您的专属客服</view>
				<view v-if="brandInfo.mini_phone" class="number" @click="getPhoneTap(brandInfo.mini_phone)" :style="'color:'+brandInfo.mini_color+';border-color:'+brandInfo.mini_color">{{brandInfo.mini_phone}}</view>
				<view v-if="brandInfo.invter_code" class="intro" @click="getCopyTap(brandInfo.invter_code)" :style="'background-color:'+brandInfo.mini_color+'10'">
					<view class="tit" :style="'color:'+brandInfo.mini_color">{{shopData.shop_name}}专属优惠码</view>
					<view class="num" :style="'color:'+brandInfo.mini_color">{{brandInfo.invter_code}}</view>
				</view>
			</view>
			<view class="qrcode">
				<view v-if="brandInfo.mini_wx_img" class="label" style="margin-right: 50rpx;" :style="'background-color:'+brandInfo.mini_color+'10;border-color:'+brandInfo.mini_color+'60;'">
					<image @click="previewImage(0)" lazy-load :src="qrcodeList[0]" />
					<view class="text">{{brandInfo.mini_wx_title}}</view>
				</view>
				<view v-if="brandInfo.mini_kf_img" class="label" :style="'background-color:'+brandInfo.mini_color+'10;border-color:'+brandInfo.mini_color+'60;'">
					<image @click="previewImage(1)" lazy-load :src="qrcodeList[1]" />
					<view class="text">{{brandInfo.mini_kf_title}}</view>
				</view>
			</view>
		</block>
		
	</view>
</template>

<script>
	import Icon from "@/components/icon/icon.vue"
	export default {
		components: {Icon},
		data() {
			return {
				shopData: uni.getStorageSync('__yipinShopInfo'),
				qrcodeList:[],
				brandInfo: ''
			}
		},
		onLoad() {
			let self = this;
			self.Get(self.Url.brandInfo, {}).then(res => {
				if(res.code === 0){
					self.brandInfo = res.data;
					self.qrcodeList = [res.data.mini_wx_img, res.data.mini_kf_img];
					uni.setNavigationBarColor({
						frontColor: '#ffffff',
						backgroundColor: res.data.mini_color
					})
				}
			})
		},
		onShareAppMessage() {
			let self = this;
			return {
			  title: `${self.brandInfo.mini_title}-${self.brandInfo.mini_intro}`,
			  path: '/pages/about',
			}
		},
		methods: {
			getPhoneTap(phone) {
				uni.makePhoneCall({phoneNumber: phone});
			},
			getCopyTap(text) {
				uni.setClipboardData({
				    data: text,
				    success: function(){}
				});
			},
			previewImage(index) {
				let self = this;
				uni.previewImage({
					urls: self.qrcodeList,
					current: self.qrcodeList[index]
				});
			}
		}
	}
</script>

<style lang="less">
	.headerBox{
		width: 100%;
		height: 290rpx;
		background-color: #00a9f4;
		display: flex;
		flex-direction: column;
		align-items: center;
		.logo{
			display: flex;
			align-items: center;
			width: 85%;
			margin: 0 auto;
			justify-content: center;
			flex-direction: column;
			image{
				width: 70rpx;
				height: 70rpx;
			}
			.title{
				color: #fff;
				font-size: 38rpx;
				font-weight: bold;
				padding-top: 20rpx;
			}
		}
		.text{
			color: #FFFFFF;
			font-size: 26rpx;
			background-color: rgba(255,255,255,.1);
			max-width: 600rpx;
			margin: 0 auto;
			height: 66rpx;
			border-radius: 100rpx;
			margin-top: 36rpx;
			line-height: 66rpx;
			box-shadow: 0 0 10rpx rgba(0,0,0,.05);
			position: relative;
			padding: 0 20rpx;
			span{
				width: 100%;
				white-space: nowrap;
				text-overflow: ellipsis;
				overflow: hidden;
			}
			text{
				border-left: 20rpx solid transparent;
				border-right: 20rpx solid transparent;
				border-bottom: 20rpx solid rgba(255,255,255,.1);
				position: absolute;
				top: -20rpx;
				left: 0;
				right: 0;
				width: 0;
				height: 0;
				margin: 0 auto;
			}
		}
	}
	.infoBox{
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		padding-top: 50rpx;
		.title{
			font-weight: bold;
			font-size: 38rpx;
			color: #222;
		}
		.text{
			color: #999;
			font-size: 28rpx;
			padding-top: 10rpx;
		}
		.number{
			color: #00a9f4;
			font-size: 52rpx;
			font-weight: bold;
			border: 1px solid #6ec0f2;
			width: 420rpx;
			height: 80rpx;
			display: flex;
			align-items: center;
			justify-content: center;
			border-radius: 100rpx;
			margin-top: 30rpx;
		}
		.intro{
			width: 400rpx;
			background-color: rgba(0,169,244,.1);
			margin-top: 30rpx;
			padding: 12rpx 0;
			border-radius: 6rpx;
			.tit{
				font-size: 24rpx;
				text-align: center;
			}
			.num{
				font-size: 48rpx;
				text-align: center;
			}
		}
	}
	.qrcode{
		display: flex;
		align-items: center;
		justify-content: center;
		padding-top: 50rpx;
		.label{
			width: 250rpx;
			background-color: rgba(0,169,244,.1);
			border: 1px solid rgba(0,169,244,.3);
			display: flex;
			flex-direction: column;
			justify-content: center;
			align-items: center;
			padding: 30rpx 0;
			border-radius: 6rpx;
			image{
				width: 200rpx;
				height: 200rpx;
			}
			.text{
				color: #222;
				font-size: 24rpx;
				padding-top: 20rpx;
			}
		}
	}
</style>
