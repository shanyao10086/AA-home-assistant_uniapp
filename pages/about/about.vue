<template>
	<view class="root">
		<!-- 自定义导航栏 -->
		<view class="navBarBox">
			<!-- 状态栏占位 -->
			<view class="statusBar" :style="{ paddingTop: statusBarHeight + 'px' }" style="background-color: #8CA9AD;"></view>
			<!-- 真正的导航栏内容 -->
			<view class="navBar">
				<view class="escp" @click="goBack()">
					<image class="escimg"
					src="/static/icons/向左.png"></image>
				</view>
			</view>
		</view>
		
		
		<view class="aboutContent" style="margin-top: 200rpx; color: #8CA9AD;">
			<text style=" text-align: center;font-size: 36rpx; font-weight: 500;">关于我们</text>
		</view>
		<view style="margin:50rpx 60rpx; color: #8CA9AD;">
			<text style=" width: 80%; margin: 0 30rpx;font-size: 28rpx; font-weight: 500;">
				AA-Home Assistant 是一个基于 Vue 的微信小程序，旨在为用户提供智能设备管理、饮食管理和个性化建议等功能。
			</text>
		</view>
		<view class="footer-space" style="color: #8CA9AD;"></view>
	</view>
</template>

<script>
export default{
	data() {
		return {
			confirmPassword: '',
			// 状态栏高度
			statusBarHeight: 0,
			// 导航栏高度
			navBarHeight: 82+11
		};
	},
	//第一次加载时调用
	created() {
		//获取手机状态栏高度
		this.statusBarHeight = uni.getSystemInfoSync()['statusBarHeight'];
	},
	methods: {
		// 返回上一页
		goBack() {
			uni.navigateBack({
				delta: 1
			});
		}
	}
}
</script>

<style>
	.root{
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}
	
</style>