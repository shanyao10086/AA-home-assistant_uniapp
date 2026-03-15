<template>
	<view class="root">
		<!-- 自定义导航栏 -->
		<view class="navBarBox">
			<!-- 状态栏占位 -->
			<view class="statusBar" :style="{ paddingTop: statusBarHeight + 'px' }"></view>
			<!-- 真正的导航栏内容 -->
			<view class="navBar">
				<view style="color: #FBF2E3;font-size: 36rpx;font-weight: 500;">
					个人中心
				</view>
			</view>
		</view>

		<!-- 顶部欢迎区域 -->
		<view class="header">
			<view class="filterGray">
				<!-- 内容容器：头像 + 文字 -->
				<view class="userInfo">						
					<image class="userAvatar"
					src="/static/avatar/def.png" 
					@click="changeAvatar()"			
					/>
					<view class="userName"	@click="changeName()">
						<text>{{ userInfo.username }}</text>
					</view>	
				</view>				
			</view>
		</view>

		<view class="funcList">
			<view class="funcItem">
				<view class="funcName">
					<text>饮食记录</text>
				</view>
			</view>
			<view class="funcItem">
				<view class="funcName">
					<text>运动记录</text>
				</view>
			</view>
			<view class="funcItem">
				<view class="funcName">
					<text>健康周报</text>
				</view>
			</view>
			<view class="funcItem">
				<view class="funcName">
					
					<text>App设置</text>
				</view>
			</view>
			<view class="funcItem">
				<view class="funcName">
					<text>关于</text>
				</view>
			</view>
		</view>
		<view class="bigbutton">
			<button @click="exitLogin()">退出登录</button>
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
			userInfo: {
				username: "未登录"
			},
			isLogin: false
		};
	},
	onShow(){
		console.log("personpage onShow");
		this.updateUserInfo();
	},
	//第一次加载时调用
	created() {
		//获取手机状态栏高度
		this.statusBarHeight = uni.getSystemInfoSync()['statusBarHeight'];
	},
	methods: {
		updateUserInfo(){
			// 从全局状态获取最新用户信息
			const globalData = getApp().globalData;
			this.userInfo = { ...globalData.userInfo };
			this.isLogin = globalData.isLogin;
			
			// 如果未登录，显示默认信息
			if (!this.isLogin) {
				this.userInfo.username = "未登录";
			}
			
			console.log("用户信息已更新:", this.userInfo);
		},
		changeAvatar(){
			this.updateUserInfo(); // 确保获取最新的用户信息
			console.log("userInfo:", this.userInfo);
			if (!this.isLogin) {
				console.log("请先登录");
				uni.navigateTo({ url: '/pages/login/login' });
				return;
			}
			console.log(" 更改头像 ");
		},
		changeName(){
			this.updateUserInfo(); // 确保获取最新的用户信息
			console.log(" 用户名:", this.userInfo.username);
			if (!this.isLogin) {
				console.log("请先登录");
				uni.navigateTo({ url: '/pages/login/login' });
				return;
			}
			console.log(" 更改昵称 ");
		},
		exitLogin(){
			console.log(" 退出登录 ");
			getApp().globalData.isLogin = false;
			getApp().globalData.userInfo = { username: "未登录" };
			uni.navigateTo({ url: '/pages/login/login' });
		},
		debugshow(){
			console.log("userInfo:", this.userInfo);
			console.log("isLogin:", this.isLogin);
		}
	}
};
</script>

<style>
	/* 顶部欢迎区域 */
	.header {
		margin-top: 30rpx;
		width: 660rpx;
		height: 240rpx;/*  可根据需要调整高度 */
		/* border-radius: 20rpx;
		background-color: #8CA9AD;
		background-image: 
		url('/static/images/profile-bg-cp.png'); 
		background-repeat: no-repeat;
		background-size: cover;
		background-position: center; */
	}

	.filterGray{
		/* background-color: rgba(0, 0, 0, 0.4); */
		border-radius: 20rpx;
	}
	/* 内容容器：叠加在背景图上 */
	.userInfo {
		height: 200rpx;
		margin-bottom: 50rpx;
		padding: 40rpx 40rpx; /* 左右留一点内边距，避免贴边 */
		display: flex;
		align-items: center;
		border-bottom: 4rpx solid #8CA9AD;
		border-right: 3rpx solid #8CA9AD;
		border-radius: 20rpx;
	}
	.userName{
		margin: 20rpx;
		color: #8CA9AD;
		font-size: 36rpx;
		text{
			font-size: 45rpx;
			font-weight: 500;
		}
	}
	.funcList{
		width: 660rpx;
		margin-top: 60rpx;
	}
	.funcItem{
		width: 100%;
		height: 80rpx;
		margin-top: 10rpx;
		color: #8CA9AD;
		border-bottom: 4rpx solid #8CA9AD;
		border-right: 3rpx solid #8CA9AD;
		border-radius: 20rpx;
		display: flex;
		align-items: center;
		text{
			margin-left: 20rpx;
			font-size: 30rpx;
			font-weight: 500;
		}
	}

</style>