<template>
	<view class="health-page">
		<view class="status-bar"></view>
		<view class="header klein-blue">
			<text class="page-title white-text">健康管理</text>
		</view>

		<!-- 数据概览 -->
		<view class="summary-cards">
			<view class="card">
				<text class="label">今日步数</text>
				<text class="value">{{ stepCount }} <text class="unit">步</text></text>
				<view class="progress-bg">
					<view class="progress" :style="{ width: stepProgress + '%' }"></view>
				</view>
				<text class="goal">目标：8000 步</text>
			</view>
			<view class="card">
				<text class="label">睡眠时长</text>
				<text class="value">{{ sleepHours }}<text class="unit">h</text></text>
				<view class="progress-bg">
					<view class="progress sleep" :style="{ width: sleepProgress + '%' }"></view>
				</view>
				<text class="goal">建议：7-9 小时</text>
			</view>
		</view>

		<!-- 近期数据（用列表代替图表） -->
		<view class="section-title"> 近3日步数</view>
		<view class="history-list">
			<view class="history-item" v-for="(item, index) in historyData" :key="index">
				<text>{{ item.date }}</text>
				<text class="history-value">{{ item.steps }} 步</text>
			</view>
		</view>

		<!-- 健康建议 -->
		<view class="section-title"> 健康建议</view>
			<view class="advice-list">
				<view class="advice-item" v-for="(item, index) in adviceList" :key="index">
				<u-icon name="order" color="#002F6C" size="36"></u-icon>
				<text class="advice-text">{{ item }}</text>
			</view>
		</view>

		<view class="footer-space"></view>
	</view>
</template>

<script>
export default {
  data() {
    return {
      stepCount: 6520,
      sleepHours: 7.2,
      historyData: [
        { date: '今天', steps: 6520 },
        { date: '昨天', steps: 8200 },
        { date: '前天', steps: 7300 }
      ],
      adviceList: [
        '保持每日 8000 步以上，有助于心血管健康',
        '建议每晚 22:30 前入睡，保证深度睡眠',
        '多喝水，每日饮水量不少于 1500ml'
      ]
    };
  },
  computed: {
    stepProgress() {
      return Math.min(100, (this.stepCount / 8000) * 100);
    },
    sleepProgress() {
      return Math.min(100, (this.sleepHours / 9) * 100);
    }
  }
};
</script>

<style scoped>
/* ... （保留之前的样式，仅新增以下部分）... */

.history-list {
  background: #FFFFFF;
  border-radius: 20rpx;
  padding: 20rpx;
  margin-bottom: 40rpx;
  box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
}

.history-item {
  display: flex;
  justify-content: space-between;
  padding: 20rpx 0;
  border-bottom: 1rpx solid #f0f0f0;
}

.history-item:last-child {
  border-bottom: none;
}

.history-value {
  color: #002F6C;
  font-weight: bold;
}

.health-page {
  padding: 0 30rpx;
  min-height: 100vh;
  background-color: #E5F4FF;
}

.status-bar {
  height: var(--status-bar-height);
}

.header {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100rpx;
  border-radius: 0 0 30rpx 30rpx;
  margin-bottom: 40rpx;
}

.page-title {
  font-size: 40rpx;
  font-weight: bold;
}

/* 数据概览卡片 */
.summary-cards {
  display: flex;
  justify-content: space-between;
  margin-bottom: 40rpx;
}

.card {
  width: 48%;
  background: #FFFFFF;
  border-radius: 20rpx;
  padding: 30rpx;
  box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
}

.label {
  font-size: 28rpx;
  color: #666;
  margin-bottom: 10rpx;
}

.value {
  font-size: 48rpx;
  font-weight: bold;
  color: #002F6C;
  margin-bottom: 20rpx;
}

.unit {
  font-size: 32rpx;
  font-weight: normal;
}

.progress-bg {
  width: 100%;
  height: 12rpx;
  background: #E0E0E0;
  border-radius: 6rpx;
  overflow: hidden;
  margin-bottom: 10rpx;
}

.progress {
  height: 100%;
  background: #002F6C;
  border-radius: 6rpx;
}

.progress.sleep {
  background: #4A90E2; /* 睡眠用浅蓝色区分 */
}

.goal {
  font-size: 24rpx;
  color: #999;
}

/* 图表与建议 */
.section-title {
  font-size: 36rpx;
  font-weight: bold;
  color: #002F6C;
  margin: 20rpx 0 30rpx;
}

.chart-container {
  background: #FFFFFF;
  border-radius: 20rpx;
  padding: 20rpx;
  margin-bottom: 40rpx;
  box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
}

.advice-list {
  background: #FFFFFF;
  border-radius: 20rpx;
  padding: 20rpx;
  box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
}

.advice-item {
  display: flex;
  align-items: flex-start;
  gap: 20rpx;
  margin-bottom: 20rpx;
  padding-bottom: 20rpx;
  border-bottom: 1rpx solid #f0f0f0;
}

.advice-item:last-child {
  border-bottom: none;
  margin-bottom: 0;
  padding-bottom: 0;
}

.advice-text {
  font-size: 28rpx;
  color: #333;
  line-height: 1.5;
}

.footer-space {
  height: 120rpx;
}
</style>