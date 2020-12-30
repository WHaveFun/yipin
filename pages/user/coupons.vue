<template>
	<view class="content">
	    <view class="header">
			<view class="duih">
				<view class="input">
					<input placeholder="请输入兑换码" v-model="couponsKey" :style="'color:'+template.color.color2" />
				</view>
				<view class="btn" @click="getNewCouponsTap()" :style="couponsKey?'background-color:'+template.color.color1:'background-color:'+template.color.color5">兑换</view>
			</view>
			<view class="menu" :style="'color:'+template.color.color2">
				<view class="label" :class="current==1?'active':''" @click="selectMenuTap(1)">
					<text :style="current==1?'color:'+template.color.color1+';border-color:'+template.color.color1:'color:'+template.color.color2">未使用</text>
				</view>
				<view class="label" :class="current==2?'active':''" @click="selectMenuTap(2)">
					<text :style="current==2?'color:'+template.color.color1+';border-color:'+template.color.color1:'color:'+template.color.color2">已使用</text>
				</view>
				<view class="label" :class="current==3?'active':''" @click="selectMenuTap(3)">
					<text :style="current==3?'color:'+template.color.color1+';border-color:'+template.color.color1:'color:'+template.color.color2">已失效</text>
				</view>
			</view>
	    </view>
		<view style="height: 190rpx;"></view>
	    <view class="noneGoodsBox" v-if="couponList.length==0">
	        <view class="none" :style="'color:'+template.color.color5">暂无相关优惠券</view>
	    </view>
	    <view class="coupons-list" v-else>
	        <view class="coupons-item" v-for="(item, index) in couponList" :key="index">
	            <view class="infoBox">
	                <view class="moneyLeft" :style="'color:'+template.color.color1">{{item.moneys}}<text>元</text></view>
	                <view class="moneyRight">
	                    <view class="name" :style="'color:'+template.color.color2">{{item.title}}</view>
	                    <view class="hold" :style="'color:'+template.color.color3">消费满{{item.min_money}}元使用</view>
	                </view>
	            </view>
	            <view class="footer" :style="'color:'+template.color.color5">
	                <view class="text">
						<text>{{item.end_time}}前有效</text>
					</view>
	                <view class="home">
						<text>{{item.goods_type==1?'全场商品可用':'限指定商品可用'}}</text>
					</view>
	            </view>
	        </view>
	    </view>
	    <!--
	    <view class="addBottom" bindtap="newCouponsTap">领取更多优惠券</view>
	    -->
	</view>
</template>

<script>
	import Icon from "@/components/icon/icon.vue"
	export default {
		components: {Icon},
		data() {
			return {
				template: uni.getStorageSync('__yipinTemplateConfig'),
				current: 1,
				couponList: [],
				couponsKey: ''
			}
		},
		onLoad() {
			this.getCouponsUserList();
		},
		methods: {
			getNewCouponsTap() {
				let self = this;
				let keys = self.couponsKey;
				if(keys){
					self.Get(self.Url.couponsUserAdd, {code: keys}).then(res => {
						if(res.code === 0){
							uni.showToast({title: '兑换成功'});
							self.current = 1;
							self.couponsKey = '';
							self.getCouponsUserList();
						}else{
							uni.showModal({
								title: '兑换失败',
								content: res.msg,
								showCancel: false
							})
						}
					})
				}else{
					uni.showToast({title: '请输入兑换码', icon: 'none'});
				}
			},
			getCouponsUserList() {
				let self = this;
				self.Get(self.Url.couponsUserList, {status: self.current}).then(res => {
					if(res.code === 0){
						self.couponList = res.data;
					}else{
						self.couponList = [];
					}
				})
			},
			selectMenuTap(id) {
				this.current = id;
				this.getCouponsUserList();
			},
		}
	}
</script>

<style lang="less">
	page{
		background-color: #f5f5f5;
	}
	.header{
		width: 100%;
		height: 190rpx;
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		background-color: #fff;
		z-index: 1000;
		.duih{
			padding: 20rpx 0;
			display: flex;
			align-items: center;
			width: 80%;
			margin: 0 auto;
			.input{
				border: 1px solid #f5f5f5;
				height: 70rpx;
				width: 70%;
				border-right: none;
				display: flex;
				align-items: center;
				justify-content: center;
				border-top-left-radius: 100rpx;
				border-bottom-left-radius: 100rpx;
				input{
					width: 88%;
					height: 100%;
					font-size: 28rpx;
				}
			}
			.btn{
				width: 30%;
				height: 74rpx;
				display: flex;
				align-items: center;
				justify-content: center;
				border-top-right-radius: 100rpx;
				border-bottom-right-radius: 100rpx;
				color: #fff;
			}
		}
		.menu{
			display: flex;
			align-items: center;
			justify-content: center;
			padding: 20rpx 0 0;
			position: relative;
			.label{
				flex: 0 0 33.33333%;
				display: flex;
				align-items: center;
				justify-content: center;
				text{
					border-bottom: 4rpx solid #fff;
					padding: 0 10rpx;
					padding-bottom: 14rpx;
					font-size: 28rpx;
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
	.coupons-list{
	    width:90%;
	    margin:0 auto;
	    margin-top:30rpx;
	    .coupons-item{
	        background-color:#fff;
	        margin-bottom:30rpx;
	        border-radius:6rpx;
	        position:relative;
	        display:flex;
	        flex-direction:column;
	        justify-content:center;
	        .infoBox{
	            display:flex;
	            padding: 22rpx 0;
	            align-items:center;
	            .moneyLeft{
	                width:180rpx;
	                align-items:center;
	                justify-content:center;
	                display:flex;
	                color:#f59072;
	                font-size:42rpx;
	                text{
	                    font-size:30rpx;
	                    padding-left:4rpx;
	                }
	            }
	            .moneyRight{
	                display:flex;
	                flex-direction:column;
	                width:480rpx;
	                justify-content:center;
	                .name{
	                    color:#293539;
	                    font-size:30rpx;
						width: 90%;
						white-space: nowrap;
						overflow: hidden;
						text-overflow: ellipsis;
	                }
	                .hold{
	                    color:#6a7275;
	                    font-size:26rpx;
	                    padding-top:6rpx;
	                }
	            }
	        }
	        .footer{
	            display:flex;
	            align-items:center;
				font-size: 24rpx;
	            border-top:2rpx dashed #f5f5f5;
	            .text{
	                flex:0 0 60%;
	                height:60rpx;
	                align-items:center;
	                display:flex;
					text{
						padding-left: 20rpx;
					}
	            }
	            .home{
	                flex:0 0 40%;
	                display:flex;
	                align-items:center;
	                height:60rpx;
	                justify-content:flex-end;
	                padding-right:20rpx;
					text{
						padding-right: 20rpx;
					}
	            }
	        }
	    }
	}
	.addBottom{
	    background-color:#f59072;
	    width:300rpx;
	    height:70rpx;
	    margin:0 auto;
	    display:flex;
	    align-items:center;
	    justify-content:center;
	    color:#fff;
	    border-radius:100rpx;
	    margin-top:100rpx;
	    margin-bottom:100rpx;
	}
</style>
