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
		<!-- 环境数据（温度、湿度等） -->
		<view class="envData" v-if="envData.length > 0">
			<view class="envHeader">
				<view class="envTitle">环境数据</view>
				<view class="envDevice-selector">
					<picker @change="onDeviceChange" :value="selectedDeviceIndex" :range="deviceList" range-key="device_name">
						<view class="picker">{{ deviceList[selectedDeviceIndex] ? deviceList[selectedDeviceIndex].device_name : '选择设备' }}</view>
					</picker>
				</view>
			</view>
			<view class="chart-controls">
				<view class="time-selector">
					<view class="time-btn" :class="{ active: selectedTimeRange === 6 }" @click="changeTimeRange(6)">6小时</view>
					<view class="time-btn" :class="{ active: selectedTimeRange === 12 }" @click="changeTimeRange(12)">12小时</view>
					<view class="time-btn" :class="{ active: selectedTimeRange === 24 }" @click="changeTimeRange(24)">24小时</view>
				</view>
			</view>
			<view class="chart-container">
				<view class="chart-y-axis">
					<text class="y-label">{{ maxTemp }}°C</text>
					<text class="y-label">{{ ((maxTemp + minTemp) / 2).toFixed(1) }}°C</text>
					<text class="y-label">{{ minTemp }}°C</text>
				</view>
				<view class="chart-area">
					<!-- 温度折线点 -->
					<view v-for="(point, index) in tempPoints" :key="index" 
						class="chart-point temperature-point" 
						:style="{ left: point.x + '%', bottom: point.y + '%' }">
					</view>
					<!-- 湿度折线点 -->
					<view v-for="(point, index) in humidityPoints" :key="'h' + index" 
						class="chart-point humidity-point" 
						:style="{ left: point.x + '%', bottom: point.y + '%' }">
					</view>
				</view>
			</view>
			<view class="chart-legend">
				<view class="legend-item">
					<view class="legend-color temperature-color"></view>
					<text>温度</text>
				</view>
				<view class="legend-item">
					<view class="legend-color humidity-color"></view>
					<text>湿度</text>
				</view>
			</view>
			<view class="current-data">
				<view class="current-item">
					<text class="current-label">当前温度:</text>
					<text class="current-value temperature-value">{{ latestData.temperature ? latestData.temperature.toFixed(1) + '°C' : '--' }}</text>
				</view>
				<view class="current-item">
					<text class="current-label">当前湿度:</text>
					<text class="current-value humidity-value">{{ latestData.humidity ? latestData.humidity.toFixed(1) + '%' : '--' }}</text>
				</view>
			</view>
		</view>
		<view class="envData empty" v-else style="color: #8AA9AD;">
			<text style="font-size: 24rpx;font-weight: 500;">暂无环境数据</text>
		</view>

		<!-- 设备列表 -->
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
			<text style="font-size: 24rpx;font-weight: 500;">暂无设备，点击右下角 + 号添加设备</text>
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
			envData: [], // 环境数据
			selectedDeviceIndex: 0, // 选中的设备索引
			selectedTimeRange: 24, // 选中的时间范围（小时）
			tempPoints: [], // 温度折线点
			humidityPoints: [], // 湿度折线点
			maxTemp: 40, // 温度最大值
			minTemp: 0, // 温度最小值
			latestData: { temperature: null, humidity: null }, // 最新数据
		};
	},
	onShow(){
			console.log("device page onShow");
			// 获取最新设备信息
			this.updateDevInfo();
			// 设备列表加载完成后，获取环境数据（延迟执行确保设备列表已更新）
			setTimeout(() => {
				this.loadEnvData();
			}, 500);
		},
	//第一次加载时调用
	created() {
		// 获取手机状态栏高度
		this.statusBarHeight = uni.getSystemInfoSync()['statusBarHeight'];
	},
	methods: {
		updateDevInfo(){
			// 从全局数据获取最新设备信息
			const globalData = getApp().globalData;
			this.deviceList = uni.getStorageSync('deviceList') || globalData.deviceList || [];
			// 从服务器获取设备列表
			uni.request({
				url: `${getApp().globalData.baseUrl}/devices/list`,
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
						uni.setStorageSync('deviceList', this.deviceList); // 缓存设备列表
					} else {
						console.error("从服务器获取设备列表失败，状态码：", res.statusCode);
						this.deviceList = [];
					}
				},
				fail: (err) => {
					console.error("请求设备列表失败：", err);
					this.deviceList = [];
					uni.showToast({
						title: '网络错误或服务器未启动,请检查连接或前往设置页面更新后端服务ip',
						icon: 'none'
					});
				}
			});
			// 如果没有返回空数组
			console.log("当前没有设备");
			return [];
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
		},
		async loadEnvData() {
			console.log('开始加载环境数据，设备列表长度:', this.deviceList.length);
			
			if (this.deviceList.length === 0) {
				console.log('设备列表为空，跳过环境数据加载');
				this.envData = [];
				this.processChartData();
				return;
			}
			
			const device = this.deviceList[this.selectedDeviceIndex];
			if (!device) {
				console.log('选中的设备不存在，索引:', this.selectedDeviceIndex);
				this.envData = [];
				this.processChartData();
				return;
			}
			
			console.log('加载设备环境数据，设备ID:', device.id, '设备名称:', device.device_name);
			
			const token = uni.getStorageSync('access_token');
			if (!token) {
				uni.showToast({ title: '请先登录', icon: 'none' });
				this.envData = [];
				this.processChartData();
				return;
			}
			
			try {
				const baseUrl = getApp().globalData.baseUrl;
				console.log('请求环境数据URL:', `${baseUrl}/devices/history/${device.id}?hours=${this.selectedTimeRange}&limit=100`);
				
				const response = await new Promise((resolve, reject) => {
					uni.request({
						url: `${baseUrl}/devices/history/${device.id}?hours=${this.selectedTimeRange}&limit=100`,
						method: 'GET',
						header: { 'Authorization': `Bearer ${token}` },
						success: resolve,
						fail: reject
					});
				});
				
				console.log('环境数据响应状态码:', response.statusCode);
				console.log('环境数据响应数据:', response.data);
				
				if (response.statusCode === 200) {
					// 检查不同的响应格式
					if (response.data.success) {
						this.envData = response.data.data || [];
					} else if (Array.isArray(response.data)) {
						this.envData = response.data;
					} else if (response.data.data && Array.isArray(response.data.data)) {
						this.envData = response.data.data;
					} else {
						this.envData = [];
					}
					
					console.log("处理后的环境数据：", this.envData);
					this.processChartData();
				} else {
					console.error('获取环境数据失败，状态码:', response.statusCode);
					this.envData = [];
					this.processChartData();
				}
			} catch (error) {
				console.error('获取环境数据失败:', error);
				this.envData = [];
				this.processChartData();
			}
		},
		processChartData() {
			console.log('开始处理图表数据，数据长度:', this.envData.length);
			
			if (!this.envData || this.envData.length === 0) {
				this.tempPoints = [];
				this.humidityPoints = [];
				this.latestData = { temperature: null, humidity: null };
				this.minTemp = 0;
				this.maxTemp = 40;
				console.log('无环境数据，使用默认图表设置');
				return;
			}
			
			// 获取最新数据
			this.latestData = {
				temperature: this.envData[0]?.temperature,
				humidity: this.envData[0]?.humidity
			};
			
			// 计算温度范围
			const temps = this.envData.map(d => d.temperature).filter(t => t != null);
			if (temps.length > 0) {
				const tempMin = Math.min(...temps);
				const tempMax = Math.max(...temps);
				this.minTemp = Math.max(0, Math.floor(tempMin - 2)); // 确保最小值不小于0
				this.maxTemp = Math.ceil(tempMax + 2);
				console.log('温度范围:', this.minTemp, '~', this.maxTemp);
			} else {
				// 默认温度范围
				this.minTemp = 0;
				this.maxTemp = 40;
			}
			
			// 处理温度和湿度数据点
			const tempRange = this.maxTemp - this.minTemp || 1; // 防止除零
			const dataLength = this.envData.length;
			
			this.tempPoints = this.envData.map((item, index) => {
				const x = dataLength > 1 ? (index / (dataLength - 1)) * 100 : 50; // 单点居中
				const temp = item.temperature || 0;
				const y = ((temp - this.minTemp) / tempRange) * 100;
				return { x, y };
			});
			
			this.humidityPoints = this.envData.map((item, index) => {
				const x = dataLength > 1 ? (index / (dataLength - 1)) * 100 : 50; // 单点居中
				const humidity = item.humidity || 0;
				const y = (humidity / 100) * 100;
				return { x, y };
			});
			
			console.log('温度点数据:', this.tempPoints);
			console.log('湿度点数据:', this.humidityPoints);
		},
		onDeviceChange(e) {
			this.selectedDeviceIndex = e.detail.value;
			this.loadEnvData();
		},
		changeTimeRange(hours) {
			this.selectedTimeRange = hours;
			this.loadEnvData();
		},
		
	}
};
</script>

<style>
	.devList{
		width: 660rpx;
		margin-top: 50rpx;
	}
	
	/* 环境数据图表样式 */
	.envData {
		width: 560rpx;
		margin: 30rpx auto;
		border-radius: 20rpx;
		padding: 30rpx;
		box-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.1);
	}
	.envData.empty {
		text-align: center;
		color: #d5d5d5;
		padding: 60rpx;
	}
	.envHeader {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 20rpx;
	}
	.envTitle {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
	}
	.envDevice-selector .picker {
		border: 1rpx solid #ddd;
		border-radius: 10rpx;
		padding: 10rpx 20rpx;
		font-size: 26rpx;
		color: #8CA9AD;
	}
	.chart-controls {
		margin-bottom: 20rpx;
	}
	.time-selector {
		display: flex;
		gap: 15rpx;
	}
	.time-btn {
		padding: 8rpx 20rpx;
		border: 1rpx solid #8CA9AD;
		border-radius: 20rpx;
		font-size: 24rpx;
		color: #8CA9AD;
	}
	.time-btn.active {
		background: #8CA9AD;
		color: white;
	}
	.chart-container {
		display: flex;
		height: 300rpx;
		position: relative;
		margin: 20rpx 0;
	}
	.chart-y-axis {
		display: flex;
		flex-direction: column;
		justify-content: space-between;
		padding: 10rpx 0;
		margin-right: 10rpx;
	}
	.y-label {
		font-size: 20rpx;
		color: #999;
	}
	.chart-area {
		flex: 1;
		position: relative;
		background: linear-gradient(to top, rgba(140, 169, 173, 0.1), transparent);
		border-left: 1rpx solid #eee;
		border-bottom: 1rpx solid #eee;
	}
	.chart-line {
		position: absolute;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
	}
	.chart-point {
		position: absolute;
		width: 12rpx;
		height: 12rpx;
		border-radius: 50%;
		transform: translate(-50%, 50%);
		z-index: 10;
	}
	.temperature-point {
		background: #FF6B6B;
		border: 2rpx solid white;
		box-shadow: 0 2rpx 4rpx rgba(0,0,0,0.2);
	}
	.humidity-point {
		background: #4ECDC4;
		border: 2rpx solid white;
		box-shadow: 0 2rpx 4rpx rgba(0,0,0,0.2);
	}
	.line-segment {
		position: absolute;
		height: 3rpx;
		background: #FF6B6B;
		transform-origin: left center;
	}
	.chart-legend {
		display: flex;
		justify-content: center;
		gap: 40rpx;
		margin-top: 20rpx;
	}
	.legend-item {
		display: flex;
		align-items: center;
		gap: 10rpx;
		font-size: 24rpx;
		color: #666;
	}
	.legend-color {
		width: 20rpx;
		height: 20rpx;
		border-radius: 50%;
	}
	.temperature-color {
		background: #FF6B6B;
	}
	.humidity-color {
		background: #4ECDC4;
	}
	.current-data {
		display: flex;
		justify-content: space-around;
		margin-top: 30rpx;
		padding-top: 20rpx;
		border-top: 1rpx solid #eee;
	}
	.current-item {
		display: flex;
		flex-direction: column;
		align-items: center;
	}
	.current-label {
		font-size: 24rpx;
		color: #999;
	}
	.current-value {
		font-size: 36rpx;
		font-weight: bold;
		margin-top: 10rpx;
	}
	.temperature-value {
		color: #FF6B6B;
	}
	.humidity-value {
		color: #4ECDC4;
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