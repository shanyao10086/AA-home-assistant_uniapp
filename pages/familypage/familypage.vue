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
		<!-- 顶部容器 -->
		<view class="topContainer card-border">
			<!-- 日期卡片 -->
			<view v-if="!Today" class="dateCard">
			<view>获取日期失败</view>
			</view>

			<view v-else class="dateCard">
			<view class="date">{{ Today.date }}</view>
			<view class="day-of-week">{{ Today.dayOfWeek }}</view>
			<view class="year">{{ Today.year }}</view>
			<view class="month">{{ Today.month }}月</view>
			</view>
			
			<!-- 日程卡片 -->
		<view class="listCard">
		<view class="listHeader">
			<view class="listTitle">今日日程</view>
			<view class="refreshBtn" @click="loadTodayTodos" :class="{ loading: loadingTodos }">
				<text v-if="!loadingTodos">刷新</text>
				<text v-else>加载中...</text>
			</view>
		</view>
		<scroll-view class="listScroll" scroll-y>
			<view v-if="loadingTodos" class="loadingText">正在加载待办事项...</view>
			<view v-else-if="todoList.length === 0" class="emptyText">今日暂无待办事项</view>
			<view v-else v-for="item in todoList" :key="item.id" class="listItem" 
				  @longpress="deleteTodo(item.id)">
				<view class="itemContent">{{ item.content }}</view>
				<view v-if="item.description" class="itemDescription">{{ item.description }}</view>
			</view>
		</scroll-view>
		</view>
      </view>
		<button class="addBtn" @click="addTodo">添加日程</button>

		  <!-- 消息板 -->
		  <view class="messageSection">
        <view class="messageHeader">
          <view class="messageTitle">留言板</view>
          <view class="refreshBtn" @click="loadMessages" :class="{ loading: loading }">
            <text v-if="!loading">刷新</text>
            <text v-else>加载中...</text>
          </view>
        </view>
			<scroll-view class="messageBoard card-border" scroll-y>
				<view v-if="loading" class="loadingText">正在加载留言...</view>
				<view v-else-if="messageList.length === 0" class="emptyText">暂无留言，点击下方按钮添加</view>
					<view v-else v-for="item in messageList" :key="item.id" class="messageItem" 
					@longpress="isMyMessage(item) && deleteMessage(item.id)">
						<view class="messageContent">{{ item.content }}</view>
						<view class="messageInfo">
							<view class="messageTime">{{ item.time }}</view>
							<view class="messageFrom">from {{ item.from }}</view>
							<view v-if="isMyMessage(item)" class="myMessageTag">我的</view>
						</view>
				</view>
			</scroll-view>
		  <button class="addBtn" @click="addMessage">添加留言</button>
      </view>

      <!-- 家庭管理按钮 -->
      <view class="familySection">
        <button class="familyBtn" @click="handleFamilyAction">
          {{ familyInfo ? '查看家庭' : '创建家庭' }}
        </button>
      </view>

      <!-- 家庭管理弹窗 -->
      <view v-if="showFamilyModal" class="modal-overlay" @click="showFamilyModal = false">
        <view class="modal-content" @click.stop>
          <view class="modal-header">
            <text class="modal-title">家庭管理</text>
            <text class="modal-close" @click="showFamilyModal = false">×</text>
          </view>
          
          <view v-if="familyInfo && familyInfo.id" class="family-details">
            <view class="family-info">
              <text class="family-name">{{ familyInfo.name }}</text>
              <text class="family-description">{{ familyInfo.description || '暂无描述' }}</text>
              <text class="member-count">成员数量: {{ familyMembers.length }}</text>
            </view>
            
            <view class="members-list">
              <text class="members-title">家庭成员</text>
              <scroll-view class="members-scroll" scroll-y>
                <view v-for="member in familyMembers" :key="member.id" class="member-item">
                  <view class="member-info">
                    <text class="member-name">{{ member.nickname }}</text>
                    <text class="member-role">{{ member.is_family_admin ? '管理员' : '成员' }}</text>
                  </view>
                  <view v-if="isFamilyAdmin && !member.is_family_admin" class="member-actions">
                    <text class="delete-btn" @click="removeMember(member.id)">删除</text>
                  </view>
                </view>
              </scroll-view>
            </view>
            
            <view v-if="isFamilyAdmin" class="admin-actions">
              <button class="add-member-btn" @click="showAddMemberModal = true">添加成员</button>
            </view>
          </view>
          
          <view v-else class="create-family">
            <text class="create-title">创建新家庭</text>
            <input v-model="familyInfo.name" class="family-input" placeholder="请输入家庭名称" />
            <input v-model="familyInfo.description" class="family-input" placeholder="请输入家庭描述（可选）" />
            <button class="create-btn" @click="createFamily">创建家庭</button>
            <text class="create-tips">创建家庭后，您将成为家庭管理员</text>
          </view>
        </view>
      </view>

      <!-- 添加成员弹窗 -->
      <view v-if="showAddMemberModal" class="modal-overlay" @click="showAddMemberModal = false">
        <view class="modal-content" @click.stop>
          <view class="modal-header">
            <text class="modal-title">添加成员</text>
            <text class="modal-close" @click="showAddMemberModal = false">×</text>
          </view>
          
          <view class="add-member-form">
            <text class="form-label">请输入成员的微信OpenID：</text>
            <input v-model="newMemberOpenId" class="member-input" placeholder="输入OpenID" />
            <view class="form-actions">
              <button class="cancel-btn" @click="showAddMemberModal = false">取消</button>
              <button class="confirm-btn" @click="addMember">确认添加</button>
            </view>
          </view>
        </view>
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
      // 待办事项列表（从后端获取）
      todoList: [],
      // 消息列表（从后端获取）
      messageList: [],
      // 加载状态
      loading: false,
      loadingTodos: false,
      loadingFamily: false,
      // 当前用户信息
      currentUser: null,
      // 家庭管理相关数据
      familyInfo: null,
      familyMembers: [],
      // 家庭管理弹窗状态
      showFamilyModal: false,
      showAddMemberModal: false,
      newMemberOpenId: ''
    }
  },
  //第一次加载时调用
  created() {
      //获取手机状态栏高度
    this.statusBarHeight = uni.getSystemInfoSync()['statusBarHeight'];
      // 调用更新日期时间方法
    this.updateDateTime();
    // 获取当前用户信息
    this.getCurrentUser();
    // 加载留言列表
    this.loadMessages();
    // 加载今日待办
    this.loadTodayTodos();
    // 检查家庭状态
    this.checkFamilyStatus();
  },
  
  // 页面显示时重新加载数据
  onShow() {
    this.loadMessages();
    this.loadTodayTodos();
    this.checkFamilyStatus();
  },
  methods: {
    // 获取当前用户信息
    getCurrentUser() {
      const userInfo = uni.getStorageSync('userInfo');
      if (userInfo) {
        this.currentUser = userInfo;
      }
    },
    
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
     * 添加待办事项
     */
    async addTodo() {
      const token = uni.getStorageSync('access_token');
      if (!token) {
        uni.showToast({
          title: '请先登录',
          icon: 'none'
        });
        return;
      }

      // 弹出输入框让用户输入事件信息
      const result = await new Promise((resolve) => {
        uni.showModal({
          title: '添加今日待办',
          content: '请输入事件标题',
          editable: true,
          placeholderText: '请输入事件标题（1-100字符）',
          success: resolve
        });
      });

      if (!result.confirm || !result.content) {
        return;
      }

      const title = result.content.trim();
      if (title.length === 0 || title.length > 100) {
        uni.showToast({
          title: '事件标题长度应在1-100字符之间',
          icon: 'none'
        });
        return;
      }

      try {
        const baseUrl = getApp().globalData.baseUrl;
        const today = new Date();
        const todayStr = today.toISOString().split('T')[0]; // YYYY-MM-DD格式
        
        const response = await new Promise((resolve, reject) => {
          uni.request({
            url: `${baseUrl}/calendar/`,
            method: 'POST',
            header: {
              'Authorization': `Bearer ${token}`,
              'Content-Type': 'application/json'
            },
            data: {
              title: title,
              date: todayStr,
              description: '今日待办事项',
              color: '#8CA9AD'
            },
            success: resolve,
            fail: reject
          });
        });

        if (response.statusCode === 200 && response.data.success) {
          uni.showToast({
            title: '待办事项添加成功',
            icon: 'success'
          });
          // 重新加载今日待办
          this.loadTodayTodos();
        } else {
          console.error('添加待办事项失败:', response.data);
          uni.showToast({
            title: response.data.message || '添加失败',
            icon: 'none'
          });
        }
      } catch (error) {
        console.error('网络请求失败:', error);
        uni.showToast({
          title: '网络错误',
          icon: 'none'
        });
      }
    },

    /**
     * 加载今日待办事项
     */
    async loadTodayTodos() {
      const token = uni.getStorageSync('access_token');
      if (!token) {
        console.log('未登录，无法获取今日待办');
        return;
      }

      this.loadingTodos = true;
      
      try {
        const baseUrl = getApp().globalData.baseUrl;
        const response = await new Promise((resolve, reject) => {
          uni.request({
            url: `${baseUrl}/calendar/today`,
            method: 'GET',
            header: {
              'Authorization': `Bearer ${token}`,
              'Content-Type': 'application/json'
            },
            success: resolve,
            fail: reject
          });
        });

        if (response.statusCode === 200 && response.data.success) {
          // 格式化待办数据
        this.todoList = response.data.data.map(item => ({
          id: item.id,
          content: item.title,
          description: item.description,
          color: item.color,
          isToday: item.is_today,
          isActive: item.is_active,
          date: item.date
        }));
          console.log('今日待办加载成功:', this.todoList);
        } else {
          console.error('获取今日待办失败:', response.data);
          // 失败时显示默认数据
          this.todoList = [
            { id: 1, content: '完成家庭作业' },
            { id: 2, content: '购买新的手机' },
            { id: 3, content: '和朋友聚餐' }
          ];
        }
      } catch (error) {
        console.error('网络请求失败:', error);
        // 失败时显示默认数据
        this.todoList = [
          { id: 1, content: '完成家庭作业' },
          { id: 2, content: '购买新的手机' },
          { id: 3, content: '和朋友聚餐' }
        ];
      } finally {
        this.loadingTodos = false;
      }
    },

    /**
     * 删除待办事项
     */
    async deleteTodo(todoId) {
      const token = uni.getStorageSync('access_token');
      if (!token) {
        uni.showToast({
          title: '请先登录',
          icon: 'none'
        });
        return;
      }

      const { confirm } = await new Promise((resolve) => {
        uni.showModal({
          title: '确认删除',
          content: '确定要删除这个待办事项吗？',
          success: resolve
        });
      });

      if (!confirm) return;

      try {
        const baseUrl = getApp().globalData.baseUrl;
        const response = await new Promise((resolve, reject) => {
          uni.request({
            url: `${baseUrl}/calendar/${todoId}`,
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
            title: '待办事项删除成功',
            icon: 'success'
          });
          // 重新加载今日待办
          this.loadTodayTodos();
        } else {
          console.error('删除待办事项失败:', response.data);
          uni.showToast({
            title: response.data.message || '删除失败',
            icon: 'none'
          });
        }
      } catch (error) {
        console.error('网络请求失败:', error);
        uni.showToast({
          title: '网络错误',
          icon: 'none'
        });
      }
    },

    /**
     * 加载留言列表
     */
    async loadMessages() {
      const token = uni.getStorageSync('access_token');
      if (!token) {
        console.log('未登录，无法获取留言列表');
        return;
      }

      this.loading = true;
      
      try {
        const baseUrl = getApp().globalData.baseUrl;
        const response = await new Promise((resolve, reject) => {
          uni.request({
            url: `${baseUrl}/messages/`,
            method: 'GET',
            header: {
              'Authorization': `Bearer ${token}`,
              'Content-Type': 'application/json'
            },
            data: {
              type: 'active',
              limit: 50
            },
            success: resolve,
            fail: reject
          });
        });

        if (response.statusCode === 200 && response.data.success) {
          // 格式化留言数据
          this.messageList = response.data.data.map(item => ({
            id: item.id,
            content: item.content,
            time: this.formatTime(item.date_posted),
            from: item.nickname || item.username || '未知用户',
            isActive: item.is_active,
            userId: item.user_id,
            endTime: item.end_time
          }));
          console.log('留言列表加载成功:', this.messageList);
        } else {
          console.error('获取留言列表失败:', response.data);
          uni.showToast({
            title: '获取留言失败',
            icon: 'none'
          });
        }
      } catch (error) {
        console.error('网络请求失败:', error);
        uni.showToast({
          title: '网络错误',
          icon: 'none'
        });
      } finally {
        this.loading = false;
      }
    },

    /**
     * 添加留言
     */
    async addMessage() {
      const token = uni.getStorageSync('access_token');
      if (!token) {
        uni.showToast({
          title: '请先登录',
          icon: 'none'
        });
        return;
      }

      // 弹出输入框让用户输入留言内容
      const result = await new Promise((resolve) => {
        uni.showModal({
          title: '发布留言',
          content: '请输入留言内容',
          editable: true,
          placeholder: '请输入留言内容（1-1000字符）',
          success: resolve
        });
      });

      if (!result.confirm || !result.content) {
        return;
      }

      const content = result.content.trim();
      if (content.length === 0 || content.length > 1000) {
        uni.showToast({
          title: '留言内容长度应在1-1000字符之间',
          icon: 'none'
        });
        return;
      }

      try {
        const baseUrl = getApp().globalData.baseUrl;
        const response = await new Promise((resolve, reject) => {
          uni.request({
            url: `${baseUrl}/messages/create`,
            method: 'POST',
            header: {
              'Authorization': `Bearer ${token}`,
              'Content-Type': 'application/json'
            },
            data: {
              content: content,
              duration_hours: 24 // 默认24小时有效
            },
            success: resolve,
            fail: reject
          });
        });

        if (response.statusCode === 201 && response.data.success) {
          uni.showToast({
            title: '留言发布成功',
            icon: 'success'
          });
          // 重新加载留言列表
          this.loadMessages();
        } else {
          console.error('发布留言失败:', response.data);
          
          let errorMsg = '发布失败';
          if (response.data.error) {
            errorMsg = response.data.error;
          } else if (response.data.message) {
            errorMsg = response.data.message;
          }
          
          uni.showToast({
            title: errorMsg,
            icon: 'none'
          });
        }
      } catch (error) {
        console.error('网络请求失败:', error);
        uni.showToast({
          title: '网络错误',
          icon: 'none'
        });
      }
    },

    /**
     * 删除留言
     */
    async deleteMessage(messageId) {
      const token = uni.getStorageSync('access_token');
      if (!token) {
        uni.showToast({
          title: '请先登录',
          icon: 'none'
        });
        return;
      }

      const { confirm } = await new Promise((resolve) => {
        uni.showModal({
          title: '确认删除',
          content: '确定要删除这条留言吗？',
          success: resolve
        });
      });

      if (!confirm) return;

      try {
        const baseUrl = getApp().globalData.baseUrl;
        const response = await new Promise((resolve, reject) => {
          uni.request({
            url: `${baseUrl}/messages/${messageId}`,
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
            title: '留言删除成功',
            icon: 'success'
          });
          // 重新加载留言列表
          this.loadMessages();
        } else {
          console.error('删除留言失败:', response.data);
          uni.showToast({
            title: response.data.message || '删除失败',
            icon: 'none'
          });
        }
      } catch (error) {
        console.error('网络请求失败:', error);
        uni.showToast({
          title: '网络错误',
          icon: 'none'
        });
      }
    },

    /**
     * 格式化时间显示
     */
    formatTime(timestamp) {
      if (!timestamp) return '';
      
      const date = new Date(timestamp);
      const now = new Date();
      const diff = now.getTime() - date.getTime();
      
      // 如果是今天，显示时间
      if (date.toDateString() === now.toDateString()) {
        return date.toLocaleTimeString('zh-CN', { 
          hour: '2-digit', 
          minute: '2-digit' 
        });
      }
      
      // 如果是昨天
      const yesterday = new Date(now);
      yesterday.setDate(yesterday.getDate() - 1);
      if (date.toDateString() === yesterday.toDateString()) {
        return '昨天 ' + date.toLocaleTimeString('zh-CN', { 
          hour: '2-digit', 
          minute: '2-digit' 
        });
      }
      
      // 其他情况显示完整日期
      return date.toLocaleDateString('zh-CN') + ' ' + 
             date.toLocaleTimeString('zh-CN', { 
               hour: '2-digit', 
               minute: '2-digit' 
             });
    },

    /**
     * 判断是否为当前用户的留言
     */
    isMyMessage(message) {
      return this.currentUser && message.userId === this.currentUser.id;
    },

    /**
     * 检查家庭状态
     */
    async checkFamilyStatus() {
      const token = uni.getStorageSync('access_token');
      if (!token) {
        console.log('未登录，无法检查家庭状态');
        return;
      }

      this.loadingFamily = true;
      
      try {
        const baseUrl = getApp().globalData.baseUrl;
        console.log('开始检查家庭状态，请求URL:', `${baseUrl}/families/members`);
        
        const response = await new Promise((resolve, reject) => {
          uni.request({
            url: `${baseUrl}/families/members`,
            method: 'GET',
            header: {
              'Authorization': `Bearer ${token}`
            },
            success: resolve,
            fail: reject
          });
        });

        console.log('家庭状态检查响应:', response);

        if (response.statusCode === 200 && response.data.success) {
          // 修复数据结构：response.data.data.data 才是真正的成员数组
          const membersData = response.data.data || [];
          this.familyMembers = membersData;
          console.log('获取到的家庭成员列表:', this.familyMembers);
          
          if (this.familyMembers && this.familyMembers.length > 0) {
            // 从第一个成员获取家庭信息
            const firstMember = this.familyMembers[0];
            this.familyInfo = {
              id: firstMember.family_id,
              name: firstMember.family_name,
              description: firstMember.family_description || ''
            };
            console.log('家庭信息设置成功:', this.familyInfo);
          } else {
            console.log('家庭成员列表为空，用户未加入家庭');
            console.log('后端返回消息:', response.data.message);
            this.familyInfo = null;
            this.familyMembers = [];
          }
        } else {
          console.log('API返回失败:', response.data);
          this.familyInfo = null;
          this.familyMembers = [];
        }
      } catch (error) {
        console.error('检查家庭状态失败:', error);
        this.familyInfo = null;
        this.familyMembers = [];
      } finally {
        this.loadingFamily = false;
      }
    },

    /**
     * 处理家庭管理按钮点击
     */
    handleFamilyAction() {
      if (!this.currentUser) {
        uni.showToast({
          title: '请先登录',
          icon: 'none'
        });
        return;
      }
      
      console.log('点击家庭管理按钮，当前家庭信息:', this.familyInfo);
      
      // 显示弹窗
      this.showFamilyModal = true;
      
      // 如果没有家庭信息，初始化创建表单
      if (!this.familyInfo || !this.familyInfo.id) {
        this.familyInfo = {
          name: '',
          description: ''
        };
        console.log('初始化创建家庭表单');
      } else {
        // 如果已经有家庭信息，重新加载成员列表
        console.log('重新加载家庭成员列表');
        this.checkFamilyStatus();
      }
    },

    /**
     * 创建家庭
     */
    async createFamily() {
      const token = uni.getStorageSync('access_token');
      if (!token) {
        uni.showToast({
          title: '请先登录',
          icon: 'none'
        });
        return;
      }

      if (!this.familyInfo.name || this.familyInfo.name.trim().length === 0) {
        uni.showToast({
          title: '请输入家庭名称',
          icon: 'none'
        });
        return;
      }

      try {
        const baseUrl = getApp().globalData.baseUrl;
        const requestData = {
          name: this.familyInfo.name.trim(),
          description: this.familyInfo.description ? this.familyInfo.description.trim() : ''
        };
        
        console.log('开始创建家庭，请求数据:', requestData);
        console.log('请求URL:', `${baseUrl}/families/create`);

        const response = await new Promise((resolve, reject) => {
          uni.request({
            url: `${baseUrl}/families/create`,
            method: 'POST',
            header: {
              'Authorization': `Bearer ${token}`,
              'Content-Type': 'application/json'
            },
            data: requestData,
            success: resolve,
            fail: reject
          });
        });

        console.log('创建家庭响应:', response);

        if (response.statusCode === 201 && response.data.success) {
          uni.showToast({
            title: '家庭创建成功',
            icon: 'success'
          });
          this.showFamilyModal = false;
          // 重置表单
          this.familyInfo = null;
          // 延迟1秒后重新检查家庭状态，确保后端数据已更新
          setTimeout(() => {
            this.checkFamilyStatus();
          }, 1000);
        } else {
          console.error('创建家庭失败，状态码:', response.statusCode);
          console.error('响应数据:', response.data);
          
          let errorMsg = '创建失败';
          if (response.statusCode === 400) {
            errorMsg = response.data.error || '请求参数错误';
          } else if (response.statusCode === 401) {
            errorMsg = '登录已过期，请重新登录';
          } else if (response.statusCode === 500) {
            errorMsg = '服务器内部错误';
          }
          
          uni.showToast({
            title: errorMsg,
            icon: 'none'
          });
        }
      } catch (error) {
        console.error('网络请求失败:', error);
        uni.showToast({
          title: '网络错误，请检查服务器连接',
          icon: 'none'
        });
      }
    },

    /**
     * 添加家庭成员
     */
    async addMember() {
      const token = uni.getStorageSync('access_token');
      if (!token) {
        uni.showToast({
          title: '请先登录',
          icon: 'none'
        });
        return;
      }

      if (!this.newMemberOpenId || this.newMemberOpenId.trim().length === 0) {
        uni.showToast({
          title: '请输入成员OpenID',
          icon: 'none'
        });
        return;
      }

      try {
        const baseUrl = getApp().globalData.baseUrl;
        const response = await new Promise((resolve, reject) => {
          uni.request({
            url: `${baseUrl}/families/add_member`,
            method: 'POST',
            header: {
              'Authorization': `Bearer ${token}`,
              'Content-Type': 'application/json'
            },
            data: {
              user_openid: this.newMemberOpenId.trim()
            },
            success: resolve,
            fail: reject
          });
        });

        if (response.statusCode === 200 && response.data.success) {
          uni.showToast({
            title: '成员添加成功',
            icon: 'success'
          });
          this.showAddMemberModal = false;
          this.newMemberOpenId = '';
          // 重新加载家庭成员列表
          this.checkFamilyStatus();
        } else {
          console.error('添加成员失败:', response.data);
          uni.showToast({
            title: response.data.error || '添加失败',
            icon: 'none'
          });
        }
      } catch (error) {
        console.error('网络请求失败:', error);
        uni.showToast({
          title: '网络错误',
          icon: 'none'
        });
      }
    },

    /**
     * 删除家庭成员
     */
    async removeMember(userId) {
      const token = uni.getStorageSync('access_token');
      if (!token) {
        uni.showToast({
          title: '请先登录',
          icon: 'none'
        });
        return;
      }

      const { confirm } = await new Promise((resolve) => {
        uni.showModal({
          title: '确认删除',
          content: '确定要删除这个家庭成员吗？',
          success: resolve
        });
      });

      if (!confirm) return;

      try {
        const baseUrl = getApp().globalData.baseUrl;
        const response = await new Promise((resolve, reject) => {
          uni.request({
            url: `${baseUrl}/families/remove_member`,
            method: 'POST',
            header: {
              'Authorization': `Bearer ${token}`,
              'Content-Type': 'application/json'
            },
            data: {
              user_id: userId
            },
            success: resolve,
            fail: reject
          });
        });

        if (response.statusCode === 200 && response.data.success) {
          uni.showToast({
            title: '成员删除成功',
            icon: 'success'
          });
          // 重新加载家庭成员列表
          this.checkFamilyStatus();
        } else {
          console.error('删除成员失败:', response.data);
          uni.showToast({
            title: response.data.error || '删除失败',
            icon: 'none'
          });
        }
      } catch (error) {
        console.error('网络请求失败:', error);
        uni.showToast({
          title: '网络错误',
          icon: 'none'
        });
      }
    },

    /**
     * 判断当前用户是否为家庭管理员
     */
    get isFamilyAdmin() {
      if (!this.currentUser || !this.familyMembers.length) return false;
      
      const currentMember = this.familyMembers.find(member => 
        member.id === this.currentUser.id
      );
      
      return currentMember ? currentMember.is_family_admin : false;
    }
    
  }
}
</script>

<style>
  .content{
    width: 660rpx;
    padding: 0 20rpx;
  }

  .topContainer{
    margin: 40rpx 0 20rpx;
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: space-between;
  }

	.dateCard{
    height: 240rpx;
    width: 240rpx;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    background: linear-gradient(135deg, #8CA9AD 0%, #7A9AA5 100%);
    border-radius: 20rpx;
    color: white;
    padding: 20rpx;
    box-shadow: 0 4rpx 20rpx rgba(140, 169, 173, 0.3);
  }

  .dateCard .date {
    font-size: 48rpx;
    font-weight: bold;
    margin-bottom: 10rpx;
  }

  .dateCard .day-of-week {
    font-size: 28rpx;
    margin-bottom: 5rpx;
  }

  .dateCard .year, .dateCard .month {
    font-size: 24rpx;
    opacity: 0.8;
  }

  /* 日程卡片样式 */
  .listCard{
    height: 240rpx;
    width: 380rpx;
    background: white;
    border-radius: 20rpx;
    padding: 20rpx;
    border-bottom: 4rpx solid #E0E0E0;
    border-right: 3rpx solid #E0E0E0;
  }

  .listHeader {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20rpx;
  }

  .listTitle{
    font-size: 32rpx;
    font-weight: 600;
    color: #333;
  }

  .listScroll {
    height: 160rpx;
  }

  .listItem {
    padding: 15rpx 0;
    border-bottom: 1rpx solid #f0f0f0;
  }

  .listItem:last-child {
    border-bottom: none;
  }

  .itemContent {
    font-size: 32rpx;
    color: #666;
  }

  /* 留言板区域样式 */
  .messageSection {
    margin: 30rpx 0;
  }

  .messageHeader {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20rpx;
  }

  .messageTitle{
    font-size: 36rpx;
    font-weight: 500;
    color: #8CA9AD;
  }

  .refreshBtn {
    padding: 10rpx 20rpx;
    background: #8CA9AD;
    color: white;
    border-radius: 20rpx;
    font-size: 24rpx;
    border: none;
  }

  .refreshBtn.loading {
    background: #A8BEC9;
  }

  .messageBoard{
    padding: 20rpx;
    height: 400rpx;
    background: white;
    border-radius: 20rpx;
    border-bottom: 4rpx solid #E0E0E0;
    border-right: 3rpx solid #E0E0E0;
  }

  .loadingText, .emptyText {
    text-align: center;
    color: #999;
    font-size: 28rpx;
    padding: 40rpx;
  }

  .messageItem{
    padding: 20rpx;
    margin: 20rpx 0;
    background: #f8f9fa;
    border-radius: 15rpx;
    border-left: 6rpx solid #8CA9AD;
    display: flex;
    justify-content: space-between;
    align-items: center;
    transition: all 0.3s ease;
  }

  .messageItem:active {
    background: #e9ecef;
    transform: scale(0.98);
  }

  .messageItem .messageContent{
    max-width: 300rpx;
    font-size: 30rpx;
    font-weight: 480;
    color: #333;
    flex: 1;
  }

  .messageInfo{
    display: flex;
    flex-direction: column;
    align-items: flex-end;
    min-width: 120rpx;
  }

  .messageInfo .messageTime{
    font-size: 20rpx;
    font-weight: 360;
    color: #b7c5c7;
    margin-bottom: 5rpx;
  }

  .messageInfo .messageFrom{
    font-size: 22rpx;
    font-weight: 360;
    color: #b7c5c7;
    margin-bottom: 5rpx;
  }

  .myMessageTag {
    background: #8CA9AD;
    color: white;
    font-size: 18rpx;
    padding: 4rpx 8rpx;
    border-radius: 8rpx;
    margin-top: 5rpx;
  }

  /* 按钮样式 */
  .addBtn {
    background: #8CA9AD;
    color: #FBF2E3;
    border: none;
    border-radius: 20rpx;
    padding: 10rpx 30rpx;
    font-size: 32rpx;
    font-weight: 500;
    margin-top: 20rpx;
    width: 100%;
    border-bottom: 4rpx solid #7A9AA5;
    border-right: 3rpx solid #7A9AA5;
  }

  .addBtn:active {
    transform: scale(0.95);
  }

  .actionBtn {
    background: #8CA9AD;
    color: white;
    border: none;
    border-radius: 20rpx;
    padding: 20rpx 60rpx;
    font-size: 28rpx;
    font-weight: 500;
    border-bottom: 4rpx solid #7A9AA5;
    border-right: 3rpx solid #7A9AA5;
  }

  .actionBtn:active {
    transform: scale(0.95);
  }

  /* 家庭管理区域样式 */
  .familySection {
    margin: 30rpx 0;
    display: flex;
    justify-content: center;
  }

  .familyBtn {
    background: #8CA9AD;
    color: white;
    border: none;
    border-radius: 20rpx;
    padding: 20rpx 60rpx;
    font-size: 28rpx;
    font-weight: 500;
    border-bottom: 4rpx solid #7A9AA5;
    border-right: 3rpx solid #7A9AA5;
  }

  .familyBtn:active {
    transform: scale(0.95);
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
    z-index: 9999;
  }

  .modal-content {
    background: white;
    border-radius: 20rpx;
    width: 600rpx;
    max-height: 80vh;
    padding: 40rpx;
    box-shadow: 0 10rpx 50rpx rgba(0, 0, 0, 0.3);
  }

  .modal-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 30rpx;
    border-bottom: 2rpx solid #f0f0f0;
    padding-bottom: 20rpx;
  }

  .modal-title {
    font-size: 32rpx;
    font-weight: 600;
    color: #333;
  }

  .modal-close {
    font-size: 40rpx;
    color: #999;
    cursor: pointer;
  }

  /* 家庭详情样式 */
  .family-info {
    margin-bottom: 30rpx;
  }

  .family-name {
    font-size: 28rpx;
    font-weight: 600;
    color: #333;
    display: block;
    margin-bottom: 10rpx;
  }

  .family-description {
    font-size: 24rpx;
    color: #666;
    display: block;
    margin-bottom: 10rpx;
  }

  .member-count {
    font-size: 22rpx;
    color: #8CA9AD;
    display: block;
  }

  .members-list {
    margin-bottom: 30rpx;
  }

  .members-title {
    font-size: 26rpx;
    font-weight: 600;
    color: #333;
    display: block;
    margin-bottom: 20rpx;
  }

  .members-scroll {
    max-height: 300rpx;
  }

  .member-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 20rpx;
    border-bottom: 1rpx solid #f0f0f0;
  }

  .member-info {
    flex: 1;
  }

  .member-name {
    font-size: 24rpx;
    color: #333;
    display: block;
    margin-bottom: 5rpx;
  }

  .member-role {
    font-size: 20rpx;
    color: #8CA9AD;
    display: block;
  }

  .member-actions {
    margin-left: 20rpx;
  }

  .delete-btn {
    color: #ff6b6b;
    font-size: 22rpx;
    cursor: pointer;
    padding: 10rpx 20rpx;
    border-radius: 10rpx;
    background: #fff5f5;
  }

  .admin-actions {
    text-align: center;
  }

  .add-member-btn {
    background: #8CA9AD;
    color: white;
    border: none;
    border-radius: 15rpx;
    padding: 15rpx 30rpx;
    font-size: 24rpx;
  }

  /* 创建家庭样式 */
  .create-title {
    font-size: 28rpx;
    font-weight: 600;
    color: #333;
    display: block;
    margin-bottom: 20rpx;
  }

  .family-input {
    width: 100%;
    padding: 20rpx;
    border: 2rpx solid #e0e0e0;
    border-radius: 10rpx;
    margin-bottom: 20rpx;
    font-size: 24rpx;
  }

  .create-btn {
    background: #8CA9AD;
    color: white;
    border: none;
    border-radius: 15rpx;
    padding: 20rpx 40rpx;
    font-size: 26rpx;
    width: 100%;
  }

  /* 添加成员表单样式 */
  .add-member-form {
    padding: 20rpx 0;
  }

  .form-label {
    font-size: 24rpx;
    color: #333;
    display: block;
    margin-bottom: 15rpx;
  }

  .member-input {
    width: 100%;
    padding: 20rpx;
    border: 2rpx solid #e0e0e0;
    border-radius: 10rpx;
    margin-bottom: 30rpx;
    font-size: 24rpx;
  }

  .form-actions {
    display: flex;
    justify-content: space-between;
  }

  .cancel-btn {
    background: #f0f0f0;
    color: #666;
    border: none;
    border-radius: 15rpx;
    padding: 15rpx 30rpx;
    font-size: 24rpx;
    flex: 1;
    margin-right: 15rpx;
  }

  .confirm-btn {
    background: #8CA9AD;
    color: white;
    border: none;
    border-radius: 15rpx;
    padding: 15rpx 30rpx;
    font-size: 24rpx;
    flex: 1;
    margin-left: 15rpx;
  }

  /* 卡片边框样式 */
  .card-border {
    border: 1rpx solid #e0e0e0;
  }
</style>