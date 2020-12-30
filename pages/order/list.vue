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
			<view v-if="orderList.length>0" class="orderListBox">
				<view class="list" v-for="(item, index) in orderList" :key="index" @click="getOrderDetail(item.id)">
					<view class="oHeader lineBottom">
						<view class="left" :style="'color:'+template.color.color2"><text>{{shopData.shop_name}}</text></view>
						<view class="right" :style="'color:'+template.color.color3">
							<text v-if="item.order_status==0">待付款</text>
							<text v-if="item.order_status==1 && item.delivery_type==1">待配送</text>
							<text v-if="item.order_status==1 && item.delivery_type==2">待自取</text>
							<text v-if="item.order_status==2">配送中</text>
							<text v-if="item.order_status==3">已完成</text>
							<text v-if="item.order_status==4 && !item.paypal_type">已过期</text>
							<text v-if="item.order_status==4 && item.paypal_type">已关闭</text>
						</view>
					</view>
					<view class="goodsBox lineBottom">
						<view class="goods" v-for="(itm, idx) in item.goods_list" :key="idx">
							<view class="left">
								<view class="image"><image lazy-load mode="aspectFill" :src="itm.goods_pic" /></view>
							</view>
							<view class="right">
								<view class="title" :style="'color:'+template.color.color2">{{itm.goods_title}}</view>
								<view class="label" :style="'color:'+template.color.color3">
									<text v-for="(value, name) in itm.goods_spec_name" :key="name">{{name}}:{{value}}</text>
								</view>
								<view class="box">
									<view class="price" :style="'color:'+template.color.color1">¥{{itm.goods_price}}</view>
									<view class="number" :style="'color:'+template.color.color3">x{{itm.buy_num}}</view>
								</view>
							</view>
						</view>
					</view>
					<view class="orderBox lineBottom">
						<view class="left">
							<view :style="'color:'+template.color.color3">下单时间：{{item.create_time}}</view>
							<view :style="'color:'+template.color.color3">订单编号：{{item.order_number}}</view>
						</view>
						<view class="right">
							<view :style="'color:'+template.color.color2">¥{{item.total_money}}</view>
						</view>
					</view>
					<view class="oFooter">
						<view class="right">
							<view v-if="item.order_status==0" class="button" @click.stop="paypalPrderTap(index)" :style="'background-color:'+template.color.color1">
								支付 <countdown v-if="item.close_time_stamp" @callback="getOrderList()" :targetTime="item.close_time_stamp"></countdown>
							</view>
							<view v-if="item.order_status==1 && item.delivery_type!=2" class="button" @click.stop="contact()" :style="'background-color:'+template.color.color4">联系客服<button open-type="contact"></button></view>
							<view v-if="item.order_status==1 && item.delivery_type==2" class="button" :style="'background-color:'+template.color.color1" @click.stop="qrcodeTap(true, item.verification_code)">出示提货码</view>
							<view v-if="item.order_status==2" class="button" :style="'background-color:'+template.color.color1" @click.stop="confirmOrder(item.id)">确认收货</view>
							<view v-if="item.order_status==4" class="button" :style="'background-color:'+template.color.color5" @click.stop="invalidOrder(item.id)">删除订单</view>
						</view>
					</view>
				</view>
			</view>
			<view v-else class="noneGoodsBox">
				<view class="none" :style="'color:'+template.color.color6">暂无相关订单</view>
			</view>
			<view style="height: 100rpx;"></view>
			
			<view class="showContent" :style="paypalShow?'bottom:0;':''">
				<view class="bg" @click="paypalPrderTap()"></view>
				<view class="showBox">
					<view class="pheader" :style="'color:'+template.color.color2">请选择支付方式</view>
					<view class="payTime" :style="'color:'+template.color.color3">
						剩余支付时间：<countdown v-if="orderData.close_time_stamp" :targetTime="orderData.close_time_stamp"></countdown>
					</view>
					<view class="paypalList">
						<view class="paypal lineBottom" v-for="(item, index) in shopData.shop_paypal" :key="index" @click="getPaypalOrder(item.lid, item.pdata)">
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
								<view v-if="item.lid==5" class="info" :style="'color:'+template.color.color6">可用积分：{{userData.score}}（{{item.pdata}}积分=1元）</view>
							</view>
							<view class="icon">
								<icon type="right" size="18" color="#e2e2e2"></icon>
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
					<view class="qrText" :style="'background-color:'+template.color.color4" @click="copyVerificationCode()">{{qrcodeText}}</view>
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
		components: {Icon, Countdown},
		data() {
			return {
				loading: true,
				template: uni.getStorageSync('__yipinTemplateConfig'),
				shopData: uni.getStorageSync('__yipinShopInfo'),
				page: 1,
				orderList: [],
				paypalShow: false,
				orderData: '',
				imageQrPath: '',
				qrcodeShow: false,
				qrcodeText: '',
				userData: {
					moneys: 0,
					score: 0
				}
			}
		},
		onShow() {
			this.getUserDetails();
			this.getOrderList();
		},
		onLoad() {
			let self = this;
		},
		methods: {
			getUserDetails() {
				let self = this;
				self.Get(self.Url.userDetails, {}).then(res => {
					if(res.code === 0){
						self.userData = res.data;
					}
				});
			},
			getOrderList() {
				let self = this;
				let postData = {
					order_type: 1,
					page: self.page,
					sizes: 50,
					status: -1
				}
				self.Get(self.Url.orderList, postData).then(res => {
					if(res.code === 0){
						self.loading = false;
						self.orderList = res.data.list;
					}
				})
			},
			paypalPrderTap(index) {
				if(index!=null){
					this.orderData = this.orderList[index];
					this.paypalShow = true;
				}else{
					this.paypalShow = false;
					this.orderData = '';
				}
			},
			getPaypalOrder(id, number) {
				let self = this;
				// 1 微信支付
				// 2 货到付款
				// 3 余额支付
				// 5 积分支付
				let postData = {
					pay_type: id,
					order_id: self.orderData.id,
					platform: 1,
				}
				if(id==3 && self.userData.moneys < self.orderData.total_money){
					uni.showModal({
						title: '余额支付提示',
						content: '您的可用余额不足以支付该订单，请更换其他支付方式',
						showCancel: false
					})
					return
				}
				if(id === 5 && (self.userData.score / number) < self.orderData.total_money ){
					uni.showModal({
						title: '积分支付提示',
						content: '您的可用积分不足以支付该订单，请更换其他支付方式',
						showCancel: false
					})
					return
				}
				self.Get(self.Url.orderPaypal, postData).then(res => {
					if(res.code === 0){
						if(id === 1){
							uni.requestPayment({
							    provider: 'wxpay',
							    timeStamp: res.data.wechat.timeStamp,
							    nonceStr: res.data.wechat.nonceStr,
							    package: res.data.wechat.package,
							    signType: res.data.wechat.signType,
							    paySign: res.data.wechat.paySign,
							    success: function (res) {
									uni.showToast({title: '微信支付成功'});
									self.paypalShow = false;
									self.getOrderList();
							    },
							    fail: function (err) {
							        uni.showModal({
							        	title: '支付失败',
										content: '订单支付失败，请重新进行支付。如针对订单金额有疑问请联系客服人员',
										showCancel: false
							        })
									return
							    }
							});
						}else if(id === 2){
							uni.showModal({
								title: '货到付款提示',
								content: '收货前请检查快递包装有无破损，如果需要拒收请先联系客服人员',
								showCancel: false
							})
							self.paypalShow = false;
							self.getOrderList();
							return
						}else if(id === 3){
							uni.showToast({title: '余额支付成功'});
							self.paypalShow = false;
							self.getOrderList();
						}else if(id === 5){
							uni.showToast({title: '积分支付成功'});
							self.paypalShow = false;
							self.getOrderList();
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
			qrcodeTap(status, code) {
				let self = this;
				if(status){
					self.qrcodeText = code;
					QRCode.api.draw(code,'canvas',200,200);
					setTimeout(() => { self.canvasToQrTempImage();},300);
				}else{
					self.qrcodeShow = false;
				}
			},
			canvasToQrTempImage:function(){
			    let self = this;
			    wx.canvasToTempFilePath({
			        canvasId: 'canvas',
			        success: function (res) {
						self.imageQrPath = res.tempFilePath;
						self.qrcodeShow = true;
			        }
			    });
			},
			copyVerificationCode() {
				let text = this.qrcodeText;
				uni.setClipboardData({
				    data: text,
				    success: function () {}
				});
			},
			confirmOrder(id) {
				let self = this;
				self.Get(self.Url.orderConfirm, {id: id}).then(res => {
					if(res.code === 0){
						uni.showToast({title: '确认收货成功'});
						self.getOrderList();
					}
				});
			},
			invalidOrder(id) {
				let self = this;
				uni.showModal({
					title: '订单删除确认',
					content: '删除订单后不可恢复，是否删除该订单？',
					success(ses) {
						if(ses.confirm){
							self.Get(self.Url.orderInvalid, {id: id}).then(res => {
								if(res.code === 0){
									uni.showToast({title: '订单删除成功'});
									self.getOrderList();
								}
							});
						}
					}
				})
			},
			getOrderDetail(id) {
				uni.navigateTo({url: '/pages/order/detail?id='+id});
			},
			contact() {
				
			}
		}
	}
</script>

<style lang="less">
	page{
		background-color: #f5f5f5;
	}
	.orderListBox{
		padding-top: 10rpx;
		.list{
			margin-bottom: 10rpx;
			background-color: #fff;
			.oHeader{
				display: flex;
				align-items: center;
				position: relative;
				width: 92%;
				margin: 0 auto;
				height: 70rpx;
				font-size: 28rpx;
				.left{
					flex: 0 0 50%;
				}
				.right{
					flex: 0 0 50%;
					display: flex;
					justify-content: flex-end;
				}
			}
			.goodsBox{
				width: 92%;
				margin: 0 auto;
				position: relative;
				.goods{
					display: flex;
					align-items: center;
					padding: 10rpx 0;
					.left{
						flex: 0 0 16%;
						.image{
							width: 100rpx;
							height: 100rpx;
							background-color: #f5f5f5;
							border-radius: 6rpx;
							overflow: hidden;
						}
					}
					.right{
						flex: 0 0 84%;
						.label{
							font-size: 26rpx;
						}
						.box{
							display: flex;
							align-items: center;
							padding-top: 6rpx;
							.price{
								flex: 0 0 50%;
							}
							.number{
								flex: 0 0 50%;
								display: flex;
								justify-content: flex-end;
							}
						}
					}
				}
			}
			.orderBox{
				width: 92%;
				margin: 0 auto;
				font-size: 24rpx;
				position: relative;
				padding: 10rpx 0;
				display: flex;
				align-items: center;
				.left{
					flex: 0 0 80%;
				}
				.right{
					flex: 0 0 20%;
					display: flex;
					justify-content: flex-end;
					font-size: 30rpx;
				}
			}
			.oFooter{
				padding: 0 30rpx;
				min-height: 70rpx;
				display: flex;
				align-items: center;
				position: relative;
				.right{
					display: flex;
					flex: 0 0 100%;
					justify-content: flex-end;
					.button{
						padding: 6rpx 10rpx;
						color: #fff;
						border-radius: 4rpx;
						font-size: 26rpx;
						position: relative;
						countdown{
							padding-left: 6rpx;
						}
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
			}
		}
	}
	.noneGoodsBox{
	    width:100%;
	    height:700rpx;
	    display:flex;
	    align-items:center;
	    justify-content:center;
	    .none{
	        font-size:30rpx;
	        color:#c5c9ca;
	        display:flex;
	        align-items:center;
	        justify-content:center;
	        position: relative;
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
