<template>
	<view class="root">
		<!-- 自定义导航栏 -->
		<view class="navBarBox">
			<!-- 状态栏占位 -->
			<view class="statusBar" :style="{ paddingTop: statusBarHeight + 'px' }"></view>
			<!-- 真正的导航栏内容 -->
			<view class="navBar">
				<view style="color: #FBF2E3;font-size: 36rpx;font-weight: 500;">
					设备中心
				</view>
			</view>
		</view>

		<view class="devList">

			<view class="devItem" v-for="item in deviceList">
				<view class="devName">
					<text>{{ item.name }}</text>
				</view>
			</view>
		</view>
		<view class="addButton">
			<image src="/static/icons/加.png" @click="addDev()"/>
		</view>
		<view class="footer-space"></view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			// 状态栏高度
			statusBarHeight: 0,
			// 导航栏高度
			navBarHeight: 82+11,
      deviceList: [
				{ name: "客厅智能灯" },
				{ name: "卧室空调" },
				{ name: "温湿度传感" },
				{ name: "自动窗帘" },
				{ name: "大门" },
				{ name: "卧室空调" },
				{ name: "温湿度传感" },
				{ name: "自动窗帘" },
				{ name: "大门" },
				{ name: "卧室空调" },
				{ name: "温湿度传感" },
				{ name: "自动窗帘" },
				{ name: "大门" },
				{ name: "卧室空调" },
				{ name: "温湿度传感" },
				{ name: "自动窗帘" },
				{ name: "大门" }
			],
			userInfo: {
				username: "未登录"
			},
			isLogin: false
		};
	},
	onShow(){
		console.log("devicepage onShow");
	},
	//第一次加载时调用
	created() {
		//获取手机状态栏高度
		this.statusBarHeight = uni.getSystemInfoSync()['statusBarHeight'];
	},
	methods: {
		updateDevInfo(){
			// 从获取最新设备信息
			const globalData = getApp().globalData;
		},
    addDev(){
			console.log(" 添加设备 ");
		}
	}
};
</script>

<style>
	.devList{
		width: 660rpx;
		margin-top: 70rpx;
	}
	.devItem{
		width: 100%;
		height: 90rpx;
		margin-top: 20rpx;
		color: #8CA9AD;
		border-bottom: 4rpx solid #8CA9AD;
		border-right: 3rpx solid #8CA9AD;
		border-radius: 20rpx;
		display: flex;
		align-items: center;
		text{
			margin-left: 40rpx;
			font-size: 36rpx;
			font-weight: 500;
		}
	}
  .addButton{
		position: fixed;
    bottom: 160rpx;
    right: 70rpx;
    image{
      width: 100rpx;
      height: 100rpx;
    }
	}
  .footer-space {
	  height: 250rpx; /* 避免被 add按钮 遮挡 */
	}
</style>