<template>
	<view class="root">
		<!-- 自定义导航栏 -->
		<view class="navBarBox">
			<!-- 状态栏占位 -->
			<view class="statusBar" :style="{ paddingTop: statusBarHeight + 'px' }" style="background-color: #8CA9AD;"></view>
			<!-- 真正的导航栏内容 -->
			<view class="navBar">
				<view class="navBarTitle">
					个人中心
				</view>
			</view>
		</view>

		<!-- 顶部欢迎区域 -->
		<view class="header">
			<view class="filterGray">
				<!-- 已登录状态：显示用户信息 -->
				<view class="userInfo" v-if="isLogin">						
					<image class="userAvatar"
					src="/static/avatar/def.png" 
					@click="updataAvatar()"				
					/>
					<view class="userName" @click="changeName()">
						<text>{{ userInfo.nickname }}</text>
					</view>	
				</view>
				
				<!-- 未登录状态：显示登录按钮 -->
				<view class="userInfo" v-else @click="triggerAutoLogin()">						
					<image class="userAvatar"
					src="/static/avatar/def.png" 
					/>
					<view class="userName">
						<text class="login-text">点击登录</text>
					</view>	
				</view>					
			</view>
		</view>

		<view class="funcList">
			<view class="funcItem" @click="toDiet()">
				<text>饮食记录</text>
			</view>
			<view class="funcItem" @click="toSport()">
				<text>运动记录</text>
			</view>
			<view class="funcItem">
				<text>健康周报</text>
			</view>
			<view class="funcItem" @click="toSettings()">
				<text>设置</text>
			</view>
			<view class="funcItem" @click="toAbout()">
				<text>关于</text>
			</view>
		</view>
		<!-- 底部按钮区域 -->
		<view class="big-button">
			<button @click="testJwtConnection()">连接测试</button>
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
			isLogin: false,
			autoLoginAttempted: false // 标记是否已尝试自动登录
		};
	},
	onShow(){
		console.log("personpage onShow");
		// 页面显示时只检查登录状态，不自动登录
		this.checkLoginStatus();
	},
	//第一次加载时调用
	created() {
		//获取手机状态栏高度
		this.statusBarHeight = uni.getSystemInfoSync()['statusBarHeight'];
	},
	methods: {
		

    /**
     * 测试 JWT 接口连通性
     * 调用后端：GET /users/jwt_test
     */
    testJwtConnection() {
      // 1. 从本地存储获取 token
      // 注意：这里 key 必须和登录成功时 uni.setStorageSync('access_token', ...) 的 key 一致
      const token = uni.getStorageSync('access_token');

      if (!token) {
          uni.showToast({
              title: '未找到 Token，请先登录',
              icon: 'none'
          });
          console.error('❌ 测试失败：本地存储中未找到 access_token');
          return;
      }

      console.log('🚀 开始测试 JWT 接口...');
      console.log('🔑 使用 Token:', token.substring(0, 20) + '...');

      // 2. 发起请求
      const baseUrl = getApp().globalData.baseUrl; // 后端地址
      const apiUrl = `${baseUrl}/users/jwt_test`; // 路由路径

      uni.request({
          url: apiUrl,
          method: 'GET',
          header: {
              'Content-Type': 'application/json',
              // ✅ 关键：JWT 标准格式 "Bearer <token>"
              'Authorization': `Bearer ${token}` 
          },
          success: (res) => {
              console.log('📡 原始响应:', res);

              if (res.statusCode === 200) {
                  const data = res.data;
                  console.log('✅ 测试成功！后端返回数据:', data);
                  
                  uni.showModal({
                      title: 'JWT 测试成功',
                      // 对应后端 return jsonify({'message': user_id})
                      content: `后端识别到的用户 ID: ${data.message}`, 
                      showCancel: false
                  });
              } else {
                  console.error('❌ 请求失败，状态码:', res.statusCode);
                  console.error('错误详情:', res.data);
                  
                  let errMsg = '请求失败';
                  if (res.statusCode === 401) {
                      errMsg = 'Token 无效或已过期 (401)';
                  } else if (res.statusCode === 404) {
                      errMsg = '接口地址不存在 (404)，请检查 URL';
                  } else if (res.data && res.data.msg) {
                      errMsg = res.data.msg;
                  }

                  uni.showToast({
                      title: errMsg,
                      icon: 'none'
                  });
              }
          },
          fail: (err) => {
              console.error('❌ 网络请求失败:', err);
              uni.showToast({
                  title: '网络错误或服务器未启动',
                  icon: 'none'
              });
          }
      });
    },
		// 检查登录状态
		checkLoginStatus() {
			// 从本地存储获取用户信息
			this.getUserInfoFromStorage();
			console.log("当前登录状态:", this.isLogin ? "已登录" : "未登录");
		},
		
		// 从本地存储获取用户信息
		getUserInfoFromStorage() {
			try {
				// 使用uni-app提供的存储函数获取用户信息
				const accessToken = uni.getStorageSync('access_token');
				const userInfo = uni.getStorageSync('userInfo');
				
				console.log("从本地存储获取的accessToken:", accessToken);
				console.log("从本地存储获取的用户信息:", userInfo);
				
				if (accessToken && userInfo) {
					// 更新本地状态
					this.userInfo = { ...userInfo };
					this.isLogin = true;
					
					// 更新全局状态
					getApp().globalData.isLogin = true;
					getApp().globalData.userInfo = userInfo;
					getApp().globalData.accessToken = accessToken;
					
					console.log("从本地存储恢复登录状态成功");
				} else {
					// 未登录状态
					this.userInfo = { username: "未登录" };
					this.isLogin = false;
					getApp().globalData.isLogin = false;
					
					console.log("本地存储中未找到有效的登录信息");
				}
			} catch (error) {
				console.error("获取本地存储信息失败:", error);
				this.userInfo = { username: "未登录" };
				this.isLogin = false;
			}
		},
		
		// 触发自动登录
		triggerAutoLogin() {
			console.log("用户点击登录按钮，触发微信登录");
			this.wechatLogin();
		},
		
		// 微信登录（点击触发）- 逻辑：先获取用户信息，再获取code
		wechatLogin() {
			console.log("开始微信登录流程...");
			
			// 第一步：立即调用getUserProfile获取用户授权（必须由用户主动触发）
			uni.getUserProfile({
				provider: 'weixin',
				desc: '用于登录智能家居系统',
				success: async (userRes) => {
					console.log("用户授权成功，获取到用户信息:", userRes.userInfo);
					
					// 显示加载提示
					uni.showLoading({
						title: '登录中...'
					});
					
					try {
						// 第二步：用户授权后，异步调用login获取code
						const loginRes = await new Promise((resolve, reject) => {
							uni.login({
								provider: 'weixin',
								timeout: 5000,
								success: resolve,
								fail: reject
							});
						});
						
						if (loginRes.errMsg === 'login:ok') {
							console.log("微信登录成功，获取到code:", loginRes.code);
							
							// 第三步：调用后端接口进行登录
							const serverRes = await new Promise((resolve, reject) => {
								uni.request({
									url: `${getApp().globalData.baseUrl}/users/wx_login`,
									method: 'POST',
									header: {
										'Content-Type': 'application/json'
									},
									data: {
										code: loginRes.code,
										nickName: userRes.userInfo.nickName,
										avatarUrl: userRes.userInfo.avatarUrl,
										gender: userRes.userInfo.gender,
										city: userRes.userInfo.city,
										province: userRes.userInfo.province,
										country: userRes.userInfo.country
									},
									success: resolve,
									fail: reject
								});
							});
							
							uni.hideLoading();
							
							if (serverRes.statusCode === 200 && serverRes.data) {
								const { message, access_token , user_info:user } = serverRes.data;
								console.log("后端返回信息:", serverRes.data);
								console.log("后端返回用户信息:", user);
								
								// 使用uni-app存储函数保存登录信息
								uni.setStorageSync('access_token', access_token);
								uni.setStorageSync('userInfo', user);
								
								// 更新全局状态
								getApp().globalData.isLogin = true;
								getApp().globalData.userInfo = user;
								getApp().globalData.accessToken = access_token;
								
								// 更新本地状态
								this.userInfo = { ...user };
								this.isLogin = true;
								
								console.log("微信登录成功，用户信息详情:", user);
								console.log("access_token:", access_token);
								
								uni.showToast({
									title: '登录成功',
									icon: 'success'
								});
							} else {
								uni.hideLoading();
								uni.showToast({
									title: serverRes.data?.message || '登录失败',
									icon: 'none'
								});
							}
						} else {
							uni.hideLoading();
							console.log("微信登录失败，用户可能未授权");
							uni.showToast({
								title: '微信登录失败，请重试',
								icon: 'none'
							});
						}
					} catch (error) {
						uni.hideLoading();
						console.error("微信登录失败:", error);
						uni.showToast({
							title: '登录失败，请检查网络',
							icon: 'none'
						});
					}
				},
				fail: (err) => {
					console.error("用户拒绝授权或获取用户信息失败:", err);
					
					// 用户拒绝授权的情况
					if (err.errMsg.includes('auth deny') || err.errMsg.includes('fail auth deny')) {
						uni.showToast({
							title: '需要授权才能使用微信登录',
							icon: 'none'
						});
					} else {
						uni.showToast({
							title: '获取用户信息失败',
							icon: 'none'
						});
					}
				}
			});
		},
		
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
			console.log("用户详细信息:", JSON.stringify(this.userInfo, null, 2));
		},
		updataAvatar(){
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
		toSport(){
			uni.navigateTo({ url: '/pages/sport/sport' });
			},
			toDiet(){
				uni.navigateTo({ url: '/pages/diet/diet' });
			},
			toSettings(){
				uni.navigateTo({ url: '/pages/settings/settings' });
			},
			toAbout(){
				uni.navigateTo({ url: '/pages/about/about' });
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
		margin-top: 50rpx;
		width: 660rpx;
		height: 240rpx;
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
		border-bottom: 6rpx solid #8CA9AD; /* 下边框 */
		border-right: 3rpx solid #8CA9AD; /* 右边框 */
		border-radius: 20rpx;
	}
	.userAvatar{
		width: 120rpx;
		height: 120rpx;
		border-radius: 50%;
	}
	.userName{
		margin: 20rpx;
		color: #8CA9AD;
		font-size: 36rpx;
	}
	.userName text{
		font-size: 48rpx;
		font-weight: 560;
	}
	
	/* 登录按钮样式 */
	.login-text {
		font-size: 48rpx;
		font-weight: 560;
		color: #8CA9AD;
		padding: 20rpx 40rpx;
		transition: all 0.3s ease;
	}
	
	.userInfo[v-else] {
		cursor: pointer;
	}
	
	.userInfo[v-else]:active .login-text {
		background-color: rgba(140, 169, 173, 0.1);
		transform: scale(0.95);
	}
	/* 底部按钮区域样式 */
	.bottomActions {
		margin: 40rpx 0;
		display: flex;
		justify-content: center;
	}

</style>