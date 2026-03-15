<template>
  <view class="content">
    <text>{{ backendMessage }}</text>
  </view>
</template>

<script>
export default {
  data() {
    return {
      backendMessage: '正在连接后端...',
    }
  },
  onLoad() {
    uni.request({
      url: 'http://localhost:5000/api/hello',
      success: (res) => {
        console.log('Backend response:', res.data);
        // 更新页面数据
        this.backendMessage = res.data.message || '后端服务连接成功！';
      },
      fail: (err) => {
        console.error('Request failed:', err);
        this.backendMessage = '连接后端失败，请检查服务是否启动';
      }
    });
  }
}
</script>