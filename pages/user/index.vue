<template>
	<view class="content">
		<view class="header">
			<view class="bg a" :style="'background-color:'+template.color.color1"></view>
			<view class="bg b" :style="'background-color:'+template.color.color1"></view>
			<view class="bg c" :style="'background-color:'+template.color.color1"></view>
			<view class="infoBox">
				<view class="user">
					<open-data class="avatar" type="userAvatarUrl"></open-data>
					<view class="info">
						<open-data class="name" type="userNickName" :style="'color:'+template.color.color2"></open-data>
						<view v-if="userDetails.level_title" class="score"><text :style="'background-color:'+template.color.color1">{{userDetails.level_title}}</text></view>
						<view v-else class="score none"><text :style="'background-color:'+template.color.color5">非会员</text></view>
					</view>
				</view>
				<view class="menu" :style="'color:'+template.color.color2">
					<view class="label" @click="getMenuListTap('/pages/user/coupons')">
						<view class="num">{{couponList.length}}</view>
						<view class="text">优惠券</view>
					</view>
					<view class="label">
						<view class="num"><text>¥</text>{{userDetails.moneys}}</view>
						<view class="text">余额</view>
					</view>
				</view>
			</view>
		</view>
		<view class="menuBox">
			<view class="label" @click="getMenuListTap('/pages/user/member')">
				<view v-if="template.user.uicon1" class="icons">
					<image :src="template.user.uicon1" lazy-load></image>
				</view>
				<icon v-else class="icons" type="huiyuanlifttime" size="19" :color="template.color.color1"></icon>
				<view class="text" :style="'color:'+template.color.color1">会员中心</view>
				<icon class="left" type="left" size="15" color="#999"></icon>
			</view>
			<view class="label" @click="getMenuListTap('/pages/order/list')">
				<view v-if="template.user.uicon3" class="icons">
					<image :src="template.user.uicon3" lazy-load></image>
				</view>
				<icon v-else class="icons" type="aiguifanfile" size="20" :color="template.color.color2"></icon>
				<view class="text" :style="'color:'+template.color.color2">全部订单 <text v-if="orderStatus.no_payment" :style="'background-color:'+template.color.color1">{{orderStatus.no_payment}}</text></view>
				<icon class="left" type="left" size="15" color="#999"></icon>
			</view>
			<view class="label" @click="getMenuListTap('/pages/user/paypal')">
				<view v-if="template.user.uicon2" class="icons">
					<image :src="template.user.uicon2" lazy-load></image>
				</view>
				<icon v-else class="icons" type="youhui" size="20" :color="template.color.color2"></icon>
				<view class="text" :style="'color:'+template.color.color2">优惠买单</view>
				<icon class="left" type="left" size="15" color="#999"></icon>
			</view>
			<view class="label" @click="getMenuListTap('/pages/user/favorite')">
				<view v-if="template.user.uicon5" class="icons">
					<image :src="template.user.uicon5" lazy-load></image>
				</view>
				<icon v-else class="icons" type="shoucang1" size="20" :color="template.color.color2"></icon>
				<view class="text" :style="'color:'+template.color.color2">我的收藏</view>
				<icon class="left" type="left" size="15" color="#999"></icon>
			</view>
			<view class="label" @click="getMenuListTap('/pages/user/adslist')">
				<view v-if="template.user.uicon4" class="icons">
					<image :src="template.user.uicon4" lazy-load></image>
				</view>
				<icon v-else class="icons" type="shouhuodizhi" size="20" :color="template.color.color2"></icon>
				<view class="text" :style="'color:'+template.color.color2">我的地址</view>
				<icon class="left" type="left" size="15" color="#999"></icon>
			</view>
			
			
			
		</view>
		
		
		<about></about>
	</view>
</template>

<script>
	import Icon from "@/components/icon/icon.vue"
	import About from "@/components/about/about.vue"
	const app = getApp()
	export default {
		components: {Icon, About},
		data() {
			return {
				template: uni.getStorageSync('__yipinTemplateConfig'),
				userDetails: {
					moneys: 0,
					score: 0
				},
				couponList: [],
				orderStatus: ''
			}
		},
		onReady() {
			this.getUserDetails();
			this.getOrderStatus();
			this.getCouponsUserList();
		},
		methods: {
			getOrderStatus() {
				let self = this;
				self.Get(self.Url.orderStatus, {}).then(res => {
					if(res.code === 0){
						self.orderStatus = res.data;
					}
				})
			},
			getMenuListTap(url) {
				if(app.globalData.user){
					uni.navigateTo({url: url});
				}else{
					uni.navigateTo({url: '/pages/user/login'});
				}
			},
			getUserDetails() {
				let self = this;
				if(app.globalData.user){
					self.Get(self.Url.userDetails, {}).then(res => {
						if(res.code === 0){
							self.userDetails = res.data;
						}
					})
				}
			},
			getCouponsUserList() {
				let self = this;
				self.Get(self.Url.couponsUserList, {status: 1}).then(res => {
					if(res.code === 0){
						self.couponList = res.data;
					}
				})
			},
		}
	}
</script>

<style lang="less">
	.header{
		width: 100%;
		height: 350rpx;
		background-color: #fff;
		overflow: hidden;
		position:relative;
		.bg{
			position: absolute;
			width: 200rpx;
			height: 200rpx;
			border-radius: 100%;
			opacity: .1;
			z-index: 1;
		}
		.a{
			top: -50rpx;
			width: 300rpx;
			height: 300rpx;
			left: -50rpx;
		}
		.b{
			width: 120rpx;
			height: 120rpx;
			right: -30rpx;
			top: 100rpx;
		}
		.c{
			bottom: -80rpx;
			right: 130rpx;
		}
		.infoBox{
			position: absolute;
			width: 100%;
			height: 350rpx;
			top: 0;
			left: 0;
			z-index: 2;
			.user{
				display: flex;
				align-items: center;
				width: 90%;
				margin: 0 auto;
				padding: 50rpx 0;
				.avatar{
					display: block;
					overflow: hidden;
					width: 120rpx;
					height: 120rpx;
					border-radius: 100%;
					box-shadow: 0 0 10rpx rgba(0,0,0,.05);
				}
				.info{
					padding-left: 30rpx;
					.name{
						font-weight: bold;
					}
					.score{
						display: flex;
						align-items: center;
						padding-top: 10rpx;
						text{
							font-size: 24rpx;
							padding: 2rpx 16rpx;
							color: #fff;
							border-radius: 100rpx;
						}
					}
				}
			}
			.menu{
				width: 90%;
				display: flex;
				align-items: center;
				justify-content: center;
				margin: 0 auto;
				height: 120rpx;
				.label{
					flex: 0 0 38%;
					display: flex;
					align-items: center;
					flex-direction: column;
					justify-content: center;
					.num{
						font-weight: bold;
						font-size: 36rpx;
						display: flex;
						align-items: flex-end;
						text{
							font-size: 24rpx;
							padding-bottom: 4rpx;
							padding-right: 4rpx;
						}
					}
					.text{
						font-size: 26rpx;
						padding-top: 2rpx;
					}
				}
			}
		}
	}
	.menuBox{
		background-color: #fff;
		padding: 20rpx 0;
		.label{
			display: flex;
			align-items: center;
			justify-content: center;
			width: 90%;
			margin: 0 auto;
			height: 110rpx;
			position: relative;
			.icons{
				flex: 0 0 8%;
				image{
					width: 40rpx;
					height: 40rpx;
				}
			}
			.text{
				flex: 0 0 82%;
				display: flex;
				align-items: center;
				text{
					width: 32rpx;
					height: 32rpx;
					display: flex;
					align-items: center;
					justify-content: center;
					border-radius: 100%;
					font-size: 24rpx;
					color: #fff;
					margin-left: 20rpx;
				}
			}
			.left{
				flex: 0 0 10%;
				display: flex;
				justify-content: flex-end;
			}
		}
	}
</style>
