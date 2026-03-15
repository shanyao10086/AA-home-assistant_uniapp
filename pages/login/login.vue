<template>
	<view class="root">
		<!-- 自定义导航栏 -->
		<view class="navBarBox">
			<!-- 状态栏占位 -->
			<view class="statusBar" :style="{ paddingTop: statusBarHeight + 'px' }"></view>
			<!-- 真正的导航栏内容 -->
			<view class="navBar">
				<view class="escp" @click="$router.back()">
					<image src="/static/icons/ic-general-arrow-left.png" class="escimg"></image>
				</view>
			</view>
		</view>
		
		<view class="titlebody">
			<text>登录一下</text>
		</view>
		<view class="loginbody">
			<view class="inputmsg">
				<view class="container">
					<text>用户名：</text>
					<input type="text" 
					v-model="username"
					placeholder="请输入用户名" />
				</view>
				<view class="container">
					<text>密码：</text>
					<input type="password" 
					v-model="password"
					placeholder="请输入密码" />
				</view>
				<view class="bigbutton">
					<button @click="userLogin">登 录</button>
				</view>	
			</view>
		</view>
		<view class="else">
			<text class="register" @click="register()">没有账号？注册一下</text>
			<text class="forgetpsd" @click="findpsd()">忘记密码？</text>
		</view>
		<view class="footer-space" style="color: #8CA9AD;"></view>
	</view>
</template>
<script>
export default{
	data() {
		return {
			username: '',
			password: '',
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
	//第一次加载时调用
	created() {
		//获取手机状态栏高度
		this.statusBarHeight = uni.getSystemInfoSync()['statusBarHeight'];
	},
	methods: {
		register(){
			console.log("注册");
		},
		findpsd(){
			console.log("忘记密码");
		},
		userLogin(event) {
			console.log("=== 登录测试 ===");
			console.log("用户名:", this.username);
			console.log("密码:", this.password);
			console.log("输入状态:", this.username ? "用户名已输入" : "用户名未输入",
			this.password ? "密码已输入" : "密码未输入");
		
			// 简单的验证逻辑
			if(!this.username || !this.password) {
				console.log("❌ 登录失败: 用户名或密码不能为空");
				return;
			}
				
			console.log("✅ 登录信息验证通过，可以提交到服务器");
				
			// 模拟API调用
			setTimeout(() => {
				console.log("📡 模拟API请求中...");
				console.log( event ); // 输出事件对象，查看详细信息
				console.log("请求参数: {", "username:", this.username, ", password:", this.password, "}");
				console.log("🎉 登录成功!");
				// 更新全局状态
				getApp().globalData.isLogin = true;
				getApp().globalData.userInfo = {
					username: this.username
				};
				console.log("全局状态已更新:", getApp().globalData);
				uni.switchTab({ url: '/pages/personpage/personpage' });
			}, 1000);
			return;
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
	.titlebody{
		margin-top: 200rpx;
		width: 660rpx;
		height: 150rpx;
		color: #8CA9AD;
		font-weight: 770;
		font-size: 66rpx;
		text-align: center;
		display: flex;
		justify-content: center;
		align-items: center;
	}
	.loginbody{
		margin-top: 20rpx;
		text-align: center;
		font-size: 40rpx;
		display: flex;
		justify-content: center;
		align-items: center;
		
		.container{
			margin-top: 20rpx;
			width: 660rpx;
			height: 85rpx;
			display: flex;
			align-items: center;
			border-bottom: #8CA9AD 4rpx solid;
			border-right: #8CA9AD 3rpx solid;
			border-radius: 20rpx;
			text{
				width: 150rpx;
				color: #8CA9AD;
				font-size: 35rpx;
			}
			input{
				width: 500rpx;
				height: 60rpx;
				color: #8CA9AD;
			}
		}
	}
	.else{
		margin: 3% 7%;
		width: 660rpx;
		text{
			color: #8CA9AD;
			font-size: 30rpx;
		}
	}
	.register{
		float: left;
	}
	.forgetpsd{
		float: right;
	}
</style>