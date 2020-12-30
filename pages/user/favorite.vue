<template>
	<view class="content">
		<view class="goodsList">
			<view class="goods" :class="index==(favoriteList.length-1)?'':'lineBottom'" v-for="(item, index) in favoriteList" :key="index" @click="getGoodsTap(index)">
				<view class="image">
					<image mode="aspectFill" lazy-load :src="item.pic"></image>
					<view class="none" v-if="!item.shelve_status">已下架</view>
				</view>
				<view class="info">
					<view class="title" :style="'color:'+template.color.color2">{{item.title}}</view>
					<view class="intro" :style="'color:'+template.color.color3">{{item.introduce}}</view>
					<view class="box">
						<view class="left" :style="'color:'+template.color.color5">{{item.update_time}}收藏</view>
						<view class="right">
							<view class="btn" @click.stop="delFavoriteTap(item.id)" :style="'background-color:'+template.color.color1">取消收藏</view>
						</view>
					</view>
				</view>
			</view>
			<view class="noneGoodsBox" v-if="favoriteList.length==0">
			    <view class="none" :style="'color:'+template.color.color5">暂无收藏的商品</view>
			</view>
		</view>
		
		<view v-if="goodShow" class="showPopBox" style="display: flex;align-items: center;justify-content: center;">
			<view class="bg" @click="goodShow=false"></view>
			<view class="goodsConten">
				<view class="image">
					<image :src="goodsOneData.pic" lazy-load mode="aspectFit"></image>
					<icon @click="goodShow=false" type="close" size="25" :color="template.color.color5" style="top: 10rpx; right: 10rpx;"></icon>	
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
		
	</view>
</template>

<script>
	import Icon from "@/components/icon/icon.vue"
	export default {
		components: {Icon},
		data() {
			return {
				template: uni.getStorageSync('__yipinTemplateConfig'),
				favoriteList: [],
				goodShow: false,
				goodsOneData: '',
				childProductData: '',
				buyNumber: 0,
				buyNumMax: 0,
				buyActive: true
			}
		},
		onShow() {
			this.getFavoriteList();
		},
		methods: {
			getFavoriteList() {
				let self = this;
				self.Get(self.Url.favoriteList, {}).then(res => {
					if(res.code === 0){
						self.favoriteList = res.data;
					}
				})
			},
			getGoodsTap(index) {
				let self = this;
				let data = self.favoriteList[index];
				if(data.shelve_status){
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
					uni.showToast({title: '该商品已下架',icon: 'none'});
				}
			},
			delFavoriteTap(id) {
				let self = this;
				self.Get(self.Url.favoriteAdd, {id: id, type: false}).then(res => {
					if(res.code == 0){
						uni.showToast({icon: 'none',title: '取消收藏'});
						self.getFavoriteList();
					}
				});
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
			},
		}
	}
</script>

<style lang="less">
	.goodsList{
		padding: 20rpx 0;
		.goods{
			width: 92%;
			margin: 0 auto;
			padding: 20rpx 0;
			display: flex;
			align-items: center;
			position: relative;
			.image{
				width: 150rpx;
				height: 150rpx;
				background-color: #f5f5f5;
				position: relative;
				overflow: hidden;
				border-radius: 6rpx;
				.none{
					position: absolute;
					background-color: rgba(0, 0, 0, 0.5);
					top: 0;
					left: 0;
					font-size: 26rpx;
					width: 100%;
					height: 100%;
					color: #fff;
					display: flex;
					align-items: center;
					justify-content: center;
				}
			}
			.info{
				width: 520rpx;
				padding-left: 20rpx;
				.title{
					font-size:26rpx;
					font-weight:bold;
					width: 95%;
					overflow: hidden;
					white-space: nowrap;
					text-overflow: ellipsis;
				}
				.intro{
					font-size:26rpx;
					overflow: hidden;
					white-space: nowrap;
					text-overflow: ellipsis;
					padding-top: 4rpx;
				}
				.box{
					display: flex;
					align-items: center;
					padding: 10rpx 0 0;
					.left{
						width: 60%;
						font-size: 24rpx;
					}
					.right{
						width: 40%;
						display: flex;
						justify-content: flex-end;
						.btn{
							color: #fff;
							font-size: 24rpx;
							padding: 2rpx 16rpx;
							border-radius: 100rpx;
						}
					}
				}
			}
		}
	}
	.noneGoodsBox{
	    width:100%;
	    height:600rpx;
	    display:flex;
	    align-items:center;
	    justify-content:center;
	    .none{
	        font-size:28rpx;
	        color:#c5c9ca;
	        display:flex;
	        align-items:center;
	        justify-content:center;
	        position: relative;
	    }
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
				height: 200rpx;
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
</style>
