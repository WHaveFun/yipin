<template>
	<view class="content">
		<view class="shopInfo">
			<view class="left">
				<view class="title" :style="'color:'+template.color.color2">{{shopData.shop_name}}</view>
				<view v-if="distance" class="address" :style="'color:'+template.color.color3">距离您 {{(distance/1000)>1?distance/1000+'千米':distance+'米'}}，请确认下单门店是否正确</view>
				<view v-else class="address" :style="'color:'+template.color.color3">{{shopData.shop_address}}</view>
			</view>
			<view class="right">
				<image src="../../static/images/icon/map.png"></image>
			</view>
		</view>
		<view class="delivery">
			<view class="selectType">
				<view class="label" @click="deliveryType(2)" style="margin-right: 20rpx;" :style="deliveryNum===2?'border-color:'+template.color.color1+';background-color:'+template.color.color1+';color:#fff;':'color:'+template.color.color2">
					<icon type="mendian" size="20" :color="deliveryNum===2?'#fff':template.color.color2"></icon>
					<text>门店自提</text>
				</view>
				<view class="label" @click="deliveryType(1)" :style="deliveryNum===1?'border-color:'+template.color.color1+';background-color:'+template.color.color1+';color:#fff;':'color:'+template.color.color2">
					<icon type="peisong" size="20" :color="deliveryNum===1?'#fff':template.color.color2"></icon>
					<text>送货上门</text>
				</view>
			</view>
			<view v-if="deliveryNum==1" class="address">
				<view class="ptitle" :style="'color:'+template.color.color2">配送至</view>
				<view class="info" @click="selectAddressTap()">
					<view class="left">
						<block v-if="addressData">
							<view class="name" :style="'color:'+template.color.color2">{{addressData.cname}}<text v-if="addressData.phone"> {{addressData.phone}}</text></view>
							<view class="adds" :style="'color:'+template.color.color3">{{addressData.province}}{{addressData.address}}</view>
						</block>
						<block v-else>
							<view class="name" :style="'color:'+template.color.color3">未设置收货地址，请先设置</view>
						</block>
					</view>
					<view class="right">
						<icon type="left" size="18" :color="template.color.color3"></icon>
					</view>
				</view>
			</view>
			<view v-if="deliveryNum==2" class="address">
				<view class="ptitle" :style="'color:'+template.color.color3">联系电话</view>
				<view class="input">
					<input type="number" v-model="userPhone" placeholder="请输入联系电话" :style="'color:'+template.color.color2"/>
				</view>
				<view class="button" :style="'color:'+template.color.color1+';border-color:'+template.color.color1+'50'">
					自动填写<button open-type="getPhoneNumber" @getphonenumber="getphonenumberTap"></button>
				</view>
			</view>
		</view>
		
		<view class="cartList">
			<view class="goodsBox">
				<view class="gheader" :style="'color:'+template.color.color2">商品列表</view>
				<view class="list">
					<view class="goods lineBottom" v-for="(item, index) in shopCartList" :key="index">
						<view class="left">
							<view class="image">
								<image :src="item.pic" mode="aspectFit" lazy-load></image>
							</view>
						</view>
						<view class="right">
							<view class="name" :style="'color:'+template.color.color2">{{item.title}}</view>
							<view class="label" :style="'color:'+template.color.color5">
								<text v-for="(value, name) in item.spec_name" :key="name">{{name}}:{{value}}</text>
							</view>
							<view class="box">
								<view class="price" :style="'color:'+template.color.color1">¥<text>{{item.mini_price}}</text></view>
								<view class="number" :style="'color:'+template.color.color5">x{{item.num}}</view>
							</view>
						</view>
					</view>
				</view>
			</view>
		</view>
		
		<view class="couponsList" v-if="couponsList.length>1">
			<view class="couponsBox">
				<view class="cheader" :style="'color:'+template.color.color2">使用优惠</view>
				<!-- <view class="codeBox lineBottom">
					<view class="left" :style="'color:'+template.color.color3">兑换优惠</view>
					<view class="right">
						<input type="text" placeholder="请输入优惠码(如有)" @confirm="codeCouponsTap" :style="'color:'+template.color.color2"/>
					</view>
				</view> -->
				<view v-if="couponsList.length>1" class="listBox lineBottom">
					<view class="left" :style="'color:'+template.color.color3">选择优惠</view>
					<view class="right">
						<picker @change="selectCoupons" :value="couponsIndex" range-key="title" :range="couponsList">
							<view class="name" :style="'color:'+template.color.color3" v-if="couponsIndex==0">共{{couponsList.length-1}}张优惠券可用</view>
							<view class="name" :style="'color:'+template.color.color2" v-else>{{couponsList[couponsIndex].title}}<text :style="'color:'+template.color.color5">(满¥{{couponsList[couponsIndex].min_money}}减¥{{couponsList[couponsIndex].moneys}})</text></view>
						</picker>
					</view>
				</view>
			</view>
		</view>
		
		<view class="orderBox">
			<view class="oheader" :style="'color:'+template.color.color2">订单信息</view>
			<view class="remark lineBottom">
				<view class="left" :style="'color:'+template.color.color2">订单备注</view>
				<view class="right">
					<input type="text" v-model="remarkText" placeholder="如有需要请备注说明" style="text-align: right;" :style="'color:'+template.color.color2"/>
				</view>
			</view>
			<view class="order">
				<view class="label lineBottom">
					<view class="title" :style="'color:'+template.color.color2">商品合计<text :style="'color:'+template.color.color3">(共{{orderData.goods_number}}件商品)</text></view>
					<view class="number" :style="'color:'+template.color.color2">¥{{orderData.goods_money}}</view>
				</view>
				<view v-if="couponsIndex>0 && orderData.coupons_money>0" class="label lineBottom">
					<view class="title" :style="'color:'+template.color.color2">优惠金额<text :style="'color:'+template.color.color5">(满¥{{couponsList[couponsIndex].min_money}}减¥{{couponsList[couponsIndex].moneys}})</text></view>
					<view class="number" :style="'color:'+template.color.color2">-¥{{orderData.coupons_money}}</view>
				</view>
				<view v-if="orderData.user_level" class="label lineBottom">
					<view class="title" :style="'color:'+template.color.color2">会员折扣<text :style="'color:'+template.color.color5">({{orderData.user_level}}{{orderData.user_count*10}}折)</text></view>
					<view class="number" :style="'color:'+template.color.color2">-¥{{orderData.user_money}}</view>
				</view>
				<view v-if="orderData.freight_money>0" class="label lineBottom">
					<view class="title" :style="'color:'+template.color.color2">配送费</view>
					<view class="number" :style="'color:'+template.color.color2">¥{{orderData.freight_money}}</view>
				</view>
				<view class="label">
					<view class="title" :style="'color:'+template.color.color2">合计</view>
					<view class="number" :style="'color:'+template.color.color1">¥{{orderData.total_money}}</view>
				</view>
			</view>
		</view>
		
		<view :style="'height:'+(IphoneX>=0?200:220)+'rpx;'"></view>
		<view class="footer" :class="IphoneX>=0?'footerX':''">
			<view class="left">
				<view class="total" :style="'color:'+template.color.color1">合计：¥{{orderData.total_money}}</view>
			</view>
			<view class="right">
				<block v-if="orderActive">
					<view v-if="orderData.order_id" class="button" :style="'background-color:'+template.color.color1" @click="orderPaypalTap()">继续支付</view>
					<block v-else>
						<view v-if="deliveryNum===1 && orderData.goods_money>=template.overall.delivery_money" class="button" :style="'background-color:'+template.color.color1" @click="paypalOrderTap('form')">提交订单</view>
						<view v-if="deliveryNum===1 && orderData.goods_money<template.overall.delivery_money" class="button none" :style="'background-color:'+template.color.color5">{{template.overall.delivery_money}}元起送</view>
						<view v-if="deliveryNum===2" class="button" :style="'background-color:'+template.color.color1" @click="paypalOrderTap('form')">提交订单</view>
					</block>
				</block>
				<block v-else>
					<view class="button" :style="'background-color:'+template.color.color5">提交订单</view>
				</block>
				
				
			</view>
		</view>
		
		<view class="showContent" :style="paypalShow?'bottom:0;':''">
			<view class="bg" @click="closePaypalShow()"></view>
			<view class="showBox">
				<view class="pheader" :style="'color:'+template.color.color2">请选择支付方式</view>
				<view class="payTime" :style="'color:'+template.color.color3">
					剩余支付时间：<countdown v-if="orderData.order_time_stamp" :targetTime="orderData.order_time_stamp"></countdown>
				</view>
				<view class="paypalList">
					<view class="paypal lineBottom" v-for="(item, index) in shopData.shop_paypal" v-if="item.lid!=5" :key="index" @click="getPaypalOrder(item.lid)">
						<view class="left">
							<icon v-if="item.lid==1" type="weixinzhifu" size="30" color="#00c250"></icon>
							<icon v-if="item.lid==2" type="huodaofukuan" size="30" color="#ff9900"></icon>
							<icon v-if="item.lid==3" type="yuezhifu" size="30" :color="template.color.color1"></icon>
						</view>
						<view class="right">
							<view class="title" :style="'color:'+template.color.color2">{{item.cname}}</view>
							<view v-if="item.lid==1" class="info" :style="'color:'+template.color.color5">简单快捷，推荐使用</view>
							<view v-if="item.lid==2" class="info" :style="'color:'+template.color.color5">购物零风险，更安全</view>
							<view v-if="item.lid==3" class="info" :style="'color:'+template.color.color5">可用余额：¥{{userData.moneys}}</view>
						</view>
						<view class="icon">
							<icon type="left" size="18" color="#e2e2e2"></icon>
						</view>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import Icon from "@/components/icon/icon.vue"
	import Countdown from "@/components/countdown/countdown.vue"
	import QQMapWX from '@/static/utils/qqmap-wx-jssdk.min.js'
	export default {
		components: {Icon, Countdown},
		data() {
			return {
				paypalShow: false,
				IphoneX: this.IphoneX,
				template: uni.getStorageSync('__yipinTemplateConfig'),
				shopData: uni.getStorageSync('__yipinShopInfo'),
				deliveryNum: uni.getStorageSync('__yipinTemplateConfig').overall.delivery_type===1?2:1,
				addressData: '',
				userPhone: '',
				remarkText: '',
				shopCartList: [],
				couponsIndex: 0,
				couponsList: [],
				goodsListId: '',
				goodsOrderList: '',
				orderData: {
					goods_number: 0,
					goods_money: 0,
					total_money: 0
				},
				goodsTotal: 0,
				userData: {
					moneys: 0,
					score: 0
				},
				distance: 0,
				orderActive: false,
				wxCode: ''
			}
		},
		onShow() {
			let self = this;
			self.getAddressList();
			self.getUserDetails();
		},
		onLoad() {
			let self = this;
			let lids = [];
			let list = [];
			let nums = 0;
			let cart = uni.getStorageSync('__yipinShopCart');
			if(cart){
				cart.forEach(res => {
					nums += res.mini_price * res.num;
					lids.push(res.id);
					list.push({
						id: res.id,
						spec: res.spec_id,
						num: res.num
					});
				});
				self.shopCartList = cart;
				self.goodsTotal = nums;
				self.goodsListId = lids.join(',');
				self.goodsOrderList = list;
				self.getCouponsUserList();
			}
			self.getShopDistance();
			self.getWechatLogin();
		},
		methods: {
			getShopDistance() {
				let self = this;
				let temp = self.template;
				if(temp.overall.qqmap_key && temp.overall.latitude && temp.overall.longitude){
					uni.getLocation({
						type:'wgs84',
						success:function(gps){
							self.mapCTX = new QQMapWX({key: temp.overall.qqmap_key});
							self.mapCTX.calculateDistance({
								from: {
									latitude: temp.overall.latitude,
									longitude: temp.overall.longitude
								},
								to: [{
									location: {
										lat: gps.latitude,
										lng: gps.longitude
									}
								}],
								success: (ctx) => {
									self.distance = ctx.result.elements[0].distance;
								}
							});
						}
					})
				}
			},
			getUserDetails() {
				let self = this;
				self.Get(self.Url.userDetails, {}).then(res => {
					if(res.code === 0){
						self.userData = res.data;
						if(res.data.phone){
							self.userPhone = res.data.phone;
						}
					}
				});
			},
			getCouponsUserList() {
				let self = this;
				self.Get(self.Url.couponsUserList, {status: 1, goods: self.goodsListId}).then(res => {
					if(res.code === 0){
						let coupons = [{id: 0, title: '不使用优惠券'}];
						let data = res.data;
						data.forEach(itm => {
							if(self.goodsTotal >= itm.min_money){
								coupons.push(itm);
							}
						});
						self.couponsList = coupons;
					}
				})
			},
			getAddressList() {
				let self = this;
				self.Get(self.Url.addressList, {default: true}).then(res => {
					if(res.code === 0){
						if(res.data.length>0){
							self.addressData = res.data[0];
							self.addressData.province = res.data[0].province.replace(/,/g,'');
						}
					}else{
						self.addressData = '';
					}
					self.paypalOrderTap();
				})
			},
			deliveryType(id) {
				this.deliveryNum = id;
				this.paypalOrderTap();
			},
			selectAddressTap() {
				uni.navigateTo({url: '/pages/user/adslist'});
			},
			getPhoneTap(number) {
				uni.makePhoneCall({
				    phoneNumber: number
				});
			},
			selectCoupons(e) {
				this.couponsIndex = e.detail.value;
				this.paypalOrderTap();
			},
			paypalOrderTap(e){
				let self = this;
				let temp = self.template;
				self.orderActive = true;
				uni.showLoading({title: '数据加载中',mask: true});
				if(self.deliveryNum==1 && !self.addressData){
					uni.showModal({
						title: '订单提示',
						content: '还没有收货地址，请先选择或添加收货地址！',
						confirmText: '选择地址',
						cancelText: '门店自提',
						success(res) {
							if(res.confirm){
								uni.navigateTo({url: '/pages/user/adslist'});
							}else if(res.cancel){
								self.deliveryNum = 2;
								self.paypalOrderTap();
							}
						}
					})
					uni.hideLoading();
					self.orderActive = false;
					return
				}
				if(self.deliveryNum==1 && self.addressData){
					if(temp.overall.qqmap_key && temp.overall.delivery_distance && temp.overall.latitude && temp.overall.longitude){
						self.mapCTX = new QQMapWX({key: temp.overall.qqmap_key});
						self.mapCTX.geocoder({
							address: self.addressData.province,
							success: function(gps) {
								self.mapCTX.calculateDistance({
									from: {
										latitude: temp.overall.latitude,
										longitude: temp.overall.longitude
									},
									to: [{
										location: {
											lat: gps.result.location.lat,
											lng: gps.result.location.lng,
										}
									}],
									success: (ctx) => {
										let distance = ctx.result.elements[0].distance;
										if(distance > temp.overall.delivery_distance){
											uni.showModal({
												title: '配送提示',
												content: '配送范围仅限店铺'+(temp.overall.delivery_distance/1000)+'千米内，您的地址已经超出配送范围！',
												showCancel: false
											})
											uni.hideLoading();
											self.orderActive = false;
											return
										}
									}
								});
							}
						})
					}
				}
				if(e && self.deliveryNum>1 && !self.userPhone){
					uni.showModal({
						title: '订单提示',
						content: '需要填写联系电话，以便紧急情况下与您取得联系！',
						showCancel: false,
						success(res) {
							if(res.confirm){
								self.orderActive = true;
							}
						}
					})
					uni.hideLoading();
					self.orderActive = false;
					return
				}
				let postData = {
					goods: self.goodsOrderList,
					delivery_type: self.deliveryNum,
					remark: self.remarkText,
					yprint: true
				}
				if(!e){
					postData.calculate = true;
				}
				if(self.deliveryNum==1){
					postData.address_id = self.addressData.id;
				}else{
					postData.user_phone = self.userPhone;
				}
				if(self.couponsIndex>0){
					postData.coupons_id = self.couponsList[self.couponsIndex].id;
				}
				self.Post(self.Url.orderCreate, postData).then(res => {
					uni.hideLoading();
					if(res.code === 0){
						self.orderData = res.data;
						if(e){
							self.getsubscribeMessage(res.data);
						}
					}else{
						uni.showModal({
							title: '订单提示',
							content: res.msg,
							showCancel: false
						});
						return
					}
				});
			},
			getsubscribeMessage(data) {
				let self = this;
				let push = self.template.push;
				let delivery = self.deliveryNum;
				let template = {};
				let postData = {
					order_id: data.order_id,
					page: '/pages/tabbar?order='+data.order_id
				};
				if(delivery==1){
					template.tmplIds = push.wechat2;
				}else if(delivery==2){
					template.tmplIds = push.wechat1;
				}else if(delivery==3){
					template.tmplIds = push.wechat3;
				}
				console.log('准备推送')
				if(template.tmplIds){
					wx.requestSubscribeMessage({
						tmplIds: [template.tmplIds],
						complete(res){ 
							console.log('Message',res)
							if(res.errMsg == 'requestSubscribeMessage:ok'){
								if(delivery==1){
									//快递物流
									postData.order_status = 2;
									postData.template_id = push.wechat2;
									postData.data = {
										character_string1: {
											value: '{order_number}'
										},
										phrase6: {
											value: '{logistics_name}'
										},
										character_string7: {
											value: '{logistics_number}'
										},
										date2: {
											value: '{logistics_time}'
										},
										thing8: {
											value: '您的商品已发货，请注意查收！'
										}
									}
								}else if(delivery==2){
									//到店自提
									postData.order_status = 1;
									postData.template_id = push.wechat1;
									postData.data = {
										amount3: {
											value: '¥{total_money}'
										},
										character_string1: {
											value: '{order_number}'
										},
										thing7: {
											value: self.shopData.shop_name
										},
										thing4: {
											value: self.shopData.shop_address
										},
										thing8: {
											value: '您的订单已经准备好，请尽快到该门店自提！'
										}
									}
								}else if(delivery==3){
									//无需配送
									postData.order_status = 3;
									postData.template_id = push.wechat3;
									postData.data = {
										amount5: {
											value: '¥{total_money}'
										},
										number1: {
											value: '{order_number}'
										},
										date2: {
											value: '{examine_time}'
										},
										thing4: {
											value: '您的订单已经确认完成，如有疑问请联系客服！'
										}
									}
								}
								console.log('postData',postData)
								self.Post(self.Url.messageWechat, postData).then(mes => {
									console.log('messageWechat',mes)
								});
								self.paypalShow = true;
							}else{
								self.paypalShow = true;
							}
						}
					})
				}else{
					console.log('没有推送')
					self.paypalShow = true;
				}
			},
			getPaypalOrder(id) {
				let self = this;
				// 1 微信支付
				// 2 货到付款
				// 3 余额支付
				let postData = {
					pay_type: id,
					order_id: self.orderData.order_id,
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
									setTimeout(function () {
										uni.reLaunch({url: '/pages/tabbar?order=1'});
									}, 600);
							    },
							    fail: function (err) {
							        uni.showModal({
							        	title: '支付失败',
										content: '订单支付失败，请重新进行支付。如对订单金额有疑问请联系客服人员',
										showCancel: false
							        })
									return
							    }
							});
						}else if(id === 2){
							uni.showModal({
								title: '货到付款提示',
								content: '收货前请检查快递包装有无破损，如果需要拒收请先联系客服人员',
								showCancel: false,
								success(ces) {
									if(ces.confirm){
										uni.reLaunch({url: '/pages/tabbar?order=1'});
									}
								}
							})
							return
						}else if(id === 3){
							uni.showToast({title: '余额支付成功'});
							setTimeout(function () {
								uni.reLaunch({url: '/pages/tabbar?order=1'});
							}, 600);
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
			orderPaypalTap() {
				this.paypalShow = true;
			},
			closePaypalShow() {
				this.paypalShow = false;
			},
			getWechatLogin() {
				let self = this;
				uni.login({
					provider: 'weixin',
					success: function(wxs) {
						self.wxCode = wxs.code;
					}
				})
			},
			getphonenumberTap(e) {
				let self = this;
				let data = e.detail;
				if (!data.errMsg || data.errMsg!="getPhoneNumber:ok") {
				  uni.showModal({
				    content: '无法获取您的手机号码，请重试！',
				    showCancel: false
				  })
				}else{
					self.Post(self.Url.userBindWechat, {
						iv: data.iv,
						code: self.wxCode,
						encryptedData: data.encryptedData
					}).then(res => {
						if(res.code === 0){
							uni.showToast({title: '获取成功'});
							self.getUserDetails();
						}else{
							self.getWechatLogin();
							uni.showModal({
								content: '手机号码获取失败，请重试！',
								showCancel: false
							})
						}
					});
				}
			}
		}
	}
</script>

<style lang="less">
	.shopInfo{
		width: 92%;
		margin: 0 auto;
		display: flex;
		align-items: center;
		justify-content: center;
		.left{
			width: 80%;
			.title {
				width: 95%;
				color: #343434;
				font-size: 30rpx;
				font-weight: bold;
				text-overflow: ellipsis;
				white-space: nowrap;
				overflow: hidden;
			}
			.address {
				width: 95%;
				color: #999;
				font-size: 26rpx;
				padding-top: 6rpx;
				text-overflow: ellipsis;
				white-space: nowrap;
				overflow: hidden;
			}
		}
		.right{
			width: 20%;
			display: flex;
			justify-content: flex-end;
			image{
				width: 60rpx;
				height: 60rpx;
				border-radius: 100%;
			}
		}
	}
	.delivery{
		width: 92%;
		margin: 0 auto;
		padding-top: 20rpx;
		.selectType{
			position: relative;
			display: flex;
			align-items: center;
			justify-content: center;
			padding: 10rpx 0;
			.label{
				flex: 0 0 48%;
				height: 80rpx;
				border: 1px solid #eee;
				border-radius: 4rpx;
				display: flex;
				align-items: center;
				justify-content: center;
				text{
					padding-left: 20rpx;
				}
			}
		}
		.address{
			display: flex;
			align-items: center;
			padding: 20rpx 0;
			.ptitle{
				flex: 0 0 20%;
				font-size: 26rpx;
			}
			.input{
				flex: 0 0 60%;
				input{
					width: 100%;
					text-align: right;
					font-size: 26rpx;
				}
			}
			.button{
				flex: 0 0 17%;
				margin-left: 3%;
				border: 1px solid;
				font-size: 24rpx;
				display: flex;
				align-items: center;
				justify-content: center;
				padding: 4rpx 0;
				position: relative;
				button{
					position: absolute;
					width: 100%;
					height: 100%;
					top: 0;
					left: 0;
					opacity: 0;
				}
			}
			.info{
				flex: 0 0 80%;
				display: flex;
				align-items: center;
				.left{
					width: 90%;
					.name{
						width: 95%;
						font-weight: bold;
						text-overflow: ellipsis;
						white-space: nowrap;
						overflow: hidden;
					}
					.adds{
						width: 95%;
						font-size: 26rpx;
						padding-top: 6rpx;
						text-overflow: ellipsis;
						white-space: nowrap;
						overflow: hidden;
					}
				}
				.right{
					width: 10%;
					display: flex;
					justify-content: flex-end;
				}
			}
		}
	}
	.couponsList{
		border-top: 10rpx solid #f5f5f5;
		.couponsBox{
			width: 92%;
			margin: 0 auto;
			.cheader{
				flex: 0 0 20%;
				font-size: 26rpx;
				font-weight: bold;
				padding: 20rpx 0 0;
			}
			.codeBox{
				position: relative;
				display: flex;
				align-items: center;
				padding: 20rpx 0;
				.left{
					flex: 0 0 21%;
				}
				.right{
					flex: 0 0 79%;
					input{
						width: 100%;
					}
				}
			}
			.listBox{
				position: relative;
				display: flex;
				align-items: center;
				padding: 20rpx 0;
				.left{
					flex: 0 0 21%;
				}
				.right{
					flex: 0 0 79%;
					picker {
						width: 100%;
					}
				}
			}
		}
	}
	
	.orderBox{
		border-top: 10rpx solid #f5f5f5;
		padding-top: 10rpx;
		.oheader{
			width: 92%;
			margin: 0 auto;
			font-weight: bold;
			font-size: 26rpx;
		}
		.remark{
			width: 92%;
			margin: 0 auto;
			position: relative;
			display: flex;
			align-items: center;
			height: 90rpx;
			.left{
				flex: 0 0 21%;
			}
			.right{
				flex: 0 0 79%;
				input{
					width: 100%;
				}
			}
		}
		.order{
			width: 92%;
			margin: 0 auto;
			.label{
				display: flex;
				align-items: center;
				height: 90rpx;
				position: relative;
				.title{
					flex: 0 0 60%;
					font-size: 28rpx;
					text{
						font-size: 24rpx;
						padding-left: 10rpx;
					}
				}
				.number{
					flex: 0 0 40%;
					display: flex;
					justify-content: flex-end;
					font-size: 30rpx;
				}
			}
		}
	}
	.cartList{
		border-top: 10rpx solid #f5f5f5;
		.goodsBox{
			width: 92%;
			margin: 0 auto;
			padding: 10rpx 0 0;
			.gheader{
				padding: 10rpx 0 0;
				font-weight: bold;
				font-size: 26rpx;
			}
			.list{
				.goods{
					display: flex;
					align-items: center;
					position: relative;
					padding: 20rpx 0;
					.left{
						flex: 0 0 20%;
						display: flex;
						align-items: center;
						.image{
							width: 110rpx;
							height: 110rpx;
							background-color: #f5f5f5;
							//border: 1px solid #f5f5f5;
							border-radius: 6rpx;
							overflow: hidden;
							image{
								width: 100%;
								height: 100%;
							}
						}
					}
					.right{
						flex: 0 0 80%;
						.name{
							font-size: 28rpx;
							width: 90%;
							white-space: nowrap;
							overflow: hidden;
							text-overflow: ellipsis;
						}
						.label{
							font-size: 26rpx;
						}
						.box{
							display: flex;
							align-items: center;
							.price{
								font-size: 30rpx;
								flex: 0 0 50%;
							}
							.number{
								font-size: 28rpx;
								flex: 0 0 50%;
								display: flex;
								justify-content: flex-end;
							}
						}
					}
				}
			}
		}
	}
	.footer{
		display: flex;
		width: 100%;
		background-color: #fff;
		position: fixed;
		bottom: 0;
		height: 100rpx;
		z-index: 999;
		box-shadow: 0 10rpx 20rpx rgba(0,0,0,.2);
		.left{
			flex: 0 0 50%;
			height: 100rpx;
			display: flex;
			align-items: center;
			.total{
				padding-left: 30rpx;
				font-size: 30rpx;
			}
		}
		.right{
			flex: 0 0 50%;
			height: 100rpx;
			display: flex;
			align-items: center;
			justify-content: center;
			.button{
				color: #fff;
				font-size: 30rpx;
				width: 82%;
				height: 70rpx;
				border-radius: 100rpx;
				display: flex;
				align-items: center;
				justify-content: center;
			}
		}
	}
	.footerX{
		height: 130rpx;
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
				font-weight: 300;
				padding: 20rpx 0 10rpx;
			}
			.payTime{
				text-align: center;
				padding: 20rpx 0 0;
				font-weight: 300;
			}
			.paypalList{
				padding-top: 20rpx;
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
						font-weight: 300;
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
