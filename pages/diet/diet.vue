<template>
	<view class="container">
		<!-- 自定义导航栏 -->
		<view class="navBarBox">
			<!-- 状态栏占位 -->
			<view class="statusBar" :style="{ paddingTop: statusBarHeight + 'px' }" style="background-color: #8CA9AD;"></view>
			<!-- 真正的导航栏内容 -->
			<view class="navBar">
				<view class="escp" @click="goBack()">
					<image class="escimg"
					src="/static/icons/向左.png"></image>
				</view>
				<text class="navBarTitle">饮食建议</text>
			</view>
		</view>
		
		<view class="content">
			<!-- 今日饮食总结 -->
			<view class="summary-card">
				<view class="summary-title">今日饮食总结</view>
				<view class="summary-info" v-if="dailySummary">
					<view class="summary-item">
						<text class="label">总热量:</text>
						<text class="value">{{ dailySummary.total_calories }} / {{ dailySummary.recommended_calories }} 千卡</text>
					</view>
					<view class="summary-item">
						<text class="label">蛋白质:</text>
						<text class="value">{{ dailySummary.total_protein }} 克</text>
					</view>
					<view class="summary-item">
						<text class="label">碳水化合物:</text>
						<text class="value">{{ dailySummary.total_carbohydrates }} 克</text>
					</view>
					<view class="summary-item">
						<text class="label">脂肪:</text>
						<text class="value">{{ dailySummary.total_fat }} 克</text>
					</view>
				</view>
				<view class="summary-info" v-else>
					<text>暂无今日饮食记录</text>
				</view>
			</view>
			
			
			<!-- 添加饮食记录按钮 -->
			<button class="add-record-btn" @click="showAddRecordForm = true">+ 添加饮食记录</button>
			
			<!-- 饮食记录列表 -->
			<view class="records-section">
				<view class="section-header">
					<view class="section-title">饮食记录</view>
					<button class="refresh-btn" @click="loadRecords">刷新</button>
				</view>
				
				<view v-if="loadingRecords" class="loading">加载中...</view>
				<view v-else-if="records.length === 0" class="empty">暂无饮食记录</view>
				<view v-else class="records-list">
					<view v-for="record in records" :key="record.id" class="record-item">
						<view class="record-header">
							<text class="meal-time">{{ getMealTimeLabel(record.meal_time) }}</text>
							<text class="date">{{ record.date }}</text>
						</view>
						<view class="record-body">
							<view class="food-info">
								<text class="food-name">{{ record.food_name }}</text>
								<text class="weight">{{ record.weight }}g</text>
							</view>
							<view class="nutrition-info">
								<text class="calories">{{ record.calories }}千卡</text>
								<text class="protein">蛋白: {{ record.protein }}g</text>
								<text class="carbs">碳水: {{ record.carbohydrates }}g</text>
								<text class="fat">脂肪: {{ record.fat }}g</text>
							</view>
						</view>
						<view class="record-actions">
							<button class="edit-btn" @click="editRecord(record)">编辑</button>
							<button class="delete-btn" @click="deleteRecord(record.id)">删除</button>
						</view>
					</view>
				</view>
			</view>
		</view>
		
		<!-- 添加/编辑记录弹窗 -->
		<view v-if="showAddRecordForm" class="modal-overlay" @click="closeForm">
			<view class="modal-content" @click.stop>
				<view class="modal-header">
					<text class="modal-title">{{ editingRecord ? '编辑饮食记录' : '添加饮食记录' }}</text>
					<text class="modal-close" @click="closeForm">×</text>
				</view>
				
				<view class="form-content">
					<view class="form-group">
						<text class="form-label">日期</text>
						<input class="form-input" v-model="formData.date" type="date" placeholder="请选择日期" />
					</view>
					
					<view class="form-group">
						<text class="form-label">用餐时间</text>
						<picker @change="onMealTimeChange" :value="selectedMealTimeIndex" :range="mealTimeOptions" range-key="label">
							<view class="picker">{{ formData.meal_time ? getMealTimeLabel(formData.meal_time) : '请选择用餐时间' }}</view>
						</picker>
					</view>
					
					<view class="form-group">
						<text class="form-label">食物名称</text>
						<input class="form-input" v-model="formData.food_name" placeholder="请输入食物名称" />
					</view>
					
					<view class="form-group">
						<text class="form-label">重量(g)</text>
						<input class="form-input" v-model.number="formData.weight" type="number" placeholder="请输入重量" />
					</view>
					
					<view class="form-group">
						<text class="form-label">蛋白质(g)</text>
						<input class="form-input" v-model.number="formData.protein" type="number" placeholder="请输入蛋白质含量" />
					</view>
					
					<view class="form-group">
						<text class="form-label">碳水化合物(g)</text>
						<input class="form-input" v-model.number="formData.carbohydrates" type="number" placeholder="请输入碳水化合物含量" />
					</view>
					
					<view class="form-group">
						<text class="form-label">脂肪(g)</text>
						<input class="form-input" v-model.number="formData.fat" type="number" placeholder="请输入脂肪含量" />
					</view>
					
					<view class="form-actions">
						<button class="cancel-btn" @click="closeForm">取消</button>
						<button class="confirm-btn" @click="saveRecord">{{ editingRecord ? '更新' : '保存' }}</button>
					</view>
				</view>
			</view>
		</view>
		
		<!-- AI饮食建议卡片 -->
		<view class="suggestion-card">
			<view class="suggestion-header">
				<view class="suggestion-title">饮食建议</view>
				<view class="suggestion-controls">
					<view class="suggestion-period-selector">
						<picker @change="onSuggestionPeriodChange" :value="selectedSuggestionPeriodIndex" :range="suggestionPeriodOptions" range-key="label">
							<view class="picker">{{ suggestionPeriodOptions[selectedSuggestionPeriodIndex].label }}</view>
						</picker>
					</view>
					<button class="refresh-suggestion-btn" @click="getAISuggestion" :disabled="loadingSuggestion">
						{{ loadingSuggestion ? '获取中...' : '刷新' }}
					</button>
				</view>
			</view>
			<view v-if="loadingSuggestion" class="suggestion-loading">正在获取智能饮食建议...</view>
			<view v-else-if="suggestionData" class="suggestion-content">
				<text class="suggestion-text">{{ suggestionData.suggestion }}</text>
				<view v-if="suggestionData.summary && Object.keys(suggestionData.summary).length > 0" class="suggestion-summary">
					<view class="summary-section-title">营养概览</view>
					<view class="summary-detail">
						<view class="summary-row" v-if="suggestionData.summary.total_calories !== undefined">
							<text class="summary-label">总热量:</text>
							<text class="summary-value">{{ suggestionData.summary.total_calories }} 千卡</text>
						</view>
						<view class="summary-row" v-if="suggestionData.summary.total_protein !== undefined">
							<text class="summary-label">蛋白质:</text>
							<text class="summary-value">{{ suggestionData.summary.total_protein }} 克</text>
						</view>
						<view class="summary-row" v-if="suggestionData.summary.total_carbohydrates !== undefined">
							<text class="summary-label">碳水:</text>
							<text class="summary-value">{{ suggestionData.summary.total_carbohydrates }} 克</text>
						</view>
						<view class="summary-row" v-if="suggestionData.summary.total_fat !== undefined">
							<text class="summary-label">脂肪:</text>
							<text class="summary-value">{{ suggestionData.summary.total_fat }} 克</text>
						</view>
						<view class="summary-row" v-if="suggestionData.summary.most_common_foods && suggestionData.summary.most_common_foods.length > 0">
							<text class="summary-label">常吃食物:</text>
							<text class="summary-value">{{ suggestionData.summary.most_common_foods.slice(0, 3).map(item => item[0]).join(', ') }}</text>
						</view>
					</view>
				</view>
				<view class="suggestion-period small">（来自AI生成，仅供参考，不构成专业建议）</view>
			</view>
			<view v-else class="suggestion-empty">点击刷新获取智能饮食建议</view>
		</view>
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

			records: [], // 饮食记录列表
			dailySummary: null, // 今日饮食总结
			loadingRecords: false,
			showAddRecordForm: false,
			editingRecord: null,
			mealTimeOptions: [], // 用餐时间选项
			formData: {
				date: this.getCurrentDate(),
				meal_time: '',
				food_name: '',
				weight: 0,
				protein: 0,
				carbohydrates: 0,
				fat: 0
			},
			selectedMealTimeIndex: 0,
			suggestionData: null, // AI饮食建议数据
			loadingSuggestion: false, // 是否正在加载建议
			suggestionPeriodOptions: [ // 建议时间段选项
				{ label: '今天', value: 'today' },
				{ label: '最近三天', value: '3days' },
				{ label: '最近一周', value: 'week' }
			],
			selectedSuggestionPeriodIndex: 0 // 默认选择今天
		}
	},
	
	onLoad() {
		this.initPage();
	},
	//第一次加载时调用
	created() {
		//获取手机状态栏高度
		this.statusBarHeight = uni.getSystemInfoSync()['statusBarHeight'];
	},
	
	methods: {
		// 初始化页面
		async initPage() {
			await this.loadMealTimeOptions();
			await this.loadDailySummary();
			await this.loadRecords();
			await this.getAISuggestion(); // 加载AI饮食建议
		},
		
		// 将数组转换为字符串（用于显示最常见食物）
		formatMostCommonFoods(foods) {
			if (!foods || foods.length === 0) return '无';
			return foods.slice(0, 3).map(item => item[0]).join(', ');
		},
		
		// 获取当前日期
		getCurrentDate() {
			const now = new Date();
			return now.toISOString().split('T')[0];
		},
		
		// 加载用餐时间选项
		async loadMealTimeOptions() {
			try {
				const token = uni.getStorageSync('access_token');
				if (!token) {
					uni.showToast({
						title: '请先登录',
						icon: 'none'
					});
					return;
				}
				
				const baseUrl = getApp().globalData.baseUrl;
				const response = await new Promise((resolve, reject) => {
					uni.request({
						url: `${baseUrl}/diets/meal_times`,
						method: 'GET',
						header: {
							'Authorization': `Bearer ${token}`
						},
						success: resolve,
						fail: reject
					});
				});
				
				if (response.statusCode === 200 && response.data.success) {
					this.mealTimeOptions = response.data.data;
				} else {
					console.error('获取用餐时间选项失败:', response.data);
				}
			} catch (error) {
				console.error('加载用餐时间选项失败:', error);
			}
		},
		
		// 获取AI饮食建议
		async getAISuggestion() {
			const token = uni.getStorageSync('access_token');
			if (!token) {
				uni.showToast({
					title: '请先登录',
					icon: 'none'
				});
				return;
			}
			
			this.loadingSuggestion = true;
			
			try {
				const baseUrl = getApp().globalData.baseUrl;
				const period = this.suggestionPeriodOptions[this.selectedSuggestionPeriodIndex].value;
				
				const response = await new Promise((resolve, reject) => {
					uni.request({
						url: `${baseUrl}/diets/ai_suggestion?period=${period}`,
						method: 'GET',
						header: {
							'Authorization': `Bearer ${token}`
						},
						success: resolve,
						fail: reject
					});
				});
				
				if (response.statusCode === 200 && response.data.success) {
					this.suggestionData = response.data.data;
				} else {
					console.error('获取AI饮食建议失败:', response.data);
					uni.showToast({
						title: response.data.message || '获取建议失败',
						icon: 'none'
					});
				}
			} catch (error) {
				console.error('获取AI饮食建议失败:', error);
				uni.showToast({
					title: '网络错误',
					icon: 'none'
				});
			} finally {
				this.loadingSuggestion = false;
			}
		},
		
		// 选择建议时间段
		onSuggestionPeriodChange(e) {
			const index = e.detail.value;
			this.selectedSuggestionPeriodIndex = index;
			// 选择时间段后自动获取建议
			this.getAISuggestion();
		},
		
		// 加载今日饮食总结
		async loadDailySummary() {
			try {
				const token = uni.getStorageSync('access_token');
				if (!token) {
					uni.showToast({
						title: '请先登录',
						icon: 'none'
					});
					return;
				}
				
				const baseUrl = getApp().globalData.baseUrl;
				const response = await new Promise((resolve, reject) => {
					uni.request({
						url: `${baseUrl}/diets/daily_summary`,
						method: 'GET',
						header: {
							'Authorization': `Bearer ${token}`
						},
						success: resolve,
						fail: reject
					});
				});
				
				if (response.statusCode === 200 && response.data.success) {
					this.dailySummary = response.data.data;
				} else {
					console.error('获取饮食总结失败:', response.data);
					this.dailySummary = null;
				}
			} catch (error) {
				console.error('加载饮食总结失败:', error);
				this.dailySummary = null;
			}
		},
		
		// 加载饮食记录
		async loadRecords() {
			this.loadingRecords = true;
			
			try {
				const token = uni.getStorageSync('access_token');
				if (!token) {
					uni.showToast({
						title: '请先登录',
						icon: 'none'
					});
					return;
				}
				
				const baseUrl = getApp().globalData.baseUrl;
				const response = await new Promise((resolve, reject) => {
					uni.request({
						url: `${baseUrl}/diets/record_get_all/`,
						method: 'GET',
						header: {
							'Authorization': `Bearer ${token}`
						},
						data: {
							date: this.getCurrentDate(),
							days: 7
						},
						success: resolve,
						fail: reject
					});
				});
				
				if (response.statusCode === 200 && response.data.success) {
					this.records = response.data.data;
				} else {
					console.error('获取饮食记录失败:', response.data);
					this.records = [];
				}
			} catch (error) {
				console.error('加载饮食记录失败:', error);
				this.records = [];
			} finally {
				this.loadingRecords = false;
			}
		},
		
		// 获取用餐时间标签
		getMealTimeLabel(mealTimeValue) {
			const option = this.mealTimeOptions.find(opt => opt.value === mealTimeValue);
			return option ? option.label : mealTimeValue;
		},
		
		// 用餐时间选择变化
		onMealTimeChange(e) {
			const index = e.detail.value;
			this.selectedMealTimeIndex = index;
			this.formData.meal_time = this.mealTimeOptions[index].value;
		},
		
		// 显示添加记录表单
		showAddRecord() {
			this.resetFormData();
			this.showAddRecordForm = true;
			this.editingRecord = null;
		},
		
		// 编辑记录
		editRecord(record) {
			this.editingRecord = record;
			this.formData = {
				date: record.date,
				meal_time: record.meal_time,
				food_name: record.food_name,
				weight: record.weight,
				protein: record.protein,
				carbohydrates: record.carbohydrates,
				fat: record.fat
			};
			
			// 设置选中的用餐时间索引
			const index = this.mealTimeOptions.findIndex(opt => opt.value === record.meal_time);
			this.selectedMealTimeIndex = index >= 0 ? index : 0;
			
			this.showAddRecordForm = true;
		},
		
		// 重置表单数据
		resetFormData() {
			this.formData = {
				date: this.getCurrentDate(),
				meal_time: '',
				food_name: '',
				weight: 0,
				protein: 0,
				carbohydrates: 0,
				fat: 0
			};
			this.selectedMealTimeIndex = 0;
		},
		
		// 关闭表单
		closeForm() {
			this.showAddRecordForm = false;
			this.editingRecord = null;
			this.resetFormData();
		},
		
		// 保存记录（创建或更新）
		async saveRecord() {
			if (!this.validateFormData()) {
				return;
			}
			
			const token = uni.getStorageSync('access_token');
			if (!token) {
				uni.showToast({
					title: '请先登录',
					icon: 'none'
				});
				return;
			}
			
			try {
				const baseUrl = getApp().globalData.baseUrl;
				let response;
				
				if (this.editingRecord) {
					// 更新记录
					response = await new Promise((resolve, reject) => {
						uni.request({
							url: `${baseUrl}/diets/record_update/${this.editingRecord.id}`,
							method: 'PUT',
							header: {
								'Authorization': `Bearer ${token}`,
								'Content-Type': 'application/json'
							},
							data: this.formData,
							success: resolve,
							fail: reject
						});
					});
				} else {
					// 创建记录
					response = await new Promise((resolve, reject) => {
						uni.request({
							url: `${baseUrl}/diets/record_create/`,
							method: 'POST',
							header: {
								'Authorization': `Bearer ${token}`,
								'Content-Type': 'application/json'
							},
							data: this.formData,
							success: resolve,
							fail: reject
						});
					});
				}
				
				if ((this.editingRecord && response.statusCode === 200 && response.data.success) ||
				    (!this.editingRecord && response.statusCode === 201 && response.data.success)) {
					uni.showToast({
						title: this.editingRecord ? '记录更新成功' : '记录创建成功',
						icon: 'success'
					});
					
					// 关闭表单并刷新数据
					this.closeForm();
					await this.loadDailySummary();
					await this.loadRecords();
				} else {
					console.error(this.editingRecord ? '更新记录失败:' : '创建记录失败:', response.data);
					uni.showToast({
						title: response.data.message || (this.editingRecord ? '更新失败' : '创建失败'),
						icon: 'none'
					});
				}
			} catch (error) {
				console.error('保存记录失败:', error);
				uni.showToast({
					title: '网络错误',
					icon: 'none'
				});
			}
		},
		
		// 验证表单数据
		validateFormData() {
			if (!this.formData.date) {
				uni.showToast({
					title: '请选择日期',
					icon: 'none'
				});
				return false;
			}
			
			if (!this.formData.meal_time) {
				uni.showToast({
					title: '请选择用餐时间',
					icon: 'none'
				});
				return false;
			}
			
			if (!this.formData.food_name.trim()) {
				uni.showToast({
					title: '请输入食物名称',
					icon: 'none'
				});
				return false;
			}
			
			if (this.formData.weight <= 0) {
				uni.showToast({
					title: '请输入正确的重量',
					icon: 'none'
				});
				return false;
			}
			
			return true;
		},
		
		// 删除记录
		async deleteRecord(recordId) {
			const result = await new Promise((resolve) => {
				uni.showModal({
					title: '确认删除',
					content: '确定要删除这条饮食记录吗？',
					success: resolve
				});
			});
			
			if (!result.confirm) {
				return;
			}
			
			const token = uni.getStorageSync('access_token');
			if (!token) {
				uni.showToast({
					title: '请先登录',
					icon: 'none'
				});
				return;
			}
			
			try {
				const baseUrl = getApp().globalData.baseUrl;
				const response = await new Promise((resolve, reject) => {
					uni.request({
						url: `${baseUrl}/diets/record_delete/${recordId}`,
						method: 'DELETE',
						header: {
							'Authorization': `Bearer ${token}`
						},
						success: resolve,
						fail: reject
					});
				});
				
				if (response.statusCode === 200 && response.data.success) {
					uni.showToast({
						title: '记录删除成功',
						icon: 'success'
					});
					
					// 刷新数据
					await this.loadDailySummary();
					await this.loadRecords();
				} else {
					console.error('删除记录失败:', response.data);
					uni.showToast({
						title: response.data.message || '删除失败',
						icon: 'none'
					});
				}
			} catch (error) {
				console.error('删除记录失败:', error);
				uni.showToast({
					title: '网络错误',
					icon: 'none'
				});
			}
		},
		
		goBack() {
			uni.navigateBack();
		}
	}
}
</script>

<style>
.container {
	padding: 0;
	min-height: 100vh;
}

.content{
	margin: 20rpx;
}

.back-button {
	left: 20rpx;
	position: absolute;
	font-size: 32rpx;
	color: #8CA9AD;
	padding: 10rpx;
	cursor: pointer;
}

.title {
	font-size: 36rpx;
	font-weight: bold;
	color: #333;
}

.summary-card {
	background: white;
	border-radius: 20rpx;
	padding: 30rpx;
	margin-bottom: 20rpx;
	box-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.1);
}

.summary-title {
	font-size: 32rpx;
	font-weight: bold;
	color: #333;
	margin-bottom: 20rpx;
}

.summary-info {
	display: flex;
	flex-direction: column;
	gap: 15rpx;
}

.summary-item {
	display: flex;
	justify-content: space-between;
	align-items: center;
	padding: 10rpx 0;
	border-bottom: 1rpx solid #eee;
}

.label {
	font-size: 28rpx;
	color: #666;
}

.value {
	font-size: 28rpx;
	color: #333;
	font-weight: 500;
}

/* AI饮食建议卡片 */
.suggestion-card {
	background: white;
	border-radius: 20rpx;
	padding: 30rpx;
	margin: 20rpx;
	box-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.1);
}

.suggestion-header {
	display: flex;
	justify-content: space-between;
	align-items: center;
	margin-bottom: 20rpx;
}

.suggestion-controls {
	display: flex;
	align-items: center;
	gap: 15rpx;
}

.suggestion-title {
	font-size: 32rpx;
	font-weight: bold;
	color: #333;
}

.suggestion-period-selector {
	display: flex;
	align-items: center;
	gap: 10rpx;
}

.suggestion-period.small {
	font-size: 24rpx;
	color: #999;
}
.suggestion-period-selector .picker {
	border: 1rpx solid #ddd;
	border-radius: 10rpx;
	padding: 8rpx 15rpx;
	font-size: 24rpx;
	background: #f9f9f9;
}

.refresh-suggestion-btn {
	background: #8CA9AD;
	color: white;
	border: none;
	border-radius: 15rpx;
	padding: 8rpx 15rpx;
	font-size: 24rpx;
	margin-left: 10rpx;
}

.refresh-suggestion-btn:disabled {
	background: #cccccc;
}

.suggestion-loading, .suggestion-empty {
	text-align: center;
	padding: 30rpx;
	color: #999;
	font-size: 28rpx;
}

.suggestion-content {
	display: flex;
	flex-direction: column;
	gap: 15rpx;
}

.suggestion-text {
	font-size: 28rpx;
	color: #333;
	line-height: 1.6;
}

.suggestion-summary {
	margin-top: 15rpx;
	background: #f9f9f9;
	border-radius: 15rpx;
	padding: 20rpx;
}

.summary-section-title {
	font-size: 28rpx;
	font-weight: bold;
	color: #333;
	margin-bottom: 10rpx;
}

.summary-detail {
	display: flex;
	flex-direction: column;
	gap: 8rpx;
}

.summary-row {
	display: flex;
	justify-content: space-between;
	align-items: center;
	padding: 5rpx 0;
}

.summary-label {
	font-size: 24rpx;
	color: #666;
}

.summary-value {
	font-size: 24rpx;
	color: #333;
	font-weight: 500;
}

.add-record-btn {
	width: 100%;
	background: #8CA9AD;
	color: white;
	border: none;
	border-radius: 20rpx;
	padding: 20rpx;
	font-size: 32rpx;
	font-weight: 500;
	margin-bottom: 20rpx;
}

.records-section {
	background: white;
	border-radius: 20rpx;
	padding: 30rpx;
	box-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.1);
	width: 100%;
	box-sizing: border-box;
}

.section-header {
	display: flex;
	justify-content: space-between;
	align-items: center;
	margin-bottom: 20rpx;
}

.section-title {
	font-size: 32rpx;
	font-weight: bold;
	color: #333;
	flex: 1;
}

.refresh-btn {
	background: #8CA9AD;
	color: white;
	border: none;
	border-radius: 15rpx;
	padding: 10rpx 20rpx;
	font-size: 24rpx;
}

.loading, .empty {
	text-align: center;
	padding: 40rpx;
	color: #999;
	font-size: 28rpx;
}

.records-list {
	display: flex;
	flex-direction: column;
	gap: 20rpx;
}

.record-item {
	border: 1rpx solid #eee;
	border-radius: 15rpx;
	padding: 20rpx;
}

.record-header {
	display: flex;
	justify-content: space-between;
	align-items: center;
	margin-bottom: 15rpx;
	padding-bottom: 10rpx;
	border-bottom: 1rpx solid #eee;
}

.meal-time {
	font-size: 28rpx;
	font-weight: bold;
	color: #333;
}

.date {
	font-size: 24rpx;
	color: #999;
}

.record-body {
	margin-bottom: 15rpx;
}

.food-info {
	display: flex;
	justify-content: space-between;
	align-items: center;
	margin-bottom: 10rpx;
}

.food-name {
	font-size: 30rpx;
	color: #333;
	font-weight: 500;
}

.weight {
	font-size: 24rpx;
	color: #999;
}

.nutrition-info {
	display: flex;
	flex-wrap: wrap;
	gap: 15rpx;
}

.calories, .protein, .carbs, .fat {
	font-size: 24rpx;
	color: #666;
	padding: 4rpx 8rpx;
	background: #f0f0f0;
	border-radius: 8rpx;
}

.record-actions {
	display: flex;
	gap: 15rpx;
	justify-content: flex-end;
}

.edit-btn, .delete-btn {
	padding: 8rpx 16rpx;
	border-radius: 10rpx;
	font-size: 24rpx;
	border: none;
	width: 300rpx;
}

.edit-btn {
	background: #4ECDC4;
	margin-right: 10rpx;
	color: white;
}

.delete-btn {
	background: #FF6B6B;
	color: white;
}

/* 弹窗样式 */
.modal-overlay {
	position: fixed;
	top: 0;
	left: 0;
	right: 0;
	bottom: 0;
	background: rgba(0, 0, 0, 0.5);
	display: flex;
	justify-content: center;
	align-items: center;
	z-index: 1000;
}

.modal-content {
	background: white;
	border-radius: 20rpx;
	width: 80%;
	max-width: 600rpx;
	padding: 30rpx;
	box-shadow: 0 20rpx 60rpx rgba(0, 0, 0, 0.3);
}

.modal-header {
	display: flex;
	justify-content: space-between;
	align-items: center;
	margin-bottom: 30rpx;
}

.modal-title {
	font-size: 32rpx;
	font-weight: bold;
	color: #333;
}

.modal-close {
	font-size: 40rpx;
	color: #999;
	cursor: pointer;
}

.form-content {
	display: flex;
	flex-direction: column;
	gap: 25rpx;
}

.form-group {
	display: flex;
	flex-direction: column;
	gap: 10rpx;
}

.form-label {
	font-size: 28rpx;
	color: #333;
	font-weight: 500;
}

.form-input {
	border: 1rpx solid #ddd;
	border-radius: 10rpx;
	padding: 15rpx;
	font-size: 28rpx;
}

.picker {
	border: 1rpx solid #ddd;
	border-radius: 10rpx;
	padding: 15rpx;
	font-size: 28rpx;
	background: #f9f9f9;
}

.form-actions {
	display: flex;
	gap: 20rpx;
	margin-top: 20rpx;
}

.cancel-btn, .confirm-btn {
	flex: 1;
	padding: 20rpx;
	border-radius: 15rpx;
	font-size: 28rpx;
	border: none;
}

.cancel-btn {
	background: #f0f0f0;
	color: #666;
}

.confirm-btn {
	background: #8CA9AD;
	color: white;
}
</style>