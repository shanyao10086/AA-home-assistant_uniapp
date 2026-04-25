<script>
	// 只能在App.vue里监听应用的生命周期
	export default {
        globalData: {
            isLogin: false,
            userInfo: {
				userid: null,
                username: null
            },
			baseUrl: "http://192.168.128.112:5000",
			accessToken: null,
			deviceList: []
        },
		onLaunch: function(options) {
			console.log('App Launch')
			console.log('应用启动路径：', options.path)
		},
		onShow: function(options) {
			console.log('App Show')
			console.log('应用启动路径：', options.path)
		},
		onHide: function() {
			console.log('App Hide')
		},
		methods:{
			
		}
	}
</script>


<style lang="scss">
    /* 自定义全局样式 */
    body {
        background-color: rgb(251,242,227); //#FBF2E3;
        color: rgb(140,169,173);    // #8CA9AD;
    }
	.root{
        width: 100%;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}
	.navBarBox {
		width: 100%;
	}
	
	.statusBar {
		width: 100%;
	}
	
	.navBar {
		width: 100%;
		height: 88rpx;
		display: flex;
		align-items: center;
		justify-content: center; /* 保持居中 */
		position: relative;
		background-color: #8CA9AD;
		font-size: 36rpx;
		font-weight: bold;
	}

	.escimg{
		width: 60rpx;
		height: 60rpx;
	}
	
	.escp {
		position: absolute;
		left: 20rpx;
		top: 50%;
		transform: translateY(-50%);
		display: flex;
		align-items: center;
		justify-content: center;
		width: 60rpx;
		height: 60rpx;
		z-index: 10;
	}
	
	.navBarTitle {
		color: #FBF2E3;
		font-size: 36rpx;
		font-weight: bold;
		text-align: center;
		position: absolute; /* 使用绝对定位，相对于.navBar */
		left: 0;
		right: 0;
		top: 50%;
		transform: translateY(-50%); /* 垂直居中 */
		text-align: center;
		z-index: 1; /* 确保标题在返回按钮后面，但显示在中间 */
	}
	
	.navBarBox .navBar {
		background-color: #8CA9AD;
		width: 100%;
		height: 82rpx;
		display: flex;
		flex-direction: row;
		justify-content: center;
		align-items: center;
		position: relative;
	}
	.mainbody{
		width: 660rpx;
        background-color: #FBF2E3;
		color: #8CA9AD;
		display: flex;
		justify-content: center;
		align-items: center;
	}
	.card-border{
		border-bottom: 6rpx solid #8CA9AD; /* 下边框 */
		border-right: 3rpx solid #8CA9AD; /* 右边框 */
		border-radius: 20rpx;
	}
	.funcList{
		width: 660rpx;
		margin-top: 60rpx;
	}
	.funcItem{
		width: 100%;
		height: 90rpx;
		margin-top: 10rpx;
		color: #8CA9AD;
		border-bottom: 6rpx solid #8CA9AD;
		border-right: 3rpx solid #8CA9AD;
		border-radius: 20rpx;
		display: flex;
		align-items: center;
	}
	.funcItem text{
		margin-left: 30rpx;
		font-size: 36rpx;
		font-weight: 480;
	}
	.big-button{
		margin-top: 40rpx;
		height: 80rpx;
		display: flex;
		justify-content: center;
	}
	.big-button	button{
		color: #FBF2E3;
		background-color: #8CA9AD;
		display: flex;
		justify-content: center;
		align-items: center;
		height: 80rpx;
		width: 660rpx;
		font-size: 40rpx;
		border: none;
		border-radius: 20rpx;
		padding: 10rpx 40rpx;
	};
	.footer-space {
	  height: 120rpx; /* 避免被 tabBar 遮挡 */
	}

	
	
</style>