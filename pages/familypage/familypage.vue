<template>
	<view class="root">
		  <!-- 自定义导航栏 -->
		  <view class="navBarBox">
			<!-- 状态栏占位 -->
			<view class="statusBar" :style="{ paddingTop: statusBarHeight + 'px' }"></view>
			<!-- 真正的导航栏内容 -->
			<view class="navBar">
				<view style="color: #FBF2E3;font-size: 36rpx;font-weight: 500;">
					家庭中心
				</view>
			</view>
		</view>
		<view class="content">
      <view class="topContainer card-border">
        <view v-if="!Today" class="dateCard">
          <view>获取日期失败</view>
        </view>

        <view v-else class="dateCard">
          <view class="date">{{ Today.date }}</view>
          <view class="day-of-week">{{ Today.dayOfWeek }}</view>
          <view class="year">{{ Today.year }}</view>
          <view class="month">{{ Today.month }}月</view>
        </view>
        <view class="listCard">
          <view class="listTitle">今日日程</view>
          <view v-for="item in todoList" :key="item.id" class="listItem">
            <view class="itemContent">{{ item.content }}</view>
          </view>
        </view>
      </view>

		  <!-- 消息板 -->
		  <scroll-view class="messageBoard card-border">
        <view class="messageTitle">留言板</view>
				<view v-for="item in messageList" :key="item.id" class="messageItem">
					<view class="messageContent">{{ item.content }}</view>
          <view class="messageInfo">
            <view class="messageTime">{{ item.time }}</view>
            <view class="messageFrom">from {{ item.from }}</view>
          </view>
				</view>
			</scroll-view>
      <view class="big-button">
        <button @click="testJwtConnection()">jwt测试</button>
      </view>

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
      // 日期时间数据
      Today: {
        date: "",
        month: "",
        year: "",
        dayOfWeek: ""
      },
      timer: null,
      // 待办事项列表
      todoList: [
        { id: 1, content: '完成家庭作业' },
        { id: 2, content: '购买新的手机' },
        { id: 3, content: '和朋友聚餐' }
      ],
      // 消息列表
      messageList: [
        { id: 1, content: '你好，我是张三', time: '2023-10-10 10:00', from: '张三' },
        { id: 2, content: '你好，我是李四', time: '2023-10-10 10:01', from: '李四' },
        { id: 3, content: '你好，我是王五', time: '2023-10-10 10:02', from: '王五' },
        { id: 4, content: '你好，我是李四', time: '2023-10-10 10:01', from: '李四' },
        { id: 5, content: '你好，我是王五', time: '2023-10-10 10:02', from: '王五' },
        { id: 6, content: '你好，我是李四', time: '2023-10-10 10:01', from: '李四' },
        { id: 7, content: '你好，我是王五', time: '2023-10-10 10:02', from: '王五' }
      ]
    }
  },
  //第一次加载时调用
  created() {
      //获取手机状态栏高度
    this.statusBarHeight = uni.getSystemInfoSync()['statusBarHeight'];
      // 调用更新日期时间方法
    this.updateDateTime();
  },
  methods: {
    updateUserInfo(){
      // 从全局状态获取最新用户信息
      const globalData = getApp().globalData;
      this.userInfo = { ...globalData.userInfo };
    },
    // 更新日期时间显示
    updateDateTime() {
      const now = new Date();
      
      console.log('开始更新日期时间...', now);
      // 获取日期信息
      const year = now.getFullYear();
      const month = now.getMonth() + 1; // 月份从0开始，需要+1
      const date = now.getDate();
      
      // 获取星期几
      const dayOfWeek = now.getDay();
      const weekDays = ['日', '一', '二', '三', '四', '五', '六'];
      const dayOfWeekText = '星期' + weekDays[dayOfWeek];
      
      console.log('获取到的日期信息:', year, month, date, dayOfWeekText);
      // 格式化月份和日期，确保两位数显示
      const formattedMonth = month < 10 ? '0' + month : month.toString();
      const formattedDate = date < 10 ? '0' + date : date.toString();
      
      console.log('格式化后的日期信息:', formattedDate, formattedMonth, year.toString(), dayOfWeekText);
      console.log('更新数据中...')
      // 更新数据
      this.Today = {
        date: formattedDate,
        month: formattedMonth,
        year: year.toString(),
        dayOfWeek: dayOfWeekText
      };
      console.log('数据更新完成:', this.Today);
    },



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
  }
    
    

  }
}
</script>

<style>
  .content{
    width: 660rpx;
  }

  .topContainer{
    margin: 40rpx 0 20rpx;
    display: flex;
    flex-direction: row;
    align-items: center;
  }

	.dateCard{
    height: 240rpx;
    width: 240rpx;
  }

  .listCard{
    height: 240rpx;
    width: 420rpx;
  }

  .messageBoard{
    padding: 20rpx;
    height: 400rpx;
  }
  .messageTitle{
    font-size: 36rpx;
    font-weight: 500;
    color: #8CA9AD;
    margin-bottom: 20rpx;
  }
  .messageItem{
    border: 1px solid #000000;
    height: 80rpx;
    margin: 20rpx 0;
  }
  .messageItem .messageContent{
    max-width: 300rpx;
    font-size: 30rpx;
    font-weight: 480;
  }
  .messageInfo{
    display: flex;
    flex-direction: column;
    align-items: flex-end;
  }
  .messageInfo .messageTime{
    margin: 0 20rpx 0 50rpx;
    display:inline-block;
    text-align: right;
    font-size: 16rpx;
    font-weight: 360;
    color: #b7c5c7;
  }
  .messageInfo .messageFrom{
    display:inline-block;
    text-align: right;
    font-size: 20rpx;
    font-weight: 360;
    color: #b7c5c7;
  }
</style>