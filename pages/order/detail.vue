<template>
	<view class="content">
		<block v-if="loading">
			<view class="loadingBox">
				<view class="loading">
					<image lazy-load :src="template.overall.loading"></image>
				</view>
			</view>
		</block>
		<block v-else>
			<view style="height: 30rpx;"></view>
			<view class="orderBox">
				<view class="header" v-if="orderDetail.order_status==0">
					<view class="title" :style="'color:'+template.color.color2">待付款</view>
					<view class="statusBox">
						<view class="status">
							<icon type="daifukuan" size="30" :color="template.color.color5"></icon>
							<view style="font-size: 24rpx;" :style="'color:'+template.color.color5">请在 <countdown v-if="orderDetail.close_time_stamp" @callback="getOrderDetail()" :targetTime="orderDetail.close_time_stamp"></countdown> 内完成支付</view>
						</view>
					</view>
					<view class="button" v-if="orderDetail.delivery_type==2" :style="'background-color:'+template.color.color1" @click="paypalPrderTap(orderDetail.id)">立即支付</view>
				</view>
				<view class="header" v-if="orderDetail.order_status==1">
					<view class="title" :style="'color:'+template.color.color2">待{{orderDetail.delivery_type==1?'配送':'自提'}}</view>
					<view class="statusBox">
						<view class="status">
							<icon type="dingdan" size="25" :color="template.color.color5"></icon>
							<text :style="'color:'+template.color.color5">下单</text>
						</view>
						<view class="line"></view>
						<view class="status">
							<icon type="zhunbeizhong" size="25" :color="template.color.color5"></icon>
							<text :style="'color:'+template.color.color5">准备</text>
						</view>
						<view class="line"></view>
						<view class="status">
							<icon :type="orderDetail.delivery_type==1?'peisong1':'ziqu'" size="25" :color="template.color.color5"></icon>
							<text :style="'color:'+template.color.color5">{{orderDetail.delivery_type==1?'配送':'自提'}}</text>
						</view>
					</view>
					<view class="button" v-if="orderDetail.delivery_type==2" :style="'background-color:'+template.color.color1" @click="qrcodeTap(true)">出示提货码</view>
					<view class="button" v-if="orderDetail.delivery_type==1" :style="'background-color:'+template.color.color4">联系客服<button open-type="contact"></button></view>
				</view>
				<view class="header" v-if="orderDetail.order_status==2">
					<view class="title" :style="'color:'+template.color.color2">配送中</view>
					<view class="statusBox">
						<view class="status">
							<icon type="peisong" size="25" :color="template.color.color5"></icon>
							<text :style="'color:'+template.color.color5">{{orderDetail.logistics_name}}：{{orderDetail.logistics_number}}</text>
						</view>
					</view>
					<view style="display: flex;align-items: center;">
						<view class="button" :style="'background-color:'+template.color.color4">联系客服<button open-type="contact"></button></view>
						<view class="button" v-if="orderDetail.logistics_name=='自营配送'" @click="getPhoneTap(orderDetail.logistics_number)" style="margin-left: 20rpx;" :style="'background-color:'+template.color.color1">联系配送</view>
					</view>
				</view>
				<view class="header" v-if="orderDetail.order_status==3">
					<view class="title" :style="'color:'+template.color.color2">{{orderDetail.delivery_type==1?'已送达':'已自提'}}</view>
					<view class="statusBox">
						<view class="status">
							<icon type="wanchen" size="25" :color="template.color.color1"></icon>
							<text style="padding-top: 6rpx;" :style="'color:'+template.color.color1">感谢惠顾，期待您的下次光临</text>
						</view>
					</view>
				</view>
				<view class="header" v-if="orderDetail.order_status==4">
					<view class="title" :style="'color:'+template.color.color2">{{orderDetail.paypal_type?'已关闭':'已过期'}}</view>
					<view class="statusBox">
						<view class="status">
							<icon type="yiguanbi" size="25" :color="template.color.color5"></icon>
							<text style="padding-top: 6rpx;" :style="'color:'+template.color.color5">{{orderDetail.paypal_type?'订单已关闭，如有疑问请联系客服':'订单因超时未支付已过期'}}</text>
						</view>
					</view>
				</view>
				<view class="content">
					<view class="storeBox">
						<view v-if="orderDetail.delivery_type==1" class="store" style="width: 100%;">
							<view class="name" :style="'color:'+template.color.color2">{{orderDetail.address_data.cname}} /
								{{orderDetail.address_data.phone}}</view>
							<view class="adds" :style="'color:'+template.color.color3">{{orderDetail.address_data.province}}{{orderDetail.address_data.address}}</view>
						</view>
						<view v-if="orderDetail.delivery_type==2" class="store">
						  <view class="name">{{shopData.shop_name}}</view>
						  <view class="adds">{{shopData.shop_address}}</view>
						</view>
						<view v-if="orderDetail.delivery_type==2" class="buttonBox">
						  <view v-if="shopData.shop_phone" class="button" @click="getPhoneTap(shopData.shop_phone)">
							  <icon type="dianhua" size="31" :color="template.color.color2"></icon>
						  </view>
						  <view v-if="template.overall.latitude" class="button" @click="getLocalTap()">
							  <icon type="daohang" size="32" :color="template.color.color2"></icon>
						  </view>
						</view>
					</view>
					<view class="lineBox">
						<view class="left"></view>
						<view class="right"></view>
					</view>
					<view class="goodsBox">
						<view class="goods" v-for="(item, index) in orderDetail.goods_list" :key="index">
							<view class="infoBox">
								<view class="title" :style="'color:'+template.color.color2">{{item.goods_title}}</view>
								<view v-if="item.goods_spec_name" class="label" :style="'color:'+template.color.color3">
									<text v-for="(value, name) in item.goods_spec_name" :key="name">{{name}}:{{value}}</text>
								</view>
							</view>
							<view class="number" :style="'color:'+template.color.color2">x{{item.buy_num}}</view>
							<view class="price" :style="'color:'+template.color.color2">¥{{item.goods_price}}</view>
						</view>
						<view class="goods">
							<view class="infoBox">
								<view class="title" :style="'color:'+template.color.color3">商品合计<text>(共{{orderDetail.goods_number}}件商品)</text></view>
							</view>
							<view class="number"></view>
							<view class="price" :style="'color:'+template.color.color3">¥{{orderDetail.goods_total}}</view>
						</view>
						<view class="goods" v-if="orderDetail.coupons_money">
							<view class="infoBox">
								<view class="title" :style="'color:'+template.color.color3">优惠抵扣<text>({{orderDetail.coupons_title}})</text></view>
							</view>
							<view class="number"></view>
							<view class="price" :style="'color:'+template.color.color3">-¥{{orderDetail.coupons_money}}</view>
						</view>
						<view class="goods" v-if="orderDetail.user_money">
							<view class="infoBox">
								<view class="title" :style="'color:'+template.color.color3">会员折扣<text>({{orderDetail.user_level}}{{orderDetail.user_count*10}}折)</text></view>
							</view>
							<view class="number"></view>
							<view class="price" :style="'color:'+template.color.color3">-¥{{orderDetail.user_money}}</view>
						</view>
						<view class="goods" v-if="orderDetail.freight_money">
							<view class="infoBox">
								<view class="title" :style="'color:'+template.color.color3">配送费</view>
							</view>
							<view class="number"></view>
							<view class="price" :style="'color:'+template.color.color3">¥{{orderDetail.freight_money}}</view>
						</view>
						<view class="total">
							<view class="name" :style="'color:'+template.color.color2">合计</view>
							<view class="price" :style="'color:'+template.color.color2">¥{{orderDetail.total_money}}</view>
						</view>
					</view>
					<view class="lineBox">
						<view class="left"></view>
						<view class="right"></view>
					</view>
					<view class="footer">
						<view class="label">
							<view class="title">订单编号</view>
							<view class="nums">{{orderDetail.order_number}}</view>
						</view>
						<view class="label">
							<view class="title">下单时间</view>
							<view class="nums">{{orderDetail.create_time}}</view>
						</view>
						<view class="label" v-if="orderDetail.paypal_time">
							<view class="title">支付时间</view>
							<view class="nums">{{orderDetail.paypal_time}}</view>
						</view>
						<view class="label" v-if="orderDetail.paypal_type">
							<view class="title">支付方式</view>
							<view class="nums">
								<text v-if="orderDetail.paypal_type==1">微信支付</text>
								<text v-if="orderDetail.paypal_type==2">货到付款</text>
								<text v-if="orderDetail.paypal_type==3">余额支付</text>
								<text v-if="orderDetail.paypal_type==4">线下支付</text>
								<text v-if="orderDetail.paypal_type==5">积分支付</text>
							</view>
						</view>
						<view class="label" v-if="orderDetail.remark_user">
							<view class="title">备注信息</view>
							<view class="nums">{{orderDetail.remark_user}}</view>
						</view>
					</view>
				</view>
			</view>
			<view style="height: 50rpx;"></view>
			
			<view class="showContent" :style="paypalShow?'bottom:0;':''">
				<view class="bg" @click="closePaypalShow()"></view>
				<view class="showBox">
					<view class="pheader" :style="'color:'+template.color.color5">请选择支付方式</view>
					<view class="paypalList">
						<view class="paypal lineBottom" v-for="(item, index) in shopData.shop_paypal" :key="index"
						 @click="getPaypalOrder(item.lid, item.pdata)">
							<view class="left">
								<icon v-if="item.lid==1" type="weixinzhifu" size="30" color="#00c250"></icon>
								<icon v-if="item.lid==2" type="huodaofukuan" size="30" color="#ff9900"></icon>
								<icon v-if="item.lid==3" type="yuezhifu" size="30" :color="template.color.color1"></icon>
								<icon v-if="item.lid==5" type="jifen1" size="35" :color="template.color.color1"></icon>
							</view>
							<view class="right">
								<view class="title" :style="'color:'+template.color.color2">{{item.cname}}</view>
								<view v-if="item.lid==1" class="info" :style="'color:'+template.color.color5">简单快捷，推荐使用</view>
								<view v-if="item.lid==2" class="info" :style="'color:'+template.color.color5">购物零风险，更安全</view>
								<view v-if="item.lid==3" class="info" :style="'color:'+template.color.color5">可用余额：¥{{userData.moneys}}</view>
								<view v-if="item.lid==5" class="info" :style="'color:'+template.color.color5">可用积分：{{userData.score}}（{{item.pdata}}积分=1元）</view>
							</view>
							<view class="icon">
								<icon type="left" size="18" color="#e2e2e2"></icon>
							</view>
						</view>
					</view>
				</view>
			</view>
			<view class="showContent" :style="qrcodeShow?'bottom:0;':''">
				<view class="bg" @click="qrcodeTap(false)"></view>
				<view class="showBox" style="min-height: 550rpx;">
					<view class="pheader" :style="'color:'+template.color.color2">请向店员出示提货码</view>
					<view class="images">
						<image :src="imageQrPath" />
					</view>
					<view class="qrText" :style="'background-color:'+template.color.color4" @click="copyVerificationCode()">{{orderDetail.verification_code}}</view>
				</view>
			</view>
		</block>
		<canvas canvas-id="canvas" style="position:fixed;left:-1000rpx;bottom:-1000rpx;width:200px;height:200px;opacity: 0;"></canvas>
	</view>
</template>

<script>
	import QRCode from '@/static/utils/qrcode.js'
	import Icon from "@/components/icon/icon.vue"
	import Countdown from "@/components/countdown/countdown.vue"
	export default {
		components: {
			Icon,
			Countdown
		},
		data() {
			return {
				loading: true,
				IphoneX: this.IphoneX,
				template: uni.getStorageSync('__yipinTemplateConfig'),
				shopData: uni.getStorageSync('__yipinShopInfo'),
				orderId: 0,
				orderDetail: '',
				paypalShow: false,
				imageQrPath: '',
				qrcodeShow: false,
				userData: {
					moneys: 0,
					score: 0
				}
			}
		},
		onShow() {
			this.getOrderDetail();
		},
		onLoad(e) {
			let self = this;
			if (e.id) {
				self.orderId = e.id;
			}
		},
		methods: {
			getUserDetails() {
				let self = this;
				self.Get(self.Url.userDetails, {}).then(res => {
					if (res.code === 0) {
						self.userData = res.data;
					}
				});
			},
			getOrderDetail() {
				let self = this;
				self.Get(self.Url.orderDetail, {
					id: self.orderId
				}).then(res => {
					if (res.code === 0) {
						self.loading = false;
						self.orderDetail = res.data;
						if(res.data.address_data.province){
							self.orderDetail.address_data.province = res.data.address_data.province.replace(/,/g,'');
						}
						self.getUserDetails();
					}
				})
			},
			qrcodeTap(status) {
				let self = this;
				if (status) {
					let code = self.orderDetail.verification_code;
					QRCode.api.draw(code, 'canvas', 200, 200);
					setTimeout(() => {
						self.canvasToQrTempImage();
					}, 300);
				} else {
					self.qrcodeShow = false;
				}
			},
			canvasToQrTempImage: function() {
				let self = this;
				wx.canvasToTempFilePath({
					canvasId: 'canvas',
					success: function(res) {
						self.imageQrPath = res.tempFilePath;
						self.qrcodeShow = true;
					}
				});
			},
			getPhoneTap(number) {
				uni.makePhoneCall({
					phoneNumber: number
				});
			},
			getLocalTap() {
				let self = this;
				let shop = self.shopData;
				let data = self.template.overall;
				uni.openLocation({
					name: shop.shop_name,
					address: shop.shop_address,
				    latitude: data.latitude,
				    longitude: data.longitude
				});
			},
			paypalPrderTap(index) {
				if (index != null) {
					this.paypalShow = true;
				} else {
					this.paypalShow = false;
				}
			},
			getPaypalOrder(id) {
				let self = this;
				// 1 微信支付
				// 2 货到付款
				// 3 余额支付
				// 5 积分支付
				let postData = {
					pay_type: id,
					order_id: self.orderDetail.id,
					platform: 1,
				}
				if (id == 3 && self.userData.moneys < self.orderDetail.total_money) {
					uni.showModal({
						title: '余额支付提示',
						content: '您的可用余额不足以支付该订单，请更换其他支付方式',
						showCancel: false
					})
					return
				}
				if (id === 5 && (self.userData.score / number) < self.orderData.total_money) {
					uni.showModal({
						title: '积分支付提示',
						content: '您的可用积分不足以支付该订单，请更换其他支付方式',
						showCancel: false
					})
					return
				}
				self.Get(self.Url.orderPaypal, postData).then(res => {
					if (res.code === 0) {
						if (id === 1) {
							uni.requestPayment({
								provider: 'wxpay',
								timeStamp: res.data.wechat.timeStamp,
								nonceStr: res.data.wechat.nonceStr,
								package: res.data.wechat.package,
								signType: res.data.wechat.signType,
								paySign: res.data.wechat.paySign,
								success: function(res) {
									uni.showToast({
										title: '微信支付成功'
									});
									self.getOrderDetail();
									self.paypalShow = false;
								},
								fail: function(err) {
									uni.showModal({
										title: '支付失败',
										content: '订单支付失败，请重新进行支付。如针对订单金额有疑问请联系客服人员',
										showCancel: false
									})
									return
								}
							});
						} else if (id === 2) {
							uni.showModal({
								title: '货到付款提示',
								content: '收货前请检查快递包装有无破损，如果需要拒收请先联系客服人员',
								showCancel: false
							})
							self.paypalShow = false;
							self.getOrderDetail();
							return
						} else if (id === 3) {
							uni.showToast({
								title: '余额支付成功'
							});
							self.paypalShow = false;
							self.getOrderDetail();
						} else if (id === 5) {
							uni.showToast({
								title: '积分支付成功'
							});
							self.paypalShow = false;
							self.getOrderDetail();
						}
					}else{
						uni.showModal({
							content: res.msg,
							showCancel: false
						})
						return
					}
				})
			},
			copyOrderLogistics() {
				let data = this.orderDetail;
				uni.setClipboardData({
					data: data.logistics_name + '\r\n' + data.logistics_number,
					success: function() {}
				});
			},
			copyVerificationCode() {
				let data = this.orderDetail;
				uni.setClipboardData({
					data: data.verification_code,
					success: function() {}
				});
			},
			confirmOrder() {
				let self = this;
				self.Get(self.Url.orderConfirm, {
					id: self.orderDetail.id
				}).then(res => {
					if (res.code === 0) {
						uni.showToast({
							title: '确认收货成功'
						});
						self.getOrderDetail();
					}
				});
			},
			evaluateGoods() {
				uni.navigateTo({
					url: '/pages/order/evaluate?id=' + this.orderDetail.id
				});
			},
		}
	}
</script>

<style lang="less">
	page {
		background-color: #e7e7e7;
	}

	.orderBox {
		width: 90%;
		margin: 0 auto;
		border-radius: 20rpx;
		overflow: hidden;
		position: relative;

		.lineBox {
			position: relative;
			width: 100%;
			height: 30rpx;
			margin-top: -16rpx;

			.left {
				position: absolute;
				left: -12rpx;
				width: 30rpx;
				height: 30rpx;
				background-color: #e7e7e7;
				border-radius: 100%;
			}

			.right {
				position: absolute;
				right: -12rpx;
				width: 30rpx;
				height: 30rpx;
				background-color: #e7e7e7;
				border-radius: 100%;
			}
		}

		.header {
			width: 100%;
			padding: 10rpx 0 30rpx;
			background-color: #fff;
			display: flex;
			flex-direction: column;
			align-items: center;
			justify-content: center;
			box-shadow: 0 0 10rpx rgba(0, 0, 0, 0.05);
			position: relative;

			.title {
				font-size: 30rpx;
				font-weight: bold;
				height: 100rpx;
				width: 100%;
				display: flex;
				justify-content: center;
				align-items: center;
			}

			.statusBox {
				display: flex;
				align-items: center;
				justify-content: center;
				position: relative;
				width: 100%;
				height: 160rpx;

				.status {
					display: flex;
					align-items: center;
					flex-direction: column;
					justify-content: center;

					text {
						font-size: 24rpx;
					}
				}

				.line {
					width: 80rpx;
					border-top: 1px dashed #e2e2e2;
					height: 1px;
					margin: 0 12rpx;
					margin-top: -30rpx;
				}
			}
			.button{
				padding: 8rpx 20rpx;
				font-size: 24rpx;
				color: #fff;
				border-radius: 100rpx;
				position: relative;
				button{
					position: absolute;
					top: 0;
					left: 0;
					width: 100%;
					height: 100%;
					opacity: 0;
				}
			}
		}

		.content {
			width: 100%;
			height: 67%;
			background-color: #f7f7f7;

			.storeBox {
				width: 90%;
				margin: 0 auto;
				display: flex;
				align-items: center;
				border-bottom: 2rpx solid #e7e7e7;
				padding-bottom: 30rpx;
				padding-top: 30rpx;

				.store {
					width: 70%;
					display: flex;
					flex-direction: column;
					align-content: center;
					justify-content: center;

					.name {
						color: #343434;
						width: 100%;
						text-overflow: ellipsis;
						white-space: nowrap;
						overflow: hidden;
					}

					.adds {
						color: #999;
						font-size: 26rpx;
						overflow: hidden;
						text-overflow: ellipsis;
						display: -webkit-box;
						-webkit-line-clamp: 2;
						-webkit-box-orient: vertical;
					}
				}

				.buttonBox {
					width: 30%;
					display: flex;
					align-items: center;
					justify-content: flex-end;

					.button {
						width: 80rpx;
						height: 80rpx;
						border-radius: 100%;
						display: flex;
						align-items: center;
						justify-content: center;
						margin-left: 20rpx;
					}
				}
			}

			.goodsBox {
				margin-top: 12rpx;
				min-height: 250rpx;

				.goods {
					display: flex;
					width: 90%;
					margin: 0 auto;
					color: #343434;
					padding-bottom: 30rpx;

					.infoBox {
						width: 50%;

						.title {
							width: 100%;
							text-overflow: ellipsis;
							white-space: nowrap;
							overflow: hidden;

							text {
								font-size: 24rpx;
								padding-left: 10rpx;
							}
						}

						.label {
							color: #999;
							font-size: 26rpx;
							overflow: hidden;
							text-overflow: ellipsis;
							display: -webkit-box;
							-webkit-line-clamp: 2;
							-webkit-box-orient: vertical;
						}
					}

					.number {
						width: 25%;
						display: flex;
						justify-content: flex-end;
						padding-right: 10rpx;
					}

					.price {
						width: 25%;
						display: flex;
						justify-content: flex-end;
					}
				}

				.total {
					width: 90%;
					display: flex;
					margin: 0 auto;
					border-top: 1px dashed #e7e7e7;
					height: 90rpx;
					align-items: center;

					.name {
						width: 50%;
						display: flex;
						justify-content: flex-start;
						color: #343434;
					}

					.price {
						width: 50%;
						display: flex;
						justify-content: flex-end;
						color: #343434;
					}
				}
			}
			.footer {
				border-top: 2rpx solid #e7e7e7;
				width: 90%;
				margin: 0 auto;
				margin-top: -16rpx;
				padding-top: 30rpx;
				color: #999;
				font-size: 26rpx;
				line-height: 42rpx;
				padding-bottom: 30rpx;
				.label{
					display: flex;
					align-items: center;
					height: 70rpx;
					.title{
						flex: 0 0 25%;
					}
					.nums{
						flex: 0 0 75%;
						display: flex;
						justify-content: flex-end;
					}
				}
			}
		}
	}
	
	.showContent{
		position: fixed;
		bottom: -2000rpx;
		left: 0;
		width: 100%;
		height: 100%;
		z-index: 1000;
		transition: all .5s;
		.bg{
			position: absolute;
			z-index: 1001;
			width: 100%;
			height: 100%;
			top: 0;
			left: 0;
		}
		.showBox{
			position: absolute;
			bottom: 0;
			width: 100%;
			min-height: 600rpx;
			z-index: 1002;
			background-color: #fff;
			border-top-left-radius: 20rpx;
			border-top-right-radius: 20rpx;
			box-shadow: 0 0 20rpx rgba(0,0,0,.08);
			.pheader{
				font-size: 32rpx;
				text-align: center;
				
				padding: 20rpx 0 10rpx;
			}
			.qrText{
				background-color: #CAB88F;
				width: 300rpx;
				margin: 0 auto;
				display: flex;
				align-items: center;
				justify-content: center;
				height: 60rpx;
				border-radius: 6rpx;
				color: #fff;
				
				font-size: 30rpx;
			}
			.images{
				width: 400rpx;
				height: 350rpx;
				margin: 0 auto;
				overflow: hidden;
				display: flex;
				align-items: center;
				justify-content: center;
				image{
					width: 400rpx;
					height: 400rpx;
				}
			}
			.payTime{
				text-align: center;
				padding: 20rpx 0 0;
				
			}
			.paypalList{
				padding-top: 20rpx;
				padding-bottom: 80rpx;
				.paypal{
					display: flex;
					align-items: center;
					width: 90%;
					margin: 0 auto;
					position: relative;
					padding: 20rpx 0;
					.left{
						flex: 0 0 18%;
						display: flex;
						align-items: center;
						justify-content: center;
					}
					.right{
						flex: 0 0 75%;
						
						.info{
							font-size: 26rpx;
						}
					}
					.icon{
						flex: 0 0 7%;
					}
				}
			}
		}
	}
</style>
