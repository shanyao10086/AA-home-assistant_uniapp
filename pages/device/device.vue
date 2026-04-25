<template>
	<view class="root">
		<!-- 自定义导航栏 -->
		<view class="navBarBox">
			<!-- 状态栏占位 -->
			<view class="statusBar" :style="{ paddingTop: statusBarHeight + 'px' }"style="background-color: #8CA9AD;"></view>
			<!-- 真正的导航栏内容 -->
			<view class="navBar">
				<view style="color: #FBF2E3;font-size: 36rpx;font-weight: 500;">
					设备中心
				</view>
			</view>
		</view>

		<view class="devList" v-if="deviceList.length > 0">
			<view class="devItem" v-for="item in deviceList" :key="item.id">
				<view class="devHeader" @click="toggleDeviceDetail(item.id)">
					<view class="devName">
						<text>{{ item.device_name }}</text>
					</view>
					<image class="expand-icon" :class="{ expanded: expandedDeviceId === item.id }" src="/static/icons/向下.png"></image>
				</view>
				<view class="devDetail" v-if="expandedDeviceId === item.id">
					<view class="detail-row">
						<text class="detail-label">设备类型:</text>
						<text class="detail-value">{{ getDeviceTypeLabel(item.device_type) }}</text>
					</view>
					<view class="detail-row">
						<text class="detail-label">位置:</text>
						<text class="detail-value">{{ item.location || '未设置' }}</text>
					</view>
					<view class="detail-row">
						<text class="detail-label">序列号:</text>
						<text class="detail-value">{{ item.serial_number }}</text>
					</view>
					<view class="detail-row">
						<text class="detail-label">状态:</text>
						<text class="detail-value" :class="{ 'status-online': item.status === 'online', 'status-offline': item.status !== 'online' }">{{ item.status === 'online' ? '在线' : '离线' }}</text>
					</view>
					<view class="detail-row">
						<text class="detail-label">最后在线:</text>
						<text class="detail-value">{{ formatDate(item.last_seen) }}</text>
					</view>
					<view class="detail-row">
						<text class="detail-label">创建时间:</text>
						<text class="detail-value">{{ formatDate(item.created_at) }}</text>
					</view>
				</view>
			</view>
		</view>
		<view class="noDev" style="margin-top: 200rpx; color: #8AA9AD;" v-else>
			<text style="font-size: 36rpx;font-weight: 500;">暂无设备，点击右下角 + 号添加设备</text>
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
      		deviceList: [],
			expandedDeviceId: null, // 当前展开的设备ID
		};
	},
	onShow(){
		console.log("device page onShow");
		// 获取最新设备信息
		this.updateDevInfo();
	},
	//第一次加载时调用
	created() {
		//获取手机状态栏高度
		this.statusBarHeight = uni.getSystemInfoSync()['statusBarHeight'];
	},
	methods: {
		updateDevInfo(){
			// 从全局数据获取最新设备信息
			const globalData = getApp().globalData;
			this.deviceList = globalData.deviceList;
			if(this.deviceList.length > 0){
				console.log("设备列表更新成功，当前设备数量：", this.deviceList.length);
				return this.deviceList;
			} else {
				// 尝试从服务器获取设备列表
				uni.request({
					url: `${getApp().globalData.baseUrl}/devices/list`, // 替换为你的设备列表接口
					method: 'GET',
					header: {
						Authorization: `Bearer ${getApp().globalData.accessToken}`
					},
					success: (res) => {
						if (res.statusCode === 200 && res.data && res.data.devices) {
							this.deviceList = res.data.devices;
							console.log("从服务器获取设备列表成功，当前设备数量：", this.deviceList.length);
							console.log("设备列表数据：", this.deviceList);
							getApp().globalData.deviceList = this.deviceList; // 更新全局数据
						} else {
							console.error("从服务器获取设备列表失败，状态码：", res.statusCode);
							this.deviceList = [];
						}
					},
					fail: (err) => {
						console.error("请求设备列表失败：", err);
						this.deviceList = [];
					}
				});

				// 如果没有返回空数组
				console.log("当前没有设备");
				return [];
			}
		},
    	addDev(){
				console.log(" 添加设备 ");
		},
		toggleDeviceDetail(deviceId) {
			if (this.expandedDeviceId === deviceId) {
				this.expandedDeviceId = null;
			} else {
				this.expandedDeviceId = deviceId;
			}
		},
		getDeviceTypeLabel(deviceType) {
			const typeMap = {
				'gateway': '网关',
				'sensor': '传感器',
				'switch': '开关',
				'light': '灯',
				'camera': '摄像头',
				'air_conditioner': '空调',
				'other': '其他'
			};
			return typeMap[deviceType] || deviceType;
		},
		formatDate(dateStr) {
			if (!dateStr) return '未知';
			const date = new Date(dateStr);
			const year = date.getFullYear();
			const month = String(date.getMonth() + 1).padStart(2, '0');
			const day = String(date.getDate()).padStart(2, '0');
			const hours = String(date.getHours()).padStart(2, '0');
			const minutes = String(date.getMinutes()).padStart(2, '0');
			return `${year}-${month}-${day} ${hours}:${minutes}`;
		}
	}
};
</script>

<style>
	.devList{
		width: 660rpx;
		margin-top: 50rpx;
	}
	.devItem{
		width: 100%;
		margin-top: 20rpx;
		color: #8CA9AD;
		border-bottom: 4rpx solid #8CA9AD;
		border-right: 3rpx solid #8CA9AD;
		border-radius: 20rpx;
		overflow: hidden;
	}
	.devHeader {
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 20rpx 30rpx;
		height: 90rpx;
		box-sizing: border-box;
	}
	.devName{
		flex: 1;
	}
	.devName text{
		font-size: 36rpx;
		font-weight: 500;
	}
	.expand-icon {
		width: 40rpx;
		height: 40rpx;
		transition: transform 0.3s;
	}
	.expand-icon.expanded {
		transform: rotate(180deg);
	}
	.devDetail {
		padding: 20rpx 30rpx;
		background-color: rgba(140, 169, 173, 0.1);
		border-top: 1rpx solid #8CA9AD;
	}
	.detail-row {
		display: flex;
		justify-content: space-between;
		padding: 10rpx 0;
		border-bottom: 1rpx solid rgba(140, 169, 173, 0.3);
	}
	.detail-label {
		font-size: 28rpx;
		color: #8CA9AD;
	}
	.detail-value {
		font-size: 28rpx;
		color: #333;
	}
	.status-online {
		color: #4CAF50;
	}
	.status-offline {
		color: #999;
		margin-left: 40rpx;
		font-size: 36rpx;
		font-weight: 500;
	}
  	.addButton{
		position: fixed;
		bottom: 160rpx;
		right: 70rpx;
	}
    .addButton image{
      width: 100rpx;
      height: 100rpx;
    }
  	.footer-space {
	  height: 250rpx; /* 避免被 add按钮 遮挡 */
	}
</style>