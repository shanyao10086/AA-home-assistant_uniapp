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
				<text class="navBarTitle">运动记录</text>
			</view>
		</view>
		
		
		<view class="sportRecordBox">
			<!-- 记录列表 -->
			<view class="sportRecordList" v-if="sportRecord.length > 0" v-scroll-y>
				<view class="sportRecordItem" v-for="item in sportRecord" :key="item.id">
					<view class="sportRecordItemTitle" style="font-size: 24px; font-weight: bold;">
						<text>{{ item.sport_type }}</text>
					</view>
					<view class="sportRecordItemDistance" style="font-size: 20px;">
						<text>运动时间：{{ item.duration }}</text>
					</view>
					<view class="sportRecordItemTime" style="font-size: 20px;">
						<text>创建时间：{{ item.created_at }}</text>
					</view>
				</view>
			</view>
			<view class="noRecord" v-else style="font-size: 20px; margin: 50px 0;">
				<text>暂无运动记录，点击下方按钮添加</text>
			</view>
		</view>
		<view style="width: 40%;">
			<button @click="showCustomModal()">添加运动记录</button>
		</view>
		
		<!-- 自定义弹窗 -->
		<view class="modal-mask" v-if="showModal" @click="hideCustomModal">
			<view class="modal-content" @click.stop>
				<view class="modal-header">
					<text class="modal-title">添加运动记录</text>
					<view class="modal-close" @click="hideCustomModal">×</view>
				</view>
				<view class="modal-body">
					<view class="input-group">
						<text class="input-label">运动类型</text>
						<picker @change="onSportTypeChange" :value="selectedSportTypeIndex" :range="sportTypes" range-key="name">
							<view class="picker-input">{{ selectedSportType.name || '请选择运动类型' }}</view>
						</picker>
					</view>
					<view class="input-group">
						<text class="input-label">运动时长</text>
						<view class="duration-picker-container">
							<view class="picker-column">
								<text class="picker-label">小时</text>
								<picker-view class="duration-picker" :value="durationHoursIndex" @change="onDurationHoursChange" indicator-style="height: 50px;">
									<picker-view-column>
										<view class="picker-item" v-for="(item, index) in hoursRange" :key="index">{{ item }}</view>
									</picker-view-column>
								</picker-view>
							</view>
							<view class="picker-column">
								<text class="picker-label">分钟</text>
								<picker-view class="duration-picker" :value="durationMinutesIndex" @change="onDurationMinutesChange" indicator-style="height: 50px;">
									<picker-view-column>
										<view class="picker-item" v-for="(item, index) in minutesRange" :key="index">{{ item }}</view>
									</picker-view-column>
								</picker-view>
							</view>
						</view>
						<text class="duration-display">总时长：{{ totalDuration }}分钟</text>
					</view>
					<view class="input-group" v-if="selectedSportTypeIndex === sportTypes.length - 1">
						<text class="input-label">自定义类型</text>
						<input class="custom-input" v-model="customSportType" placeholder="请输入自定义运动类型" maxlength="20" />
					</view>
				</view>
				<view class="modal-footer">
					<button class="modal-btn cancel" @click="hideCustomModal">取消</button>
					<button class="modal-btn confirm" @click="confirmAddRecord">确定</button>
				</view>
			</view>
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
			baseUrl: '', // 服务器ip地址
			accessToken: '', // 访问令牌
			// 记录列表
			sportRecord: [],
			// 自定义弹窗相关数据
			showModal: false, // 是否显示弹窗
			sportTypes: [
				{ name: '跑步', value: 'running' },
				{ name: '游泳', value: 'swimming' },
				{ name: '骑行', value: 'cycling' },
				{ name: '健身', value: 'fitness' },
				{ name: '其他', value: 'other' }
			], // 运动类型列表
			selectedSportTypeIndex: 0, // 选中的运动类型索引
			durationHoursIndex: 0, // 小时选择索引
			durationMinutesIndex: 0, // 分钟选择索引
			customSportType: '', // 自定义运动类型
			// 时长选择范围
			hoursRange: Array.from({length: 25}, (_, i) => i.toString().padStart(2, '0')), // 0-24小时
			minutesRange: Array.from({length: 60}, (_, i) => i.toString().padStart(2, '0')) // 0-59分钟
		};
	},
	//第一次加载时调用
	created() {
		//获取手机状态栏高度
		this.statusBarHeight = uni.getSystemInfoSync()['statusBarHeight'];
		this.baseUrl = getApp().globalData.baseUrl;
		this.accessToken = uni.getStorageSync('access_token');
	},
	onShow() {
		// 从服务器获取运动记录列表
		this.getSportRecord();
	},
	computed: {
		// 计算总时长（分钟）
		totalDuration() {
			// 确保数组已初始化
			if (!this.hoursRange || !this.minutesRange) {
				return 0;
			}
			
			// 确保索引在有效范围内
			const hoursIndex = Math.max(0, Math.min(this.durationHoursIndex, this.hoursRange.length - 1));
			const minutesIndex = Math.max(0, Math.min(this.durationMinutesIndex, this.minutesRange.length - 1));
			
			const hours = parseInt(this.hoursRange[hoursIndex] || '0');
			const minutes = parseInt(this.minutesRange[minutesIndex] || '0');
			return hours * 60 + minutes;
		},
		
		// 获取选中的运动类型
		selectedSportType() {
			// 确保数组已初始化
			if (!this.sportTypes) {
				return { name: '请选择' };
			}
			
			const index = Math.max(0, Math.min(this.selectedSportTypeIndex, this.sportTypes.length - 1));
			return this.sportTypes[index] || {};
		}
	},
	methods: {
		// 返回上一页
		goBack() {
			uni.navigateBack({ delta: 1 });
		},
		// 从服务器获取运动记录列表
		getSportRecord() {
			uni.request({
				url: this.baseUrl + '/sports/get_records',
				method: 'GET',
				header: {
              		'Content-Type': 'application/json',
					'Authorization': `Bearer ${this.accessToken}`
				},
				success: (res) => {
					if (res.statusCode === 200) {
						this.sportRecord = res.data;
						console.log('获取运动记录成功:',JSON.stringify(this.sportRecord));
					} else {
						console.log('状态码:',res.statusCode,'原始响应数据:',JSON.stringify(res.data));
						uni.showToast({
							title: res.data,
							icon: 'none'
						});
					}
				},
				fail: function (error) {
					uni.showToast({
						title: '获取运动记录失败',
						icon: 'none'
					});
					console.log('获取运动记录失败:', error);
				}
			});
		},
		// 显示自定义弹窗
		showCustomModal() {
			const token = uni.getStorageSync('access_token');
			if (!token) {
				uni.showToast({
					title: '请先登录',
					icon: 'none'
				});
				return;
			}
			
			// 重置表单数据
			this.selectedSportTypeIndex = 0;
			this.durationHoursIndex = 0;
			this.durationMinutesIndex = 0;
			this.customSportType = '';
			this.showModal = true;
		},
		
		// 隐藏自定义弹窗
		hideCustomModal() {
			this.showModal = false;
		},
		
		// 运动类型选择变化
		onSportTypeChange(e) {
			this.selectedSportTypeIndex = e.detail.value;
		},
		
		// 小时选择变化
		onDurationHoursChange(e) {
			this.durationHoursIndex = e.detail.value[0];
		},
		
		// 分钟选择变化
		onDurationMinutesChange(e) {
			this.durationMinutesIndex = e.detail.value[0];
		},
		
		// 确认添加运动记录
		async confirmAddRecord() {
			// 验证总时长
			if (this.totalDuration === 0) {
				uni.showToast({
					title: '请选择运动时长',
					icon: 'none'
				});
				return;
			}
			
			// 获取运动类型名称
			let sportTypeName = this.selectedSportType.name;
			if (this.selectedSportTypeIndex === this.sportTypes.length - 1) {
				// 如果是"其他"类型，使用自定义输入
				if (!this.customSportType.trim()) {
					uni.showToast({
						title: '请输入自定义运动类型',
						icon: 'none'
					});
					return;
				}
				sportTypeName = this.customSportType.trim();
				if (sportTypeName.length > 20) {
					uni.showToast({
						title: '运动类型长度不能超过20字符',
						icon: 'none'
					});
					return;
				}
			}
			
			// 调用API添加记录
			await this.addRecord(sportTypeName, this.totalDuration);
		},
		
		// 添加运动记录到服务器
		async addRecord(sportType, duration) {
			uni.showLoading({
				title: '添加中...'
			});
			
			try {
				const res = await uni.request({
					url: this.baseUrl + '/sports/add_record',
					method: 'POST',
					header: {
						'Content-Type': 'application/json',
						'Authorization': `Bearer ${this.accessToken}`
					},
					data: {
						sport_type: sportType,
						duration: duration
					}
				});
				
				uni.hideLoading();
				
				if (res[1].statusCode === 200) {
					uni.showToast({
						title: '添加成功',
						icon: 'success'
					});
					this.hideCustomModal();
					// 刷新记录列表
					this.getSportRecord();
				} else {
					uni.showToast({
						title: res[1].data || '添加失败',
						icon: 'none'
					});
				}
			} catch (error) {
				uni.hideLoading();
				uni.showToast({
					title: '网络错误，请重试',
					icon: 'none'
				});
				console.error('添加运动记录失败:', error);
			}
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
	.sportRecordList{
		width: 100%;
		max-height: 80vh;
		margin: 20px 0;
	}
	.sportRecordItem{
		width: 90%;
		border: 0 1rpx 0 2rpx solid #8CA9AD;
		padding: 5rpx 10rpx;
		margin: 10px 0;
	}
	
	button {
		width: 100%;
		color: #FBF2E3;
		background-color: #8CA9AD;
		border-radius: 20rpx;
	}
	
	/* 自定义弹窗样式 */
	.modal-mask {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		background-color: rgba(0, 0, 0, 0.5);
		display: flex;
		justify-content: center;
		align-items: center;
		z-index: 999;
	}
	
	.modal-content {
		background-color: #fff;
		border-radius: 20rpx;
		width: 80%;
		max-width: 600rpx;
		padding: 40rpx;
		box-shadow: 0 10rpx 30rpx rgba(0, 0, 0, 0.3);
	}
	
	.modal-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 40rpx;
		border-bottom: 2rpx solid #f0f0f0;
		padding-bottom: 20rpx;
	}
	
	.modal-title {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
	}
	
	.modal-close {
		font-size: 40rpx;
		color: #999;
		width: 60rpx;
		height: 60rpx;
		display: flex;
		justify-content: center;
		align-items: center;
		border-radius: 50%;
		background-color: #f0f0f0;
	}
	
	.modal-body {
		margin-bottom: 40rpx;
	}
	
	.input-group {
		margin-bottom: 30rpx;
	}
	
	.input-label {
		display: block;
		font-size: 28rpx;
		color: #666;
		margin-bottom: 15rpx;
	}
	
	.picker-input {
		border: 2rpx solid #ddd;
		border-radius: 10rpx;
		padding: 20rpx;
		font-size: 28rpx;
		background-color: #f9f9f9;
	}
	
	.duration-picker-container {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 15rpx;
	}
	
	.picker-column {
		flex: 1;
		display: flex;
		flex-direction: column;
		align-items: center;
		margin: 0 10rpx;
	}
	
	.picker-label {
		font-size: 24rpx;
		color: #999;
		margin-bottom: 10rpx;
	}
	
	.duration-picker {
		height: 200rpx;
		width: 100%;
		background-color: #f9f9f9;
		border-radius: 10rpx;
	}
	
	.picker-item {
		height: 50rpx;
		line-height: 50rpx;
		text-align: center;
		font-size: 28rpx;
		color: #333;
	}
	
	.duration-display {
		display: block;
		font-size: 26rpx;
		color: #8CA9AD;
		text-align: center;
		margin-top: 10rpx;
	}
	
	.custom-input {
		border: 2rpx solid #ddd;
		border-radius: 10rpx;
		padding: 20rpx;
		font-size: 28rpx;
		width: 100%;
		box-sizing: border-box;
	}
	
	.modal-footer {
		display: flex;
		justify-content: space-between;
		gap: 20rpx;
	}
	
	.modal-btn {
		flex: 1;
		padding: 20rpx;
		border-radius: 10rpx;
		font-size: 28rpx;
		border: none;
	}
	
	.modal-btn.cancel {
		background-color: #f0f0f0;
		color: #666;
	}
	
	.modal-btn.confirm {
		background-color: #8CA9AD;
		color: #fff;
	}
</style>