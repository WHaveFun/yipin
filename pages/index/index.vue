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
			<view class="header">
				<view class="left">
					<view class="title" :style="'color:'+template.color.color2">{{shopInfo.shop_name}}</view>
					<view v-if="distance" class="address" :style="'color:'+template.color.color3">距离您 {{(distance/1000)>1?distance/1000+'千米':distance+'米'}}</view>
					<view v-else class="address" :style="'color:'+template.color.color3">{{shopInfo.shop_address}}</view>
				</view>
				<view class="right">
					<view class="type" :style="'background-color:'+template.color.color1">
						<view @click="selectTypeTap(1)" :class="template.overall.delivery_type===1?'name':'text'" :style="template.overall.delivery_type===1?'background-color:'+template.color.color2:''">自提</view>
						<view @click="selectTypeTap(2)" :class="template.overall.delivery_type===2?'name':'text'" :style="template.overall.delivery_type===2?'background-color:'+template.color.color2:''">外卖</view>
					</view>
					<view class="more" @click="infoShow=!infoShow" :style="'color:'+template.color.color3">
						{{infoShow?'收起门店信息':'更多门店信息'}}
						<icon :type="infoShow?'up':'down'" size="15" color="#999"></icon>
					</view>
				</view>
			</view>
			<view style="height: 130rpx;"></view>
			<view class="goodsBox" :style="'height:'+goodsHeight+'px;'">
				<view class="left">
					<scroll-view scroll-y scroll-with-animation :style="'height:'+goodsHeight+'px;'" class="menuList">
						<view class="menu" :class="menuCurrent==index?'active':''" :style="menuCurrent==index?'color:'+template.color.color2+';border-color:'+template.color.color1:'color:'+template.color.color3"
						 @click="selectMenuTap(index)" v-for="(item, index) in goodsAllList" :key="index">{{item.title}}</view>
						 <view style="height: 200rpx;"></view>
					</scroll-view>
				</view>
				<view class="right">
					<scroll-view scroll-y scroll-with-animation :scroll-into-view="goodsInto" @scroll="goodsScroll" :style="'height:'+goodsHeight+'px;'">
						<view v-if="swiperList.length>0" class="swiperBpx">
							<swiper class="swiper" circular autoplay interval="6000" duration="500" @change="swiperChange">
								<swiper-item v-for="(item, index) in swiperList" :key="index" @click="getBannerTap(item.target_data)">
									<image lazy-load="true" mode="aspectFill" :src="item.pic" />
								</swiper-item>
							</swiper>
							<view class="swiperDots">
								<block v-for="(item, index) in swiperList" :key="index">
									<view class="dot" :class="swiperCurrent==index?'active':''"></view>
								</block>
							</view>
						</view>
						<view id="goodsQuery" v-for="(item, index) in goodsAllList" :key="index">
							<view class="goodsList" :id="'gory-'+index">
								<view class="gheader" :style="'color:'+template.color.color2">{{item.title}}</view>
								<view class="goods lineBottom" v-for="(itm, idx) in item.goods" :key="idx" @click="getDteailsTap(index, idx)">
									<view class="left">
										<view class="image">
											<image :src="itm.pic" lazy-load mode="aspectFill" />
											<icon v-if="itm.recom_status" class="icon" type="hot" size="30" color="#d3aa79"></icon>
											<view v-if="itm.stock_number==0" class="none" :style="'color:'+template.color.color5">已售罄</view>
										</view>
									</view>
									<view class="right">
										<view class="title" :style="'color:'+template.color.color2">{{itm.title}}</view>
										<view class="intro" :style="'color:'+template.color.color3">{{itm.introduce}}</view>
										<view class="box">
											<view class="price" :style="'color:'+template.color.color1">¥{{itm.mini_price}}</view>
											<view class="button">
												<block v-if="openStoreStatus">
													<block v-if="itm.specs_status">
														<view v-if="itm.stock_number" class="text" :style="'background-color:'+template.color.color1">选规格</view>
														<view v-else class="text" :style="'background-color:'+template.color.color5">选规格</view>
													</block>
													<block v-else>
														<icon v-if="itm.stock_number" @click.stop="addCartGoods(itm.id, 0, 1, 'add')" type="jia" size="23" :color="template.color.color1"></icon>
														<icon v-else type="jia" size="23" :color="template.color.color5"></icon>
													</block>
												</block>
												<block v-else>
													<view v-if="itm.specs_status" class="text" :style="'background-color:'+template.color.color5">选规格</view>
													<icon v-else type="jia" size="23" :color="template.color.color5"></icon>
												</block>
											</view>
										</view>
									</view>
								</view>
							</view>
						</view>
						<view style="height: 150rpx;"></view>
					</scroll-view>
				</view>
			</view>
			<view v-if="openStoreStatus && cartList.length>0" class="footer" :class="IphoneX>=0?'footerX':''">
				<view class="left">
					<view class="cart" @click="cartShow=!cartShow" :style="'color:'+template.color.color2">
						<icon type="liwu" size="18" color="#343434"></icon>购物袋<text :style="'background-color:'+template.color.color1">{{cartNumber}}</text>
					</view>
					<view class="price" :style="'color:'+template.color.color2">¥{{cartTotal}}</view>
				</view>
				<view class="right">
					<block v-if="template.overall.delivery_type===1">
						<view v-if="cartNumber>0" class="button" @click="toPaypalTap()" :style="'background-color:'+template.color.color1">去结算</view>
						<view v-else class="button none" :style="'background-color:'+template.color.color5">去结算</view>
					</block>
					<block v-if="template.overall.delivery_type===2">
						<view v-if="cartTotal>=template.overall.delivery_money" class="button" @click="toPaypalTap()" :style="'background-color:'+template.color.color1">去结算</view>
						<view v-else class="button none" :style="'background-color:'+template.color.color5">{{template.overall.delivery_money}}元起送</view>
					</block>
				</view>
			</view>
			<view v-if="!openStoreStatus" class="footer" :class="IphoneX>=0?'footerX':''" style="overflow: hidden;">
				<view class="label" :style="'background-color:'+template.color.color2">本店已休息，营业时间：{{openStoreTime}}</view>
			</view>
			
			<view v-if="goodShow" class="showPopBox" style="display: flex;align-items: center;justify-content: center;">
				<view class="bg" @click="goodShow=false"></view>
				<view class="goodsConten">
					<view class="image">
						<image :src="goodsOneData.pic" lazy-load mode="aspectFit"></image>
						<icon @click="goodShow=false" type="close" size="25" :color="template.color.color5" style="top: 10rpx; right: 10rpx;"></icon>
						<icon @click="shareShow=true" type="fenxiang" size="25" :color="template.color.color1" style="top: 10rpx; left: 10rpx;"></icon>
						<icon @click="favGoodsTap(goodsOneData.id)" type="shoucang" size="25" :color="template.color.color1" style="top: 10rpx; left: 70rpx;"></icon>
					</view>
					<view class="title" :style="'color:'+template.color.color2">{{goodsOneData.title}}</view>
					<scroll-view scroll-y class="moreBox" style="max-height: 500rpx;">
						<view class="label" v-for="(item, index) in goodsOneData.specs_list.specification" :key="index">
							<view class="stitle"><text>{{item.name}}</text></view>
							<view class="specBox">
								<view class="spec" :style="itm.active?'background-color:'+template.color.color1+';border-color:'+template.color.color1:''" :class="itm.active?'active':''" @click="selectSpecsTap(index, idx)" v-for="(itm, idx) in item.value" :key="idx">{{itm.name}}</view>
							</view>
						</view>
						<view class="more">
							<view class="labtit" :style="'color:'+template.color.color3">产品介绍</view>
							<view class="intro" :style="'color:'+template.color.color3">{{goodsOneData.introduce}}</view>
						</view>
					</scroll-view>
					<view class="gfooter">
						<view class="price">
							<view class="left">
								<view v-if="childProductData.child_product_price" class="text" :style="'color:'+template.color.color1">¥{{childProductData.child_product_price}}<text :style="'color:'+template.color.color3" v-if="childProductData.child_product_cost">¥{{childProductData.child_product_cost}}</text></view>
								<view v-else class="text" :style="'color:'+template.color.color1">¥{{goodsOneData.mini_price}}<text :style="'color:'+template.color.color3" v-if="goodsOneData.max_price">¥{{goodsOneData.max_price}}</text></view>
								<view class="label" v-if="childProductData.child_product_spec" :style="'color:'+template.color.color3">
									<text v-for="(value, name) in childProductData.child_product_spec" :key="name">{{name}}:{{value}} </text>
								</view>
							</view>
							<view class="right">
								<view class="buy">
									<icon type="jian" @click="editGoodsOne('jian')" size="23" :color="buyNumber<=1?template.color.color5:template.color.color1"></icon>
									<input type="number" :value="buyNumber" disabled :style="'color:'+template.color.color2" />
									<icon type="jia" @click="editGoodsOne('jia')" size="23" :color="buyNumber>=goodsOneData.stock_number?template.color.color5:template.color.color1"></icon>
								</view>
							</view>
						</view>
						<block v-if="openStoreStatus">
							<block v-if="buyActive">
								<view v-if="goodsOneData.specs_status && !childProductData" class="button none" :style="'background-color:'+template.color.color5">加入购物袋</view>
								<view v-if="goodsOneData.specs_status && childProductData" class="button" @click="addShopCar()" :style="'background-color:'+template.color.color1">加入购物袋</view>
								<view v-if="!goodsOneData.specs_status" class="button" @click="addShopCar()" :style="'background-color:'+template.color.color1">加入购物袋</view>
							</block>
							<view v-else class="button none" :style="'background-color:'+template.color.color5">{{goodsOneData.stock_number===0?'商品已售罄':'加入购物袋'}}</view>
						</block>
						<block v-else>
							<view class="button none" :style="'background-color:'+template.color.color5">本店已休息</view>
						</block>
					</view>
				</view>
			</view>
			
			<view v-if="cartShow && cartList.length>0" class="showPopBox" style="z-index: 898;">
				<view class="bg" @click="cartShow=false"></view>
				<view class="showContent" :style="IphoneX>=0?'bottom:146rpx;':''">
					<view class="sheade">
						<view class="left">
							<view class="text">共{{cartNumber}}件商品</view>
						</view>
						<view class="right">
							<view class="text" @click="delNoneCartTap()"><icon type="shanchu" size="14"></icon> 清空购物袋</view>
						</view>
					</view>
					<view class="sconten" :style="IphoneX>=0?'padding-bottom:146rpx;':'padding-bottom: 120rpx;'">
						<view class="goods" v-for="(item, index) in cartList" :key="index" :style="cartList.length==(index+1)?'border-bottom: none;':''">
							<view class="left">
								<view class="title">{{item.title}}</view>
								<view class="label" v-if="item.spec_name">
									<text v-for="(value, name) in item.spec_name" :key="name">{{name}}:{{value}}</text>
								</view>
							</view>
							<view class="right">
								<view class="price">¥{{item.mini_price}}</view>
								<view class="buy">
									<block v-if="item.shelve_status">
										<block v-if="item.stock_number">
											<icon type="jian" @click="editGoods(item, 'jian')" size="23" :color="item.num==1?'#cccccc':'#d3aa79'"></icon>
											<input type="number" :value="item.num" disabled />
											<icon type="jia" @click="editGoods(item, 'jia')" size="23" :color="item.num>=item.stock_number?'#cccccc':'#d3aa79'"></icon>
										</block>
										<block v-else>
											<text>商品已售罄</text>
										</block>
									</block>
									<block v-else>
										<text>商品已下架</text>
									</block>
								</view>
							</view>
						</view>
					</view>
				</view>
			</view>
			
			<view v-if="infoShow" class="showPopBox" style="z-index: 899;">
				<view class="bg" @click="infoShow=false"></view>
				<view class="infoContent">
					<view v-if="template.overall.discount_text" class="label">
						<view class="title" :style="'color:'+template.color.color2">优惠信息</view>
						<view class="text" :style="'color:'+template.color.color3">
							<text>{{template.overall.discount_text}}</text>
						</view>
					</view>
					<view v-if="template.overall.delivery_text" class="label">
						<view class="title" :style="'color:'+template.color.color2">配送信息</view>
						<view class="text" :style="'color:'+template.color.color3">
							<text>{{template.overall.delivery_text}}</text>
						</view>
					</view>
					<view v-if="template.overall.store_text" class="label">
						<view class="title" :style="'color:'+template.color.color2">门店信息</view>
						<view class="text" :style="'color:'+template.color.color3">
							<text>{{template.overall.store_text}}</text>
						</view>
					</view>
					<view v-if="template.overall.licence_list.length>0" class="label">
						<view class="title" :style="'color:'+template.color.color2">门店展示</view>
						<view class="images">
							<view class="image" v-for="(item, index) in template.overall.licence_list" :key="index" @click="previewImage(index)">
								<image mode="aspectFill" :src="item.pics" lazy-load></image>
								<view v-if="item.title" class="text" :style="'color:'+template.color.color3">{{item.title}}</view>
							</view>
						</view>
					</view>
				</view>
			</view>
			
			<view v-if="shareShow" class="showShare">
				<view class="bg" @click="[haibaoStu=false,shareShow=false]"></view>
				<view class="share">
					<view class="title" :style="'color:'+template.color.color2">{{haibaoStu?'预览分享海报':'选择分享方式'}}</view>
					<view v-if="haibaoStu" class="haibao">
						<image @click="previewHaiImage()" :src="haibaoImg" mode="aspectFill"></image>
						<view class="save" @click="saveImageToPhotosAlbum()" :style="'background-color:'+template.color.color1">保存海报到本地</view>
					</view>
					<view v-else class="box">
						<view class="label">
							<icon type="weixin" size="42" color="#07c160"></icon>
							<view class="text" :style="'color:'+template.color.color3">微信好友</view>
							<button open-type="share" :data-goods="goodsOneData"></button>
						</view>
						<view class="label" @click="getGoodsHaibao()">
							<icon type="haibao" size="42" :color="template.color.color1"></icon>
							<view class="text" :style="'color:'+template.color.color3">生成海报</view>
						</view>
					</view>
				</view>
			</view>
			
			<view v-if="couponsList.length>0 && !couponsShow" @click="couponsShow=true" class="couponsIcon" :style="cartList.length>0?'bottom: 250rpx;':'bottom: 160rpx;'">
				<image v-if="template.overall.coupons_icon" mode="aspectFill" lazy-load :src="template.overall.coupons_icon"></image>
				<image v-else mode="aspectFill" lazy-load src="../../static/images/icon/bonus.png"></image>
			</view>
			<view v-if="couponsShow" class="showCoupons">
				<view class="bg" @click="couponsShow=false"></view>
				<view class="coupons">
					<view class="headerTitle">赠送您{{couponsList.length}}张优惠券</view>
					<scroll-view scroll-y class="listBox" style="height:246rpx;">
					  <view class="list" v-for="(item, index) in couponsList" :key="index">
						<view class="left">
						  <view class="price" :style="'color:'+template.color.color1"><text>¥</text>{{item.moneys}}</view>
						  <view class="text" :style="'color:'+template.color.color3">满{{item.min_money}}元可用</view>
						</view>
						<view class="active">
						  <view class="top"></view>
						  <view class="buttom"></view>
						</view>
						<view class="right">
						  <view class="title" :style="'color:'+template.color.color2">{{item.title}}</view>
						  <view class="times" v-if="item.time_type===1" :style="'color:'+template.color.color3">{{item.end_time}}日截止</view>
						  <view class="times" v-else :style="'color:'+template.color.color3">领取后{{item.time_day}}天内有效</view>
						</view>
					  </view>
					</scroll-view>
					<view class="buttonBox">
					  <view class="button" @click="getNewCouponsTap()">立即领取</view>
					</view>
					<image src="../../static/images/icon/vip_bg.png"></image>
					<icon type="guanbi" color="#fff" size="35" class="close" @click="couponsShow=false"></icon>
				</view>
			</view>
		</block>
	</view>
</template>

<script>
	import Icon from "@/components/icon/icon.vue";
	import QQMapWX from '@/static/utils/qqmap-wx-jssdk.min.js'
	const app = getApp();
	export default {
		components: {
			Icon
		},
		props: {
			goods: {
				type: Number,
				default: 0,
			},
		},
		data() {
			return {
				loading: true,
				IphoneX: this.IphoneX,
				template: uni.getStorageSync('__yipinTemplateConfig'),
				shopInfo: uni.getStorageSync('__yipinShopInfo'),
				userData: uni.getStorageSync('__yipinUserInfo'),
				goodsHeight: this.IphoneX >= 0 ? this.WindowHeight - 138 : this.WindowHeight - 125,
				menuCurrent: 0,
				menuList: [],
				swiperCurrent: 0,
				swiperList: [],
				page: 1,
				goodsListData: {
					page: 1,
					total: 0,
					list: []
				},
				cartTotal: 0,
				cartNumber: 0,
				cartList: [],
				goodsOneData: '',
				buyNumber: 1,
				buyNumMax: 0,
				canSubmit: false,
				childProductData: '',
				buyActive: false,
				cartShow: false,
				infoShow: false,
				goodShow: false,
				goodsInto: '',
				goodsAllList: [],
				heightArr: [],
				distance: 0,
				haibaoStu: false,
				haibaoImg: '',
				shareShow: false,
				openStoreStatus: true,
				openStoreTime: '',
				couponsList: [],
				couponsShow: false
			}
		},
		onReady() {
			let self = this;
			let temp = self.template.overall.open_time;
			if(temp){
				let news = temp.split(',');
				let date = `${new Date().getHours()}:${new Date().getMinutes()<10?'0'+new Date().getMinutes():new Date().getMinutes()}`;
				self.openStoreTime = `${news[0]}~${news[1]}`;
				if(self.CompareDate(date, news[0]) && self.CompareDate(news[1], date)){
					self.openStoreStatus = true;
				}else{
					self.openStoreStatus = false;
				}
			}
			self.getGoodsAllList();
			self.getBannerList();
			self.getCartList();
			self.getShopDistance();
			self.getCouponsList();
			if(self.goods){
				self.Get(self.Url.goodsDetail, {id: self.goods}).then(res => {
					if(res.code === 0){
						self.goodsOneData = res.data;
						self.goodsOneData.pic = res.data.pic_list[0].pic;
						if(res.data.stock_number){
							self.buyActive = true;
							self.buyNumber = 1;
							self.buyNumMax = res.data.stock_number;
						}else{
							self.buyActive = false;
							self.buyNumber = 0;
							self.buyNumMax = 0;
						}
						self.goodShow = true;
						self.$emit('gooods', 0);
						//self.goods = 0;
					}
				})
			}
		},
		methods: {
			CompareDate(t1,t2){
				let date = new Date();
				let a = t1.split(":");
				let b = t2.split(":");
				return date.setHours(a[0],a[1]) > date.setHours(b[0],b[1]);
			},
			getBannerList() {
				let self = this;
				self.Get(self.Url.bannerList, {ctype: 'home'}).then(res => {
					if(res.code === 0){
						self.swiperList = res.data;
					}
				})
			},
			getCouponsList() {
				let self = this;
				if(self.openStoreStatus && self.template.overall.coupons_list){
					self.Post(self.Url.couponsGoods, {coupons_list: self.template.overall.coupons_list}).then(res => {
						if(res.code === 0){
							let list = [];
							res.data.forEach(lis => {
								if(lis.user_status){
									list.push(lis)
								}
							})
							self.couponsList = list;
							if(list.length>0){
								self.couponsShow = true;
							}
						}
					})
				}
			},
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
			getCartList() {
				let self = this;
				self.Get(self.Url.cartList, {}).then(res => {
					if(res.code === 0){
						let data = res.data;
						let totals = 0;
						let number = 0;
						data.forEach(itm => {
							if(itm.shelve_status && itm.stock_number>0){
								number += itm.num;
								totals += itm.mini_price * itm.num;
							}
						})
						self.cartList = data;
						self.cartTotal = totals.toFixed(2);
						self.cartNumber = number;
					}
				})
			},
			getGoodsAllList() {
				let self = this;
				self.Get(self.Url.goodsAllList, {}).then(res => {
					if(res.code === 0){
						self.loading = false;
						self.goodsAllList = res.data;
						setTimeout(function (){
						  self.goodsQuery();
						}, 500)
					}
				})
			},
			goodsQuery() {
			  let self = this;
			  let heightArr = [];
			  const query = uni.createSelectorQuery().in(self);
			  query.selectAll('#goodsQuery').boundingClientRect()
			  query.exec(function(res){
			    let react = res[0];
			    for(let i = 0; i < react.length; i++){
			      heightArr.push(react[i].bottom-65)
			    }
				self.heightArr = heightArr;
			  });
			},
			goodsScroll(e) {
				let self = this;
				let scrollTop = e.detail.scrollTop;
				let scrollArr = self.heightArr;
				for(let i = 0; i < scrollArr.length; i++){
					if(scrollTop <= scrollArr[i]){
						self.menuCurrent = i;
						return
					}
				}
			},
			selectTypeTap(type) {
				let self = this;
				let data = self.template;
				if(app.globalData.user){
					data.overall.delivery_type = type;
					uni.setStorage({key: '__yipinTemplateConfig',data: data});
				}else{
					uni.navigateTo({url: '/pages/user/login'});
				}
				
				
			},
			selectMenuTap(index) {
				this.menuCurrent = index;
				this.goodsInto = 'gory-'+index;
			},
			swiperChange(e) {
				this.swiperCurrent = e.detail.current;
			},
			getBannerTap(id) {
				let self = this;
				if(app.globalData.user){
					self.childProductData = '';
					self.Get(self.Url.goodsDetail, {id: id}).then(res => {
						if(res.code === 0){
							self.goodsOneData = res.data;
							self.goodsOneData.pic = res.data.pic_list[0].pic;
							if(res.data.stock_number){
								self.buyActive = true;
								self.buyNumber = 1;
								self.buyNumMax = res.data.stock_number;
							}else{
								self.buyActive = false;
								self.buyNumber = 0;
								self.buyNumMax = 0;
							}
							self.goodShow = true;
						}
					})
				}else{
					uni.navigateTo({
						url: '/pages/user/login'
					})
				}
			},
			getDteailsTap(index, idx) {
				let self = this;
				let data = self.goodsAllList[index].goods[idx];
				if(app.globalData.user){
					self.childProductData = '';
					if(data.stock_number){
						self.buyActive = true;
						self.buyNumber = 1;
						self.buyNumMax = data.stock_number;
					}else{
						self.buyActive = false;
						self.buyNumber = 0;
						self.buyNumMax = 0;
					}
					if(data.specs_status){
						self.Get(self.Url.goodsDetail, {id: data.id}).then(res => {
							if(res.code === 0){
								self.goodsOneData = res.data;
								self.goodsOneData.pic = res.data.pic_list[0].pic;
								if(res.data.stock_number){
									self.buyActive = true;
									self.buyNumber = 1;
									self.buyNumMax = res.data.stock_number;
								}else{
									self.buyActive = false;
									self.buyNumber = 0;
									self.buyNumMax = 0;
								}
								self.goodShow = true;
							}
						})
					}else{
						self.goodsOneData = data;
						self.goodShow = true;
					}
				}else{
					uni.navigateTo({
						url: '/pages/user/login'
					})
				}
			},
			selectSpecsTap(index, idx) {
				let self = this;
				let specification = self.goodsOneData.specs_list.specification;
				for(let i = 0; i < specification.length; i++){
					if(i == index) {
						specification[i].active = true;
						for(let s = 0; s < specification[i].value.length; s++){
							if(s == idx) {
								specification[i].value[s].active = true;
							}else{
								specification[i].value[s].active = false
							}
						}
					}
				}
				self.$set(self.goodsOneData.specs_list.specification, index, self.goodsOneData.specs_list.specification[index])
				let curSelectNum = 0;
				let propertyChildIds = '';
				let propertyChildNames = '';
				let selectSpecification = {};
				for(let i = 0; i < specification.length; i++){
					if(specification[i].active){
						curSelectNum++
						for(let s = 0; s < specification[i].value.length; s++){
							if(specification[i].value[s].active){
								selectSpecification[specification[i].name] = specification[i].value[s].name;
							}
						}
					}
				}
				if(curSelectNum==specification.length){
					self.canSubmit = false;
				}else{
					self.canSubmit = true;
				}
				let childProductArray = self.goodsOneData.specs_list.childProductArray;
				for(let i = 0; i < childProductArray.length; i++){
					if(!self.canSubmit && Object.keys(selectSpecification).every(key => childProductArray[i].child_product_spec[key] === selectSpecification[key])){
						self.buyActive = true;
						self.childProductData = childProductArray[i];
						self.buyNumber = 1;
						self.buyNumMax = childProductArray[i].child_product_stock;
						if(childProductArray[i].child_product_stock==0){
							self.buyActive = false;
							self.buyNumber = 0;
							uni.showToast({
								title: '该商品库存不足',
								icon: 'none'
							})
						}
					}
				}
			},
			addShopCar() {
				let self = this;
				let data = self.goodsOneData;
				if(data.specs_status){
					let child = self.childProductData;
					if(child.child_product_id){
						self.addCartGoods(data.id, child.child_product_id, self.buyNumber, 'add');
					}else{
						uni.showToast({title: '请选择规格', icon: 'none'});
						return
					}
				}else{
					self.addCartGoods(data.id, 0, self.buyNumber, 'add');
				}
			},
			addCartGoods(id, spec, number, type){
				let self = this;
				if(app.globalData.user){
					let postData = {
						id: id,
						number: number
					}
					let url = {};
					if(spec){
						postData.spec = spec;
					}
					if(type=='add'){
						url.link = self.Url.cartAdd;
					}else if(type=='edit'){
						url.link = self.Url.cartEdit;
					}
					self.Get(url.link, postData).then(res => {
						if(res.code === 0){
							self.goodShow = false;
							//uni.showToast({title: '加入成功'});
							self.getCartList();
						}else{
							uni.showModal({
								title: '购物袋提示',
								content: res.msg,
								showCancel: false
							})
							return
						}
					})
				}else{
					uni.navigateTo({
						url: '/pages/user/login'
					})
				}
			},
			delNoneCartTap() {
				let self = this;
				let data =self.cartList;
				let list = [];
				data.forEach(res => {
					list.push(res)
				})
				if(list.length>0){
					uni.showModal({
						title: '清空购物袋',
						content: '是否清空已经加入购物袋的商品？',
						success(res) {
							if(res.confirm){
								self.cartShow = false;
								list.forEach(itm => {
									let postData = {
										id: itm.id,
										number: 0
									}
									if(itm.spec_id){
										postData.spec = itm.spec_id;
									}
									self.Get(self.Url.cartEdit, postData).then(res => {
										if(res.code === 0){
											self.getCartList();
										}
									})
								})
							}
						}
					})
				}
			},
			editGoodsOne(type) {
				let self = this;
				let data = self.goodsOneData;
				if(type == 'jian'){
					if(self.buyNumber>1){
						self.buyNumber--
					}
				}else if(type == 'jia'){
					if(self.buyNumber<data.stock_number){
						self.buyNumber++
					}else{
						uni.showToast({title: '库存不足', icon: 'none'});
						return
					}
				}
			},
			editGoods(data, type){
				let self = this;
				if(type=='jian'){
					self.addCartGoods(data.id, data.spec_id, data.num-1, 'edit');
				}else if(type=='jia'){
					if(data.num == data.stock_number){
						uni.showModal({
							title: '库存不足',
							content: '购买数量已经超过最大库存数量',
							showCancel: false
						})
						return
					}else{
						self.addCartGoods(data.id, data.spec_id, data.num+1, 'edit');
					}
				}
			},
			previewImage(index) {
				let data = this.template.overall.licence_list;
				let list = [];
				data.forEach(res => {
					list.push(res.pics);
				})
				uni.previewImage({
					urls: list,
					current: list[index]
				});
			},
			favGoodsTap(id){
				let self = this;
				if(app.globalData.user){
					self.Get(self.Url.favoriteAdd, {id: id, type: true}).then(res => {
						if(res.code == 0){
							uni.showToast({title: '收藏成功'});
						}else{
							self.Get(self.Url.favoriteAdd, {id: id, type: false}).then(res => {
								if(res.code == 0){
									uni.showToast({title: '取消收藏', icon: 'none'});
								}
							})
						}
					});
				}else{
					uni.navigateTo({url: '/pages/user/login'});
				}
			},
			toPaypalTap() {
				let self = this;
				let data = self.cartList;
				if(app.globalData.user){
					uni.setStorage({key: '__yipinShopCart',data: data});
					uni.navigateTo({url: '/pages/order/paypal'});
				}else{
					uni.navigateTo({
						url: '/pages/user/login'
					})
				}
			},
			getNewCouponsTap() {
				let self = this;
				if(app.globalData.user){
					let data = self.couponsList;
					data.forEach((lis, idx) => {
						self.Get(self.Url.couponsUserAdd, {id: lis.id}).then(res => {
							if(res.code === 0){
								uni.showToast({title: '领取成功'});
								self.couponsShow = false;
								if(idx===(data.length-1)){
									self.couponsShow = false;
									self.couponsList = [];
								}
							}else{
								uni.showModal({
									content: res.msg,
									showCancel: false
								})
							}
						})
					})
				}else{
					uni.navigateTo({
						url: '/pages/user/login'
					})
				}
			},
			getGoodsHaibao() {
				let self = this;
				let data = self.goodsOneData;
				let pics = data.pic.replace(/http:/g,'https:');
				// if(self.haibaoImg){
				// 	self.haibaoStu = true;
				// 	return
				// }
				let postData = {
					scene: 'g='+data.id,
					page: 'pages/tabbar',
					dtime: 10
				}
				uni.showLoading({title: '海拔绘制中'});
				self.Post(self.Url.qrcodeWechat, postData).then(res => {
					if(res.code === 0){
						wx.downloadFile({
							url: res.url,
							success (qrcode) {
								uni.showLoading({title: '海报绘制中'});
								wx.downloadFile({
									url: pics,
									success (pic) {
										let sics = self.shopInfo.shop_logo.replace(/http:/g,'https:');
										wx.downloadFile({
											url: sics,
											success (sic) {
												let ctxs = wx.createCanvasContext('goodsHaibao');
												let postData = {
													pic: pic.tempFilePath,
													color: self.template.color,
													shop_logo: sic.tempFilePath,
													max_price: '¥'+data.max_price,
													mini_price: '¥'+data.mini_price,
													stock_number: '库存：'+data.stock_number,
													shop_name: self.shopInfo.shop_name,
													qrcode: qrcode.tempFilePath,
													nikename: self.userData.nikename
												};
												console.log(postData)
												//商品标题
												if(data.title.split('').length >= 16){
													postData.title = data.title.substr(0,16)+'...';
												}else{
													postData.title = data.title;
												}
												//商品介绍
												if(data.introduce.split('').length >= 18){
													postData.introduce = data.introduce.substr(0,18)+'...';
												}else{
													postData.introduce = data.introduce;
												}
												ctxs.setFillStyle('#FFFFFF')
												ctxs.fillRect(0, 0, 600, 1000)
												ctxs.setFillStyle(postData.color.color1)
												ctxs.setFontSize(30)
												ctxs.setTextAlign('left')
												ctxs.fillText(postData.shop_name, 105, 73)
												ctxs.save()
												ctxs.setFillStyle('#ffffff')
												self.roundRect(ctxs, 30, 30, 60, 60, 6)
												ctxs.drawImage(postData.shop_logo, 30, 30, 60, 60)
												ctxs.restore()
												ctxs.save()
												self.roundRect(ctxs, 30, 120, 540, 610, 10)
												ctxs.setFillStyle(postData.color.color5)
												ctxs.fillRect(30, 120, 540, 540)
												ctxs.drawImage(postData.pic, 30, 120, 540, 540)
												ctxs.setFillStyle(postData.color.color1)
												ctxs.fillRect(30, 650, 540, 80)
												ctxs.setFillStyle('#ffffff')
												ctxs.setFontSize(42)
												ctxs.setTextAlign('left')
												ctxs.fillText(postData.mini_price, 45, 705)
												const minrics = ctxs.measureText(postData.mini_price)
												ctxs.setFontSize(30)
												ctxs.setTextAlign('left')
												ctxs.fillText(postData.max_price, 55+minrics.width, 703)
												const maxrics = ctxs.measureText(postData.max_price)
												ctxs.beginPath()
												ctxs.setStrokeStyle('#FFFFFF')
												ctxs.setLineWidth(2)
												ctxs.moveTo(55+minrics.width, 695)
												ctxs.lineTo(55+minrics.width+maxrics.width, 695)
												ctxs.setFontSize(24)
												ctxs.setTextAlign('right')
												ctxs.fillText(postData.stock_number, 550, 700)
												ctxs.stroke()
												ctxs.restore()
												ctxs.setFillStyle(postData.color.color3)
												ctxs.setFontSize(32)
												ctxs.setTextAlign('left')
												ctxs.fillText(postData.title, 30, 775)
												ctxs.setFillStyle(postData.color.color5)
												ctxs.setFontSize(28)
												ctxs.setTextAlign('left')
												ctxs.fillText(postData.introduce, 30, 820)
												ctxs.drawImage(postData.qrcode, 430, 850, 120, 120)
												ctxs.setFillStyle(postData.color.color5)
												ctxs.setFontSize(22)
												ctxs.setTextAlign('right')
												ctxs.fillText('长按扫描二维码查看详情', 400, postData.nikename?940:920)
												ctxs.fillText('来自'+postData.nikename+'的推荐', 400, 910)
												ctxs.restore()
												ctxs.save()
												ctxs.draw()
												setTimeout(function () {
													uni.canvasToTempFilePath({
														x: 0,y: 0,width: 600,height: 1000,
														canvasId: 'goodsHaibao',
														success: function (res) {
															uni.hideLoading();
															self.haibaoImg = res.tempFilePath;
															self.haibaoStu = true;
														},
														fail: function(ces){
															console.log(ces)
															uni.hideLoading();
															uni.showToast({title: '海拔绘制失败', icon:'none'});
														}
													})
												}, 500);
											}
										})
									}
								});
							}
						})
					}else{
						uni.hideLoading();
						uni.showModal({
							title: '绘制失败',
							content: res.msg,
							showCancel: false
						})
					}
				});
			},
			roundRect(ctx, x, y, w, h, r) {
			  ctx.beginPath()
			  ctx.setFillStyle('transparent')
			  ctx.arc(x + r, y + r, r, Math.PI, Math.PI * 1.5)
			  ctx.moveTo(x + r, y)
			  ctx.lineTo(x + w - r, y)
			  ctx.lineTo(x + w, y + r)
			  ctx.arc(x + w - r, y + r, r, Math.PI * 1.5, Math.PI * 2)
			  ctx.lineTo(x + w, y + h - r)
			  ctx.lineTo(x + w - r, y + h)
			  ctx.arc(x + w - r, y + h - r, r, 0, Math.PI * 0.5)
			  ctx.lineTo(x + r, y + h)
			  ctx.lineTo(x, y + h - r)
			  ctx.arc(x + r, y + h - r, r, Math.PI * 0.5, Math.PI)
			  ctx.lineTo(x, y + r)
			  ctx.lineTo(x + r, y)
			  ctx.fill()
			  ctx.closePath()
			  ctx.clip()
			},
			previewHaiImage() {
				let self = this;
				uni.previewImage({
					urls: [self.haibaoImg],
					current: self.haibaoImg,
					success() {
						uni.saveImageToPhotosAlbum({
							filePath: self.haibaoImg,
							success: function () {}
						});
					}
				});
			},
			saveImageToPhotosAlbum() {
				let self = this;
				uni.saveImageToPhotosAlbum({
					filePath: self.haibaoImg,
					success: function () {
						uni.showToast({title: '海报保存成功'});
					}
				});
			},
		}
	}
</script>

<style lang="less">
	page {
		background-color: #f5f5f5;
		overflow: hidden;
	}

	.header {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 100%;
		height: 130rpx;
		position: fixed;
		top: 0;
		z-index: 900;
		background-color: #fff;
		box-shadow: 0 0 10rpx rgba(0, 0, 0, .05);

		.left {
			flex: 0 0 60%;
			display: flex;
			flex-direction: column;
			justify-content: center;

			.title {
				width: 90%;
				color: #343434;
				font-size: 30rpx;
				font-weight: bold;
				text-overflow: ellipsis;
				white-space: nowrap;
				overflow: hidden;
			}

			.address {
				width: 90%;
				color: #999;
				font-size: 26rpx;
				padding-top: 6rpx;
				text-overflow: ellipsis;
				white-space: nowrap;
				overflow: hidden;
			}
		}

		.right {
			flex: 0 0 35%;
			display: flex;
			flex-direction: column;
			align-items: flex-end;

			.type {
				display: flex;
				width: 180rpx;
				height: 60rpx;
				border-radius: 100rpx;
				align-items: center;
				justify-content: center;
				font-size: 26rpx;

				.name {
					background-color: #343434;
					color: #fff;
					width: 86rpx;
					border-radius: 100rpx;
					display: flex;
					align-items: center;
					justify-content: center;
					height: 54rpx;
					transition: all .3s;
				}

				.text {
					width: 86rpx;
					display: flex;
					align-items: center;
					justify-content: center;
					color: #fff;
				}
			}

			.more {
				color: #999;
				font-size: 24rpx;
				display: flex;
				align-items: center;
				padding-top: 6rpx;

				icon {
					padding-left: 4rpx;
				}
			}
		}
	}

	.goodsBox {
		width: 100%;
		display: flex;

		.left {
			flex: 0 0 28%;

			.menuList {
				overflow-y: scroll;
				padding-bottom: 20rpx;

				.menu {
					color: #999999;
					font-size: 26rpx;
					padding: 30rpx 10rpx;
					text-align: center;
					background-color: transparent;
					border-left: 6rpx solid transparent;
					transition: all 0.6s;
				}

				.active {
					font-weight: bold;
					background-color: #FFFFFF;
				}
			}
		}

		.right {
			flex: 0 0 72%;
			background-color: #FFFFFF;

			.swiperBpx {
				width: 95%;
				margin: 0 auto;
				padding-top: 14rpx;
				position: relative;

				swiper {
					height: 220rpx;

					.swiper-item {
						width: 100%;
						height: 100%;
						display: flex;
						align-items: center;
						justify-content: center;
						color: #FFFFFF;
						font-size: 56rpx;
					}
				}

				.swiperDots {
					position: absolute;
					left: 0;
					right: 0;
					bottom: 12rpx;
					display: flex;
					justify-content: center;
					height: 10rpx;

					.dot {
						margin: 0 5rpx;
						width: 25rpx;
						height: 4rpx;
						background: #fff;
						border-radius: 20rpx;
						transition: all 0.6s;
						opacity: 0.5;
					}

					.active {
						opacity: 1;
					}
				}
			}
			.goodsList{
				width: 95%;
				margin: 0 auto;
				.gheader{
					font-weight: bold;
					font-size: 26rpx;
					display: flex;
					height: 100rpx;
					align-items: center;
				}
				.goods{
					display: flex;
					position: relative;
					padding-bottom: 20rpx;
					margin-bottom: 20rpx;
					.left{
						flex: 0 0 30%;
						display: flex;
						.image{
							width: 140rpx;
							height: 140rpx;
							background-color: #f5f5f5;
							border-radius: 6rpx;
							overflow: hidden;
							position: relative;
							.icon{
								position: absolute;
								top: 0;
								left: 0;
							}
							.none{
								position: absolute;
								top: 0;
								left: 0;
								width: 100%;
								height: 100%;
								background-color: rgba(255,255,255,.6);
								display: flex;
								align-items: center;
								justify-content: center;
								font-size: 26rpx;
								color: #999;
							}
						}
					}
					.right{
						flex: 0 0 70%;
						.title{
							color:#343434;
							font-size:26rpx;
							font-weight:bold;
						}
						.intro{
							color:#999;
							font-size:24rpx;
							overflow: hidden;
							text-overflow: ellipsis;
							display: -webkit-box;
							-webkit-line-clamp: 2;
							-webkit-box-orient: vertical;
							padding-top: 4rpx;
						}
						.box{
							display:flex;
							align-items:center;
							padding-top: 10rpx;
							position: relative;
							.price{
								color:#343434;
								font-size:28rpx;
								font-weight:bold;
								flex: 0 0 50%
							}
							.button{
								flex: 0 0 50%;
								display: flex;
								justify-content: flex-end;
								.text{
									background-color: #d3aa79;
									color: #fff;
									font-size: 24rpx;
									padding: 4rpx 16rpx;
									border-radius: 100rpx;
								}
							}
						}
					}
					
				}
			}
		}
	}
	.footer{
		position:fixed;
		bottom: 140rpx;
		width:92%;
		left:0;
		right: 0;
		height:80rpx;
		margin: 0 auto;
		border-radius: 100rpx;
		background-color:#fff;
		display:flex;
		align-items:center;
		z-index:899;
		box-shadow: 0 0 20rpx rgba(0,0,0,.05);
		.label{
			color: #fff;
			width: 100%;
			height: 100%;
			display: flex;
			align-items: center;
			justify-content: center;
		}
		.left{
			flex: 0 0 70%;
			display: flex;
			align-items: center;
			justify-content: flex-end;
			height: 100%;
			.cart{
				position: absolute;
				width: 180rpx;
				height: 100%;
				left: 0;
				color: #343434;
				font-size: 26rpx;
				display: flex;
				align-items: center;
				justify-content: center;
				icon{
					padding-right: 8rpx;
				}
				text{
					position: absolute;
					top: -8rpx;
					right: -12rpx;
					background-color: #d3aa79;
					border-radius: 100%;
					width: 36rpx;
					height: 36rpx;
					display: flex;
					align-items: center;
					justify-content: center;
					color: #fff;
					font-size: 24rpx;
				}
			}
			.price{
				color:#343434;
				font-size:30rpx;
				padding-right: 30rpx;
				font-weight: bold;
			}
		}
		.right{
			flex: 0 0 30%;
			height: 100%;
			border-top-right-radius: 100rpx;
			border-bottom-right-radius: 100rpx;
			overflow: hidden;
			.button{
				width: 100%;
				height: 100%;
				color: #FFFFFF;
				display: flex;
				align-items: center;
				justify-content: center;
				background-color:#d3aa79;
			}
			.none{
				background-color:#bbbbbb;
			}
		}
	}
	.footerX{
		bottom: 166rpx;
	}
	
	.showPopBox{
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		background-color: rgba(0,0,0,.5);
		z-index: 970;
		transition: all 0.3s;
		.bg{
			position: absolute;
			top: 0;
			left: 0;
			width: 100%;
			height: 100%;
			z-index: 971;
		}
		.showContent{
			position: absolute;
			z-index: 972;
			width: 100%;
			max-height: 600rpx;
			bottom: 120rpx;
			background-color: #fff;
			left: 0;
			.sheade{
				width: 100%;
				background-color: #f2f2f2;
				font-size: 24rpx;
				height: 56rpx;
				display: flex;
				align-items: center;
				.left{
					flex: 0 0 50%;
					.text{
						color: #999;
						padding-left: 20rpx;
					}
				}
				.right{
					flex: 0 0 50%;
					.text{
						color: #999;
						display: flex;
						align-items: center;
						justify-content: flex-end;
						padding-right: 20rpx;
						icon{
							padding-right: 6rpx;
						}
					}
				}
			}
			.sconten{
				max-height: 544rpx;
				overflow-y: scroll;
				.goods{
					width: 95%;
					margin: 0 auto;
					display: flex;
					align-items: center;
					border-bottom: 1px dashed #f5f5f5;
					height: 110rpx;
					.left{
						flex: 0 0 60%;
						.title{
							color:#343434;
						}
						.label{
							color:#999;
							font-size: 24rpx;
						}
					}
					.right{
						flex: 0 0 40%;
						display: flex;
						align-items: center;
						justify-content: flex-end;
						.price{
							flex: 0 0 40%;
							color:#343434;
						}
						.buy{
							display: flex;
							align-items: center;
							justify-content: center;
							flex: 0 0 60%;
							input{
								text-align: center;
								color:#343434;
							}
							text{
								color: #999;
								font-size: 24rpx;
							}
						}
					}
				}
			}
		}
		.infoContent{
			position: absolute;
			z-index: 972;
			width: 100%;
			max-height: 55vh;
			top: 130rpx;
			background-color: #fff;
			left: 0;
			overflow-y: scroll;
			padding-bottom: 20rpx;
			padding-top: 10rpx;
			.label{
				width: 95%;
				margin: 0 auto;
				padding: 12rpx 0;
				.title{
					color: #343434;
					font-size: 26rpx;
				}
				.text{
					color: #999;
					font-size: 24rpx;
					padding-top: 4rpx;
				}
			}
			.images{
				display: flex;
				align-items: center;
				flex-wrap: wrap;
				padding-top: 6rpx;
				.image{
					width: 200rpx;
					height: 200rpx;
					border: 2rpx solid #f5f5f5;
					background-color: #f5f5f5;
					margin-right: 20rpx;
					position: relative;
					.text{
						position: absolute;
						bottom: 0;
						left: 0;
						width: 100%;
						background-color: rgba(255,255,255,.6);
						height: 50rpx;
						display: flex;
						align-items: center;
						justify-content: center;
						color: #666;
						font-size: 26rpx;
					}
				}
			}
		}
		.goodsConten{
			position: absolute;
			z-index: 972;
			width: 80%;
			background-color: #fff;
			border-radius: 10rpx;
			padding-bottom: 20rpx;
			margin-top: -5vh;
			box-shadow: 0 10rpx 20rpx rgba(0,0,0,.06);
			.image{
				width: 100%;
				height: 320rpx;
				display: flex;
				align-items: center;
				justify-content: center;
				position: relative;
				icon{
					position: absolute;
				}
			}
			.title{
				width: 90%;
				margin: 0 auto;
				padding-top: 10rpx;
				padding-bottom: 10rpx;
				color: #343434;
				font-weight: bold;
			}
			.moreBox{
				width: 90%;
				margin: 0 auto;
				.label{
					padding-bottom: 10rpx;
					.stitle{
						padding-bottom: 10rpx;
						text{
							color: #999;
							font-size: 26rpx;
						}
					}
					.specBox{
						display: flex;
						flex-wrap: wrap;
						.spec{
							border: 1px solid #eee;
							padding: 8rpx 20rpx;
							margin-right: 12rpx;
							font-size: 24rpx;
							color: #999999;
						}
						.active{
							color: #FFFFFF;
						}
					}
				}
				.more{
					padding-bottom: 20rpx;
					.labtit{
						color: #999;
						font-size: 26rpx;
						padding-top: 10rpx;
					}
					.intro{
						color: #999;
						font-size: 24rpx;
					}
				}
			}
			.gfooter{
				border-top: 1px solid #f5f5f5;
				padding-top: 20rpx;
				.price{
					width: 90%;
					margin: 0 auto;
					display: flex;
					padding: 10rpx 0;
					align-items: center;
					justify-content: center;
					.left{
						flex: 0 0 65%;
						.text{
							color: #d3aa79;
							font-size: 32rpx;
							font-weight: bold;
							display: flex;
							align-items: center;
							text{
								text-decoration: line-through;
								color: #999;
								font-size: 26rpx;
								margin-left: 10rpx;
								font-weight: normal;
							}
						}
						.label{
							color: #999;
							font-size: 24rpx;
						}
					}
					.right{
						flex: 0 0 35%;
						display: flex;
						justify-content: flex-end;
						.buy{
							display: flex;
							align-items: center;
							input{
								text-align: center;
								color:#343434;
							}
						}
					}
				}
				.button{
					width: 90%;
					margin: 0 auto;
					margin-top: 20rpx;
					background-color: #d3aa79;
					color: #fff;
					height: 66rpx;
					display: flex;
					align-items: center;
					justify-content: center;
					border-radius: 100rpx;
					font-size: 26rpx;
				}
				.none{
					background-color: #cccccc;
				}
			}
		}
	}
	.couponsIcon{
		position: fixed;
		bottom: 230rpx;
		width: 100rpx;
		height: 100rpx;
		left: 60rpx;
	}
	.showCoupons{
		position: fixed;
		width: 100%;
		height: 100%;
		top: 0;
		left: 0;
		background-color: rgba(0,0,0,.5);
		z-index: 999;
		display: flex;
		align-items: center;
		justify-content: center;
		.bg{
			position: absolute;
			width: 100%;
			height: 100%;
			top: 0;
			left: 0;
			z-index: 1000;
		}
		.coupons{
			position: absolute;
			width: 600rpx;
			height: 568rpx;
			border-radius: 10rpx;
			z-index: 1001;
			transition: all 0.6s;
			margin-top: -10%;
			.headerTitle{
				position: absolute;
				top: 76rpx;
				width: 100%;
				text-align: center;
				color: #af8859;
				font-size: 26rpx;
			}
			.listBox{
			  width: 72%;
			  margin: 0 auto;
			  margin-top: 50rpx;
			  position: absolute;
			  left: 0;
			  right: 0;
			  top: 100rpx;
			  height: 260rpx;
			  border-bottom-left-radius: 20%;
			  border-bottom-right-radius: 20%;
			  overflow: hidden;
			  .list{
			    display:flex;
			    align-items:center;
			    width:99%;
			    height:120rpx;
			    justify-content:center;
			    border-radius:4rpx;
			    position: relative;
			    overflow:hidden;
			    padding: 2rpx;
			    margin: 0 auto;
			    margin-bottom: 20rpx;
			    .left{
			      width:35%;
			      display:flex;
			      flex-direction:column;
			      align-items:center;
			      justify-content:center;
			      height:120rpx;
			      border:2rpx solid #f5f5f5;
			      border-right:none;
			      .price{
			        color:#d3aa79;
			        font-size:42rpx;
			        font-weight:500;
			        display:flex;
			        text{
			          font-size:26rpx;
			          padding-top:6rpx;
			          padding-right:2rpx;
			        }
			      }
			      .text{
			        color:#999;
			        font-size:22rpx;
			      }
			    }
			    .active{
			      position:absolute;
			      width:2rpx;
			      height:80rpx;
			      left:34%;
			      border-right:2rpx dashed #eee;
			      display:flex;
			      align-items:center;
			      justify-content:center;
			      .top{
			        width:20rpx;
			        height:20rpx;
			        background-color:#fff;
			        border-radius:100%;
			        position:absolute;
			        top:-30rpx;
			        border:2rpx solid #f5f5f5;
			      }
			      .buttom{
			        width:20rpx;
			        height:20rpx;
			        background-color:#fff;
			        border-radius:100%;
			        position:absolute;
			        bottom:-30rpx;
			        border:2rpx solid #f5f5f5;;
			      }
			    }
			    .right{
			      width:65%;
			      height:120rpx;
			      display:flex;
			      flex-direction:column;
			      justify-content:center;
			      border:2rpx solid #f5f5f5;
			      border-left:none;
			      .title{
			        color:#343434;
			        font-weight:500;
			        font-size:26rpx;
			        width:90%;
			        margin:0 auto;
			        overflow:hidden;
			        text-overflow:ellipsis;
			        white-space:nowrap;
			      }
			      .times{
			        color:#999;
			        font-size:24rpx;
			        padding-top:10rpx;
			        width:90%;
			        margin:0 auto;
			        overflow:hidden;
			        text-overflow:ellipsis;
			        white-space:nowrap;
			      }
			    }
			  }
			}
			.buttonBox{
			  width: 80%;
			  margin: 0 auto;
			  height: 70rpx;
			  display:flex;
			  align-items:flex-end;
			  justify-content:center;
			  position: absolute;
			  bottom: 36rpx;
			  left: 0;
			  right: 0;
			  .button{
			    background-color:#af8859;
			    width:100%;
			    height:100%;
			    display:flex;
			    align-items:center;
			    justify-content:center;
			    color:#fff;
			    border-radius:4rpx;
				font-size: 26rpx;
			  }
			}
			.close{
				position: absolute;
				left: 0;
				right: 0;
				margin: 0 auto;
				width: 100rpx;
				display: flex;
				align-items: center;
				justify-content: center;
				height: 100rpx;
			}
		}
	}
	.showShare{
		position: fixed;
		width: 100%;
		height: 100%;
		top: 0;
		left: 0;
		background-color: rgba(0,0,0,.5);
		z-index: 999;
		display: flex;
		align-items: center;
		justify-content: center;
		.bg{
			position: absolute;
			width: 100%;
			height: 100%;
			top: 0;
			left: 0;
			z-index: 1000;
		}
		.share{
			position: absolute;
			width: 500rpx;
			background-color: #FFFFFF;
			border-radius: 10rpx;
			z-index: 1001;
			transition: all 0.6s;
			.title{
				font-weight: bold;
				display: flex;
				align-items: center;
				justify-content: center;
				padding: 38rpx 0 30rpx;
			}
			.box{
				display: flex;
				align-items: center;
				justify-content: center;
				padding-bottom: 38rpx;
				.label{
					flex: 0 0 45%;
					display: flex;
					align-items: center;
					flex-direction: column;
					justify-content: center;
					position: relative;
					.text{
						font-size: 24rpx;
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
			.haibao{
				width: 90%;
				margin: 0 auto;
				display: flex;
				flex-direction: column;
				align-items: center;
				image{
					width: 400rpx;
					height: 600rpx;
					border: 1px solid #f5f5f5;
					border-radius: 6rpx;
				}
				.save{
					color: #FFFFFF;
					width: 280rpx;
					height: 60rpx;
					border-radius: 100rpx;
					font-size: 24rpx;
					display: flex;
					align-items: center;
					justify-content: center;
					margin: 30rpx 0;
				}
			}
		}
	}
</style>
