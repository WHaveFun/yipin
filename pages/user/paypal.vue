<template>
	<view class="content">
		<view style="height: 50rpx;"></view>
		<view v-if="paypalStatus" class="successBox">
			<view class="info">
				<icon type="wanchen" :color="template.color.color1" size="38"></icon>
				<view class="text" :style="'color:'+template.color.color1">支付成功</view>
				<view class="money" :style="'color:'+template.color.color2"><text>¥</text>{{checkpayOrder.moneys-checkpayOrder.reduce}}</view>
			</view>
			<view v-if="checkpayOrder.reduce" class="youhui">
				<view class="label" :style="'color:'+template.color.color3">
					<view class="left">原价</view>
					<view class="right">¥{{checkpayOrder.moneys}}</view>
				</view>
				<view class="label" style="padding-top: 10rpx;" :style="'color:'+template.color.color1">
					<view class="left">优惠</view>
					<view class="right">-¥{{checkpayOrder.reduce}}</view>
				</view>
			</view>
			<view class="button" @click="[paypalStatus=false, paypalNumber='']" :style="'background-color:'+template.color.color1">继续买单</view>
		</view>
		<view v-else class="paypalBox">
			<view class="header">
				<view class="left">
					<view class="title" :style="'color:'+template.color.color2">{{shopData.shop_name}}</view>
					<view class="addrs" :style="'color:'+template.color.color3">{{shopData.shop_address}}</view>
				</view>
				<view class="right">
					<image mode="aspectFill" :src="shopData.shop_logo" />
				</view>
			</view>
			<view class="moneyBox">
				<view class="title" :style="'color:'+template.color.color3">买单金额</view>
				<view class="input">
					<view class="text" :style="'color:'+template.color.color2">¥</view>
					<input type="digit" maxlength="6" focus :value="paypalNumber" @input="bindinput" placeholder="请询问店员后输入" :style="'color:'+template.color.color2" />
				</view>
				<view v-if="paypalNumber" class="button" :style="'background-color:'+template.color.color1" @click="getPayBillTap()">确认付款</view>
				<view v-else class="button" :style="'background-color:'+template.color.color5">确认付款</view>
			</view>
			<view class="payListBox" v-if="checkpayList.length>0">
				<view class="title" :style="'color:'+template.color.color3">买单优惠</view>
				<view class="payList">
					<view class="listBox" v-for="(item, index) in checkpayList" :key="index" :style="'border-color:'+template.color.color1+';color:'+template.color.color1">
						<view class="list" @click="getPayNumber(item.moneys)">
							<view class="name">满{{item.moneys}}元</view>
							<view class="text">减{{item.reduce}}元</view>
						</view>
					</view>
				</view>
			</view>
		</view>



	</view>
</template>

<script>
	import Icon from "@/components/icon/icon.vue"
	const app = getApp()
	export default {
		components: {
			Icon
		},
		data() {
			return {
				template: uni.getStorageSync('__yipinTemplateConfig'),
				shopData: uni.getStorageSync('__yipinShopInfo'),
				paypalNumber: '',
				checkpayList: [],
				paypalStatus: false,
				checkpayOrder: ''
			}
		},
		onLoad() {
			let self = this;
			self.Get(self.Url.checkpayList, {}).then(res => {
				if (res.code === 0) {
					self.checkpayList = res.data;
				}
			})
		},
		methods: {
			bindinput(e) {
				this.paypalNumber = e.detail.value;
			},
			getPayNumber(number) {
				this.paypalNumber = number;
			},
			getPayBillTap() {
				let self = this;
				if(app.globalData.user){
					if(self.paypalNumber){
						uni.showLoading({title: '请稍等...'});
						self.Get(self.Url.checkpayOrder, {money: self.paypalNumber}).then(res => {
							self.checkpayOrder = res.data;
							if(res.code === 0){
								let postData = {
									order_id: res.data.order_id,
									pay_type: 1,
									platform: 1,
									type: 4
								}
								self.Get(self.Url.orderPaypal, postData).then(res => {
									uni.hideLoading();
									if(res.code === 0){
										uni.requestPayment({
										    provider: 'wxpay',
										    timeStamp: res.data.wechat.timeStamp,
										    nonceStr: res.data.wechat.nonceStr,
										    package: res.data.wechat.package,
										    signType: res.data.wechat.signType,
										    paySign: res.data.wechat.paySign,
										    success: function (res) {
												uni.showToast({title: '微信支付成功'});
												self.paypalStatus = true;
										    },
										    fail: function (err) {
										        uni.showModal({
										        	title: '支付失败',
													content: '支付失败，请重新进行支付!',
													showCancel: false
										        })
												return
										    }
										});
									}
								})
							}else{
								uni.hideLoading();
								uni.showToast({title: res.msg, icon: 'none'});
							}
						})
					}else{
						uni.showToast({title: '请输入买单金额', icon: 'none'});
					}
				}else{
					uni.navigateTo({url: '/pages/user/login'});
				}
			}
		}
	}
</script>

<style lang="less">
	page {
		background-color: #e7e7e7;
	}
	.successBox{
		width: 80%;
		background-color: #fff;
		margin: 0 auto;
		min-height: 500rpx;
		border-radius: 6rpx;
		overflow: hidden;
		text-align: center;
		box-shadow: 0 8rpx 20rpx rgba(0, 0, 0, .03);
		.info{
			padding-top: 50rpx;
			.text{
				padding-top: 6rpx;
			}
			.money{
				font-size: 66rpx;
				padding-top: 50rpx;
				display: flex;
				justify-content: center;
				font-weight: bold;
				text{
					font-size: 38rpx;
					padding-right: 8rpx;
					padding-top: 14rpx;
				}
			}
		}
		.youhui{
			width: 60%;
			margin: 0 auto;
			border-top: 1px solid #f5f5f5;
			margin-top: 30rpx;
			.label{
				display: flex;
				align-items: center;
				padding-top: 16rpx;
				font-size: 24rpx;
				.left{
					flex: 0 0 50%;
					display: flex;
				}
				.right{
					flex: 0 0 50%;
					display: flex;
					justify-content: flex-end;
				}
			}
		}
		.button{
			background-color: #D3AA79;
			width: 50%;
			height: 60rpx;
			display: flex;
			align-items: center;
			justify-content: center;
			color: #fff;
			margin: 0 auto;
			margin-bottom: 50rpx;
			margin-top: 50rpx;
			border-radius: 6rpx;
			font-size: 26rpx;
		}
	}
	.paypalBox {
		width: 85%;
		background-color: #fff;
		margin: 0 auto;
		min-height: 500rpx;
		border-radius: 6rpx;
		overflow: hidden;
		box-shadow: 0 8rpx 20rpx rgba(0, 0, 0, .03);

		.header {
			display: flex;
			align-items: center;
			justify-content: center;
			background-color: #f5f5f5;
			height: 120rpx;

			.left {
				flex: 0 0 70%;

				.title {
					font-weight: bold;
					font-size: 28rpx;
					width: 90%;
					text-overflow: ellipsis;
					white-space: nowrap;
					overflow: hidden;
				}

				.addrs {
					color: #999;
					font-size: 24rpx;
					width: 90%;
					padding-top: 8rpx;
					text-overflow: ellipsis;
					white-space: nowrap;
					overflow: hidden;
				}
			}

			.right {
				flex: 0 0 20%;
				display: flex;
				justify-content: flex-end;

				image {
					width: 80rpx;
					height: 80rpx;
					border-radius: 100%;
				}
			}
		}

		.moneyBox {
			width: 90%;
			margin: 0 auto;
			padding-top: 30rpx;

			.title {
				color: #999;
				height: 60rpx;
				display: flex;
				align-items: center;
				font-size: 26rpx;
			}

			.input {
				display: flex;
				width: 100%;
				height: 100rpx;
				border-bottom: 2rpx solid #f5f5f5;
				align-items: center;

				.text {
					width: 15%;
					color: #343434;
					font-size: 42rpx;
					font-weight: bold;
					display: flex;
					align-items: center;
					justify-content: center;
				}

				input {
					width: 85%;
					font-size: 38rpx;
					color: #343434;
				}
			}

			.button {
				background-color: #e7e7e7;
				width: 100%;
				height: 80rpx;
				margin-top: 38rpx;
				color: #fff;
				display: flex;
				align-items: center;
				justify-content: center;
				border-radius: 6rpx;
			}

			.active {
				background-color: #d3aa79;
			}
		}

		.payListBox {
			width: 90%;
			margin: 0 auto;
			padding-top: 60rpx;
			padding-bottom: 20rpx;

			.title {
				color: #999;
				height: 60rpx;
				display: flex;
				align-items: flex-start;
				font-size: 26rpx;
			}

			.payList {
				display: flex;
				align-items: center;
				flex-wrap: wrap;

				.listBox {
					flex: 0 0 33.333%;
					display: flex;
					align-items: center;
					justify-content: center;
					margin-bottom: 20rpx;

					.list {
						width: 90%;
						height: 110rpx;
						border: 2rpx solid #d3aa79;
						color: #d3aa79;
						display: flex;
						flex-direction: column;
						align-items: center;
						justify-content: center;
						border-radius: 4rpx;

						.name {
							font-size: 24rpx;
						}

						.text {
							font-size: 26rpx;
							font-weight: 500;
						}
					}
				}
			}
		}
	}
</style>
