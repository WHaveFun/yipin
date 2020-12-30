<template>
	<view class="power" @click="viewDdminiTap()">
		<view v-if="brandInfo.mini_status" class="logo">
			<image :src="brandInfo.mini_logo"></image>
			<text>{{brandInfo.mini_title}}</text>
		</view>
		<view v-if="brandInfo.mini_status" class="text">提供技术与平台服务</view>
		<view class="text">v1.0.4</view>
	</view>
</template>

<script>
	import Icon from "@/components/icon/icon.vue"
	export default {
		name: 'about',
		components: {Icon},
		props: {},
		data() {
			return {
				brandInfo: ''
			}
		},
		onReady() {
			let self = this;
			self.Get(self.Url.brandInfo, {}).then(res => {
				if(res.code === 0){
					self.brandInfo = res.data;
				}
			})
		},
		methods: {
			viewDdminiTap() {
				let self = this;
				if(self.brandInfo.mini_status){
					uni.navigateTo({url: '/pages/about'});
				}
			}
		}
	}
</script>

<style lang="less">
	.power{
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		padding: 50rpx;
		.logo{
			display: flex;
			align-items: center;
			color: #e2e2e2;
			image{
				width: 36rpx;
				height: 36rpx;
				filter: brightness(90%);
			}
			text{
				padding-left: 6rpx;
			}
		}
		.text{
			font-weight: 300;
			color: #e2e2e2;
			font-size: 24rpx;
		}
	}
</style>
