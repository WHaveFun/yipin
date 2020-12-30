<template>
	<view class="content">
		<view class="formBox">
			<view class="label lineBottom">
				<view class="title" :style="'color:'+template.color.color3">联系人</view>
				<view class="input">
					<input type="text" v-model="addressData.cname" placeholder="请输入收货人姓名" :style="'color:'+template.color.color2" />
				</view>
			</view>
			<view class="label lineBottom">
				<view class="title" :style="'color:'+template.color.color3">手机号</view>
				<view class="input">
					<input type="number" :maxlength="11" v-model="addressData.phone" placeholder="请输入联系手机号码" :style="'color:'+template.color.color2" />
				</view>
			</view>
			<!-- <view class="label lineBottom">
				<view class="title" :style="'color:'+template.color.color3">身份证</view>
				<view class="input">
					<input type="number" v-model="addressData.idcard" placeholder="如需要海关报关则必填" :style="'color:'+template.color.color2" />
				</view>
			</view> -->
			<view class="label lineBottom">
				<view class="title" :style="'color:'+template.color.color3">所属区域</view>
				<view class="input">
					<picker mode="region" :value="region" @change="selectRegion">
						<view class="picker" :style="'color:'+template.color.color2">
							<block v-for="(item, index) in region" :key="index">
								<text v-if="index>0">-</text>{{item}}
							</block>
						</view>
					</picker>
				</view>
			</view>
			<view class="label lineBottom">
				<view class="title" :style="'color:'+template.color.color3">街道信息</view>
				<view class="input">
					<input type="text" v-model="addressData.address" placeholder="请输入详细的街道门牌信息" :style="'color:'+template.color.color2" />
				</view>
			</view>
			<view class="button">
				<view class="saveBtn" @click="saveAddress()" :style="'background-color:'+template.color.color1">保存地址信息</view>
				<view class="saveBtn" v-if="addressData.id" @click="delAddress()" :style="'background-color:'+template.color.color5">删除地址信息</view>
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
				region: ['北京市','北京市','东城区'],
				addressData: {
					id: '',
					cname: '',
					phone: '',
					idcard: '',
					province: '',
					region_id: "110101",
					address: '',
					is_default: true
				}
			}
		},
		onLoad(e) {
			let self = this;
			let temp = self.template;
			if(e.id){
				uni.setNavigationBarTitle({title: '修改收货地址'});
				self.Get(self.Url.addressList, {id: e.id}).then(res => {
					if(res.code === 0){
						self.addressData = res.data[0];
						self.region = res.data[0].province.split(',');
					}
				})
			}else{
				uni.setNavigationBarTitle({title: '添加收货地址'});
			}
		},
		methods: {
			selectRegion(e) {
				this.region = e.detail.value;
				this.addressData.region_id = e.detail.code[2];
			},
			saveAddress() {
				let self = this;
				let data = self.addressData;
				if(!data.cname){
					uni.showModal({
						title: '地址保存提示',
						content: '请输入收货人姓名',
						showCancel: false
					});
					return
				}else if(!data.phone ){
					uni.showModal({
						title: '地址保存提示',
						content: '请输入联系手机号码',
						showCancel: false
					});
					return
				} else if(!(/^1[3456789]\d{9}$/.test(data.phone))){
					uni.showModal({
						title: '手机格式错误',
						content: '请输入正确的手机号码',
						showCancel: false
					});
					return
				} else if(!data.address){
					uni.showModal({
						title: '地址保存提示',
						content: '请输入详细的街道门牌信息',
						showCancel: false
					});
					return
				}else{
					data.is_default = true;
					data.province = self.region.join(',');
					self.Post(self.Url.addressEdit, data).then(res => {
						if(res.code == 0){
							uni.showToast({title: '地址保存成功'});
							setTimeout(function () {
								uni.navigateBack();
							}, 500);
						}else{
							uni.showModal({
								content: res.mag,
								showCancel: false
							});
						}
					});
				}
			},
			delAddress() {
				let self = this;
				uni.showModal({
					title: '删除地址',
					content: '地址删除后不可恢复，是否删除该地址？',
					success(res) {
						if(res.confirm){
							self.Get(self.Url.addressDel, {id: self.addressData.id}).then(res => {
								if(res.code === 0){
									uni.showToast({title: '地址删除成功'});
									setTimeout(function () {
										uni.navigateBack();
									}, 500);
								}else{
									uni.showModal({
										content: res.mag,
										showCancel: false
									});
								}
							})
						}
					}
				})
			}
		}
	}
</script>

<style lang="less">
	page{
		background-color: #FFFFFF;
	}
	.formBox{
		padding-top: 30rpx;
		.label{
			display: flex;
			align-items: center;
			width: 90%;
			margin: 0 auto;
			padding: 18rpx 0;
			position: relative;
			.title{
				flex: 0 0 20%;
				color: #222;
				font-size: 28rpx;
				text-align: right;
			}
			.input{
				flex: 0 0 80%;
				display: flex;
				align-items: center;
				justify-content: center;
				input{
					width: 92%;
					height: 60rpx;
					font-size: 28rpx;
					color: #222;
				}
				picker{
					width: 92%;
					.picker{
						width: 100%;
						height: 60rpx;
						line-height: 60rpx;
						font-size: 28rpx;
						color: #222;
						white-space: nowrap;
						text-overflow: ellipsis;
						overflow: hidden;
						text{
							color: #bbb;
							padding: 0 12rpx;
						}
					}
				}
			}
		}
		.button{
			width: 90%;
			margin: 0 auto;
			display: flex;
			justify-content: center;
			flex-direction: column;
			align-items: center;
			margin-top: 50rpx;
			.saveBtn{
				width: 52%;
				height:70rpx;
				line-height: 70rpx;
				color: #fff;
				text-align: center;
				border-radius: 100rpx;
				margin-bottom: 30rpx;
			}
		}
	}
</style>
