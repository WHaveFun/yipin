<template>
	<view class="content">
		<view class="headerBox" :style="'background-color:'+template.color.color1">
			<view class="user">
				<open-data class="avatar" type="userAvatarUrl"></open-data>
				<view class="nameBox">
					<open-data class="nickname" type="userNickName"></open-data>
					<view v-if="shopData.level_type=='score'" class="totle">累计获得{{userDetails.total_score}}分</view>
					<view v-if="shopData.level_type=='money'" class="totle">累计消费{{userDetails.total_money}}元</view>
					<view v-if="shopData.level_type=='order'" class="totle">累计购买{{userDetails.total_order}}次</view>
				</view>
			</view>
			<view class="status">
				<view v-if="userDetails.level_id" class="name">{{userDetails.level_title}}</view>
				<view v-else class="name">非会员</view>
				<view class="text" v-if="shopData.level_type=='score'">累计积分达指定分数可自动成为会员</view>
				<view class="text" v-if="shopData.level_type=='money'">累计消费达指定金额可自动成为会员</view>
				<view class="text" v-if="shopData.level_type=='order'">累计购买达指定次数可自动成为会员</view>
			</view>
			<icon class="bgImg" type="huiyuanlifttime" size="100" color="#fff"></icon>
		</view>
		<view v-if="userLevelList.length>0" class="agentListBox">
			<view class="header" :style="'color:'+template.color.color2">会员成长体系</view>
			<view class="menuBox" :style="'background-color:'+template.color.color1+';border-color:'+template.color.color1">
				<view class="menu">级别名称</view>
				<view class="menu">购物折扣</view>
				<view class="menu">
					<text v-if="shopData.level_type=='score'">累计积分(分)</text>
					<text v-if="shopData.level_type=='money'">累计消费(元)</text>
					<text v-if="shopData.level_type=='order'">累计购买(次)</text>
				</view>
			</view>
			<view class="listBox" v-for="(item, index) in userLevelList" :key="index" :style="'color:'+template.color.color2">
				<view class="list">{{item.title}}</view>
				<view class="list">{{item.discount}}%</view>
				<view class="list">
					<text v-if="shopData.level_type=='score'">{{item.level_score}}</text>
					<text v-if="shopData.level_type=='money'">{{item.level_money}}</text>
					<text v-if="shopData.level_type=='order'">{{item.level_order}}</text>
				</view>
			</view>
			<view v-if="shopData.level_type=='score'" class="text" :style="'color:'+template.color.color3">注：当累计积分达指定标准则自动成为该级别会员</view>
			<view v-if="shopData.level_type=='money'" class="text" :style="'color:'+template.color.color3">注：当累计消费达指定标准则自动成为该级别会员</view>
			<view v-if="shopData.level_type=='order'" class="text" :style="'color:'+template.color.color3">注：当累计购买次数达指定标准则自动成为该级别会员</view>
		</view>



	</view>
</template>

<script>
	import Icon from "@/components/icon/icon.vue"
	const app = getApp();
	export default {
		components: {
			Icon
		},
		data() {
			return {
				template: uni.getStorageSync('__yipinTemplateConfig'),
				shopData: uni.getStorageSync('__yipinShopInfo'),
				userDetails: {
					moneys: 0,
					score: 0,
					total_money: 0,
					total_order: 0,
					total_score: 0
				},
				userLevelList: []
			}
		},
		onShow() {
			let self = this;
			self.Get(self.Url.userDetails, {}).then(res => {
				if(res.code === 0){
					self.userDetails = res.data;
				}
			});
			self.Get(self.Url.userLevelList, {}).then(res => {
				if(res.code === 0){
					self.userLevelList = res.data;
				}
			})
		},
		methods: {

		}
	}
</script>

<style lang="less">
	page{
		background-color: #FFFFFF;
	}
	.headerBox {
		width: 90%;
		margin: 0 auto;
		height: 320rpx;
		margin-top: 30rpx;
		box-shadow: 0 6rpx 20rpx rgba(0, 0, 0, .08);
		position: relative;
		overflow: hidden;
		border-radius: 6rpx;

		.bgImg {
			position: absolute;
			top: 0;
			right: -10rpx;
			width: 300rpx;
			height: 320rpx;
			opacity: 0.05;
			display: flex;
			align-items: center;
			justify-content: center;
		}

		.user {
			display: flex;
			width: 90%;
			margin: 0 auto;
			height: 120rpx;
			align-items: flex-end;

			.avatar {
				width: 90rpx;
				height: 90rpx;
				display: block;
				border-radius: 100%;
				overflow: hidden;
			}

			.nameBox {
				color: #fff;
				width: 80%;
				height: 96rpx;
				padding-left: 20rpx;
				display: flex;
				flex-direction: column;
				justify-content: center;
				align-items: flex-start;

				.nickname {
					font-size: 30rpx;
				}

				.totle {
					font-size: 24rpx;
					padding-top: 4rpx;
				}
			}
		}

		.status {
			width: 100%;
			height: 200rpx;
			display: flex;
			flex-direction: column;
			justify-content: center;
			align-items: center;
			color: #fff;

			.name {
				font-size: 46rpx;
			}

			.text {
				font-size: 26rpx;
				padding-top: 12rpx;
			}
		}
	}

	.agentListBox {
		width: 90%;
		margin: 0 auto;
		margin-top: 30rpx;

		.header {
			color: #343434;
			display: flex;
			width: 100%;
			height: 90rpx;
			align-items: center;
			justify-content: center;
			font-size: 36rpx;
			margin-bottom: 20rpx;
		}

		.menuBox {
			display: flex;
			width: 100%;
			height: 80rpx;
			align-items: center;
			border: 2rpx solid #f5f5f5;

			.menu {
				flex: 0 0 33.33%;
				color: #fff;
				display: flex;
				align-items: center;
				justify-content: center;
				height: 80rpx;
			}

			.menu:nth-child(1),
			.menu:nth-child(2) {
				border-right: 2rpx solid rgba(255, 255, 255, .3);
			}
		}

		.listBox {
			display: flex;
			width: 100%;
			height: 80rpx;
			align-items: center;

			.list {
				flex: 0 0 33.33%;
				color: #343434;
				display: flex;
				align-items: center;
				justify-content: center;
				height: 80rpx;
			}

			.list:nth-child(1),
			.list:nth-child(2) {
				border-right: 2rpx solid #eee;
			}
		}

		.listBox:nth-child(odd) {
			border-bottom: 2rpx solid #eee;
			border-left: 2rpx solid #eee;
			border-right: 2rpx solid #eee;
		}

		.listBox:nth-child(even) {
			border-bottom: 2rpx solid #eee;
			border-left: 2rpx solid #eee;
			border-right: 2rpx solid #eee;
			background-color: #f7f7f7;
		}

		.text {
			display: flex;
			height: 80rpx;
			align-items: center;
			justify-content: center;
			color: #999;
			font-size: 24rpx;
		}
	}
</style>
