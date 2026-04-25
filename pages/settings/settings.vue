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
				<text class="navBarTitle">设置</text>
			</view>
		</view>
		
		
		<view class="funcList">
			<view class="funcItem">
				<text>网关配置</text>
			</view>
		</view>
		<view class="bigbutton">
			<button @click="confirmLogout()">注销账号</button>
		</view>
		<view class="bigbutton">
			<button @click="exitLogin()">退出登录</button>
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
		},

		// 注销账号
		logout(){
			// 获取当前登录用户信息
			const app = getApp();
			const userId = app.globalData.userInfo.id; // 假设用户ID存储在userInfo.id中
			const accessToken = uni.getStorageSync('access_token');
			
			if (!userId) {
				uni.showToast({
					title: '无法获取用户信息',
					icon: 'none'
				});
				return;
			}
			
			if (!accessToken) {
				uni.showToast({
					title: '请先登录',
					icon: 'none'
				});
				return;
			}
			
			// 显示加载提示
			uni.showLoading({
				title: '注销中...'
			});
			
			// 调用后端注销API
			uni.request({
				url: `${getApp().globalData.baseUrl}/api/users/${userId}/delete`,
				method: 'DELETE',
				header: {
					'Content-Type': 'application/json',
					'Authorization': `Bearer ${accessToken}`
				},
				success: (res) => {
					uni.hideLoading();
					
					if (res.statusCode === 200 && res.data) {
						const { message } = res.data;
						
						uni.showToast({
							title: message || '账号已注销',
							icon: 'success'
						});
						
						// 清除本地存储的用户信息
						uni.removeStorageSync('access_token');
						uni.removeStorageSync('userInfo');
						
						// 更新全局状态
						app.globalData.isLogin = false;
						app.globalData.userInfo = { username: "未登录" };
						app.globalData.accessToken = null;
						
						// 跳转到登录页面
						setTimeout(() => {
							uni.reLaunch({
								url: '/pages/login/login'
							});
						}, 1500);
					} else {
						// 注销失败
						let errorMessage = '注销失败';
						if (res.data && res.data.message) {
							errorMessage = res.data.message;
						} else if (res.statusCode === 403) {
							errorMessage = '权限不足';
						} else if (res.statusCode === 404) {
							errorMessage = '用户不存在';
						}
						
						uni.showToast({
							title: errorMessage,
							icon: 'none'
						});
					}
				},
				fail: (err) => {
					uni.hideLoading();
					uni.showToast({
						title: '网络错误，请重试',
						icon: 'none'
					});
					console.error('注销请求失败:', err);
				}
			});
		},
		confirmLogout(){
			uni.showModal({
				title: '确认注销',
				content: '确定注销账号吗？',
				success: (res) => {
					if (res.confirm) {
						console.log('用户点击了确认');
						// 执行注销账号
						this.logout();
					} else if (res.cancel) {
						console.log('用户点击了取消');
						// 不执行任何操作
					}
				}
			});
		},
		exitLogin(){
			console.log(" 退出登录 ");
			getApp().globalData.isLogin = false;
			getApp().globalData.userInfo = { username: "未登录" };
			uni.navigateTo({ url: '/pages/login/login' });
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
	
	.funcList {
		width: 100%;
		padding: 40rpx 20rpx;
		box-sizing: border-box;
	}
	
	.funcItem {
		width: 100%;
		padding: 20rpx;
		margin-bottom: 20rpx;
		border-radius: 10rpx;
		text-align: center;
	}
	
	.bigbutton {
		width: 90%;
		margin: 20rpx auto;
	}
	
	button {
		width: 100%;
		background-color: #8CA9AD;
		color: white;
		border-radius: 10rpx;
	}
</style>