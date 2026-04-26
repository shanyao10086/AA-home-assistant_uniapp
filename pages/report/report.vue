<template>
	<view class="root">
		<!-- 自定义导航栏 -->
		<view class="navBarBox">
			<!-- 状态栏占位 -->
			<view class="statusBar" :style="{ paddingTop: statusBarHeight + 'px' }" style="background-color: #8CA9AD;"></view>
			<!-- 真正的导航栏内容 -->
			<view class="navBar">
				<!-- 返回按钮 -->
				<view class="escp" @click="goBack()">
					<image class="escimg"
					src="/static/icons/向左.png"></image>
				</view>
				<text class="navBarTitle">健康周报</text>
			</view>
		</view>
		
		
		<view class="bigbutton">
			<button @click="getReport()">生成周报</button>
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
			uni.navigateBack({ delta: 1 });
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
	
	.bigbutton {
		width: 90%;
		margin: 20rpx auto;
	}
	
	button {
		width: 100%;
		color: #FBF2E3;
		background-color: #8CA9AD;
		border-radius: 20rpx;
	}
</style>