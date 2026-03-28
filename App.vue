<script>
	// 只能在App.vue里监听应用的生命周期
	export default {
        globalData: {
            isLogin: false,
            userInfo: {
				userid: null,
                username: null
            },
			baseUrl: "http://192.168.17.112:5000",
			accessToken: null
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
	.navBarBox .navBar {
		background-color: #8CA9AD;
		width: 750rpx;
		height: 82rpx;
		padding: 3rpx 50rpx;
		padding-bottom: 8rpx;
		display: flex;
		flex-direction: row;
		justify-content: center;
		align-items: center;
	}
	.navBarTitle{
		color: #FBF2E3;
		font-size: 36rpx;
		font-weight: 500;
	}
	.mainbody{
		width: 660rpx;
        background-color: #FBF2E3;
		color: #8CA9AD;
		display: flex;
		justify-content: center;
		align-items: center;
	}
    .escp{
        position: absolute;
        top: 20rpx;
        left: 15rpx;
    }
	.escimg{
		width: 60rpx;
		height: 60rpx;
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