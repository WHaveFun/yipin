<template>
	<view class="content">
	    <view class="noneGoodsBox" v-if="addressList.length==0">
	        <view class="none" :style="'color:'+template.color.color5">暂无收货地址</view>
	    </view>
	    <view v-else class="addressList">
	        <view class="list lineBottom" v-for="(item, index) in addressList" :key="index" @click="selectAddress(index)">
				<view class="left" :style="'color:'+template.color.color2">
					<view v-if="item.is_default" class="status"><text :style="'background-color:'+template.color.color1">默认地址</text></view>
					<view class="title">{{item.cname}} / {{item.phone}}</view>
					<view class="text" :style="'color:'+template.color.color3">{{item.province.replace(/,/g,'')}}{{item.address}}</view>
				</view>
				<view class="right" @click.stop="addAddess(item.id)">
					<icon type="bianji" size="25" :color="template.color.color5"></icon>
				</view>
	        </view>
	    </view>
	    <view class="addBottom" @click="addAddess()" :style="'background-color:'+template.color.color1">新增地址</view>
	</view>
</template>

<script>
	import Icon from "@/components/icon/icon.vue"
	export default {
		components: {Icon},
		data() {
			return {
				template: uni.getStorageSync('__yipinTemplateConfig'),
				addressList: []
			}
		},
		onShow() {
			let self = this;
			self.Get(self.Url.addressList, {}).then(res => {
				if(res.code === 0){
					self.addressList = res.data;
				}
			})
		},
		methods: {
			selectAddress(index) {
				let self = this;
				let data = self.addressList[index];
				data.is_default = true;
				self.Post(self.Url.addressEdit, data).then(res => {
					if(res.code === 0){
						uni.navigateBack();
					}
				});
			},
			addAddess(id) {
				if(id){
					uni.navigateTo({url: '/pages/user/adsadd?id='+id});
				}else{
					uni.navigateTo({url: '/pages/user/adsadd'});
				}
			}
		}
	}
</script>

<style lang="less">
	page{
		background-color: #FFFFFF;
	}
	.noneGoodsBox{
	    width:100%;
	    height:700rpx;
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
	.addressList{
		padding-top: 20rpx;
		.list{
			display: flex;
			align-items: center;
			width: 90%;
			margin: 0 auto;
			position: relative;
			padding: 12rpx 0;
			margin-bottom: 10rpx;
			.left{
				flex: 0 0 85%;
				.status{
					text{
						color: #fff;
						padding: 2rpx 16rpx;
						font-size: 24rpx;
						border-radius: 6rpx;
					}
				}
				.title{
					width: 570rpx;
					overflow: hidden;
					text-overflow: ellipsis;
					white-space: nowrap;
					font-size: 30rpx;
					padding-top: 8rpx;
				}
				.text{
					width: 570rpx;
					overflow: hidden;
					text-overflow: ellipsis;
					white-space: nowrap;
					padding-top: 4rpx;
					padding-bottom: 6rpx;
					font-size: 26rpx;
				}
			}
			.right{
				flex: 0 0 15%;
				height: 80rpx;
				display: flex;
				align-items: center;
				justify-content: center;
			}
		}
	}
	.addBottom{
		position: fixed;
		left: 0;
		right: 0;
		bottom: 60rpx;
	    background-color:#f59072;
	    width:300rpx;
	    height:70rpx;
	    margin:0 auto;
	    display:flex;
	    align-items:center;
	    justify-content:center;
	    color:#fff;
	    border-radius:100rpx;
	    span{
	        padding-right:10rpx;
	    }
	}
</style>
