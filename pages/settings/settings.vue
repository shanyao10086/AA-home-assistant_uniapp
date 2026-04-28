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
			<view @click="toggleExpand()" class="funcItem">
				<text>后端服务IP配置</text>
			</view>
			<view v-if="isExpand" class="funcItem" style="height: 180rpx; margin: 20rpx 0;">
				<form>
					<view class="form-item" style="display: flex; flex-direction: column; align-items: flex-start; padding: 20rpx;">
						<text style="font-size: 30rpx; margin-right: 20rpx;">当前使用的后端服务IP: {{backendIP}}</text>
						<input type="text" v-model="updateIP" placeholder="请输入要更改的后端服务IP：" style="font-size: 36rpx; font-weight: 300; margin: 20rpx 0; width: 100%;"/>
						<button @click="updateBackendIP()">修改ip</button>
					</view>
				</form>

			</view>
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
			navBarHeight: 82+11,
			isExpand: false,
			backendIP: '',
			updateIP: ''
		};
	},
	//第一次加载时调用
	created() {
		//获取手机状态栏高度
		this.statusBarHeight = uni.getSystemInfoSync()['statusBarHeight'];
	},
	onShow() {
		this.backendIP = getApp().globalData.baseUrl.split('/')[2] || '';
	},
	methods: {
		// 返回上一页
		goBack() {
			uni.navigateBack({
				delta: 1
			});
		},
		// 切换展开状态
		toggleExpand() {
			this.isExpand = !this.isExpand;
		},
		// 更新后端服务IP
		updateBackendIP() {
			// 确认是否更新IP
			uni.showModal({
				title: '确认更新IP',
				content: '确定要更新后端服务IP吗？',
				showCancel: true,
				cancelText: '取消',
				confirmText: '确认'
			}).then((res) => {
				if (res.confirm) {
					// 检查输入的IP地址	
					uni.request({
						url: `http://${this.updateIP}/check_connextion`,
						method: 'GET',
						timeout: 5000, // 设置请求超时时间
						success: (res) => {
							console.log('测试请求成功:', res);
						},
						fail: (err) => {
							console.error('测试请求失败:', err);
							showToast('请输入有效的IP地址');
							return;
						}
					});
					this.backendIP = this.updateIP;
				}
				getApp().globalData.baseUrl = `http://${this.backendIP}`;
				// 存储到本地缓存
				uni.setStorageSync('baseUrl', getApp().globalData.baseUrl);

				uni.showToast({
					title: '后端服务IP已更新',
					icon: 'success'
				});
				this.updateIP = '';
			}).catch((err) => {
				console.error('更新IP失败:', err);
				uni.showToast({
					title: '更新IP失败',
					icon: 'none'
				});
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
								url: '/pages/personpage/personpage'
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
			uni.navigateTo({ url: '/pages/personpage/personpage' });
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
		color: #FBF2E3;
		background-color: #8CA9AD;
		border-radius: 20rpx;
	}
</style>