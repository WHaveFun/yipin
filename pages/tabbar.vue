<template>
	<view class="content" :style="current===1?'background-color: #e7e7e7;':''">
		<block v-if="loading">
			<view class="loadingBox">
				<view class="loading">
					<image lazy-load :src="template.overall.loading"></image>
				</view>
			</view>
		</block>
		<block v-else>
			<block v-if="template">
				<index :goods="goods" @gooods="backgoods" v-if="current==0"></index>
				<order v-if="current==1" @tabbbar="gotohome"></order>
				<user v-if="current==2"></user>
				<view :style="'height:'+(IphoneX>=0?'160rpx':'130rpx')"></view>
				<view class="tabbarBox lineTop" :class="IphoneX>=0?'tabbarX':''">
					<view class="list" :class="current==0?'active':''" @click="selectTabbar(0)">
						<image v-if="template.tabbar.ticon1" :src="current==0?template.tabbar.ticon1:template.tabbar.ticon2" mode="aspectFill"></image>
						<image v-else :src="current==0?'../static/images/tabbar/1a.png':'../static/images/tabbar/1b.png'" mode="aspectFill"></image>
						<view class="name" :style="'color:'+(current==0?template.color.color2:template.color.color5)">{{template.tabbar.dtitle}}</view>
					</view>
					<view class="list" :class="current==1?'active':''" @click="selectTabbar(1)">
						<image v-if="template.tabbar.ticon3" :src="current==1?template.tabbar.ticon3:template.tabbar.ticon4" mode="aspectFill"></image>
						<image v-else :src="current==1?'../static/images/tabbar/2a.png':'../static/images/tabbar/2b.png'" mode="aspectFill"></image>
						<view class="name" :style="'color:'+(current==1?template.color.color2:template.color.color5)">{{template.tabbar.qtitle}}</view>
					</view>
					<view class="list" :class="current==2?'active':''" @click="selectTabbar(2)">
						<image v-if="template.tabbar.ticon5" :src="current==2?template.tabbar.ticon5:template.tabbar.ticon6" mode="aspectFill"></image>
						<image v-else :src="current==2?'../static/images/tabbar/3a.png':'../static/images/tabbar/3b.png'" mode="aspectFill"></image>
						<view class="name" :style="'color:'+(current==2?template.color.color2:template.color.color5)">{{template.tabbar.utitle}}</view>
					</view>
				</view>
			</block>
			<block v-else>
				<view class="noneBox">
					<view class="noneText">
						<view class="title">模版出现异常</view>
						<view class="text">可能出现的问题有：</view>
						<view class="text">1、没有该模版的使用权限</view>
						<view class="text">2、小程序appID密钥信息未配置</view>
						<view class="text">3、后台填写的appID和当前小程序不一致</view>
						<view class="text">4、后台余额不足导致账单扣费失败，请先充值</view>
						<view class="text">5、如果以上均为解决您的问题请联系模版开发人员</view>
						<view class="tips dev">调试信息：</view>
						<view class="tips">brand：{{wxinfos.brand}}</view>
						<view class="tips">model：{{wxinfos.model}}</view>
						<view class="tips">system：{{wxinfos.system}}</view>
						<view class="tips">version：{{wxinfos.version}}</view>
						<view class="tips">SDKVersion：{{wxinfos.SDKVersion}}</view>
						<view class="tips">appId：{{wxappid}}</view>
					</view>
				</view>
			</block>
		</block>
		
		<canvas canvas-id="goodsHaibao" style="width:600px;height:1000px;opacity:0;position:fixed;left:-1000px;top:-1000px;"></canvas>
		<canvas canvas-id="canvas" style="width:200px;height:200px;opacity:0;position:fixed;left:-1000px;top:-1000px;"></canvas>
	</view>
</template>

<script>
	import Index from './index/index.vue'
	import Order from './order/index.vue'
	import User from './user/index.vue'
	import Icon from "@/components/icon/icon.vue"
	const app = getApp();
	export default {
		components: {
			Index, Order, User, Icon
		},
		data() {
			return {
				loading: true,
				active: false,
				current: -1,
				IphoneX: this.IphoneX,
				wxinfos: uni.getSystemInfoSync(),
				wxappid: uni.getAccountInfoSync().miniProgram.appId,
				template: '',
				goods: 0
			}
		},
		onShareAppMessage(e) {
			let shop = uni.getStorageSync('__yipinShopInfo');
			let id = e.target.dataset.goods.id;
			let pic = e.target.dataset.goods.pic;
			let title = e.target.dataset.goods.title;
			return {
			  title: title?title:shop.shop_name,
			  path: title?('/pages/tabbar?goods='+id):'/pages/tabbar',
			  imageUrl: pic
			}
		},
		onLoad(e) {
			let self = this;
			self.Config().then(res => {
				if(res.code === 0){
					let data = res.data;
					data.overall.loading = 'https://ddmini.com/loading.svg';
					self.template = data;
					if(e.scene){
						let scene = decodeURIComponent(e.scene);
						let goods = scene.split('=')[1];
						self.goods = goods;
					}
					if(e.goods){
						self.goods = e.goods;
					}
					if(e.order){
						uni.navigateTo({url: '/pages/order/list?id='+e.order});
					}
					if(e.cid){
						self.current = e.cid;
					}else{
						self.current = 0;
					}
					self.loading = false;
					uni.setStorageSync('__yipinTemplateConfig', data);
				}else{
					self.loading = false;
					uni.removeStorageSync('__yipinTemplateConfig');
				}
			});
		},
		methods: {
			backgoods(){
				this.goods = 0;
			},
			gotohome(){
				this.current = 0;
			},
			selectTabbar(id) {
				let self = this;
				self.current = id;
				return;
				if(id>=1){
					if(app.globalData.user){
						self.current = id;
					}else{
						uni.navigateTo({url: '/pages/user/login'});
					}
				}else{
					self.current = id;
				}
			}
		}
	}
</script>

<style lang="less">
	.tabbarBox {
		position: fixed;
		width: 100%;
		height: 120rpx;
		display: flex;
		justify-content: center;
		bottom: 0;
		background-color: #fff;
		//box-shadow: 0 0 10rpx rgba(0,0,0,.01);
		z-index: 900;

		.list {
			flex: 0 0 33.333333%;
			display: flex;
			flex-direction: column;
			align-items: center;
			padding-top: 16rpx;
			transition: all 0.3s;
			
			image{
				width: 52rpx;
				height: 52rpx;
			}
			
			.name {
				color: #6e6d6b;
				font-size: 24rpx;
				padding-top: 4rpx;
			}
		}

		.active {
			.name {
				color: #e64340;
			}
		}
	}

	.tabbarX {
		height: 146rpx;
	}
	
	.noneBox{
		display: flex;
		align-items: center;
		justify-content: center;
		height: 80vh;
		.noneText{
			font-size: 24rpx;
			border: 1px solid #f2f2f2;
			background-color: #FFFFFF;
			width: 550rpx;
			margin: 0 auto;
			padding: 20rpx;
			box-shadow: 0 10rpx 20rpx rgba(0, 0, 0, 0.05);
			.title{
				font-weight: bold;
				padding-bottom: 20rpx;
				text-align: center;
			}
			.text{
				color: #999;
				line-height: 50rpx;
			}
			.tips{
				color: #4CD964;
				padding-top: 6rpx;
			}
			.dev{
				padding-top: 20rpx;
			}
		}
	}
</style>
