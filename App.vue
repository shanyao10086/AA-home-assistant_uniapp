<script>
	// 只能在App.vue里监听应用的生命周期
	export default {
        globalData: {
            isLogin: false,
            userInfo: {
                username: "未登录"
            }
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
	.bigbutton{
		margin-top: 40rpx;
		height: 80rpx;
		display: flex;
		justify-content: center;
		button{
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
	}
	.footer-space {
	  height: 120rpx; /* 避免被 tabBar 遮挡 */
	}
	
</style>