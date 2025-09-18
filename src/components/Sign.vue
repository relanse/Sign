<template>
  <div class="signin-container">
    <h1>课程实时签到二维码生成器</h1>

    <!-- 输入框和按钮区域 -->
    <div class="input-section">
      <label for="url-input">粘贴任意时间的签到链接:</label>
      <input
        id="url-input"
        type="text"
        v-model="inputUrl"
        placeholder="例如: https://acm.sztu.edu.cn:40130/hwsys/..."
        @keyup.enter="parseUrlAndGenerate"
      />
      <button @click="parseUrlAndGenerate">
        从链接生成
      </button>
      <p v-if="errorMessage" class="error-message">{{ errorMessage }}</p>
    </div>

    <!-- 二维码显示区域 -->
    <div v-if="courseId" class="qrcode-display">
      <p>请扫描下方二维码进行签到</p>
      <div class="qrcode-wrapper">
        <qrcode-vue
          :value="qrCodeUrl"
          :size="300"
          level="H"
        />
      </div>
      <div class="info">
        <p><strong>当前课程ID:</strong> {{ courseId }}</p>
        <p>
          <strong>实时链接 (每2秒刷新):</strong>
          <span class="url-text">{{ qrCodeUrl }}</span>
        </p>
      </div>
    </div>
    <div v-else class="placeholder">
      <p>粘贴链接并点击按钮后，将在此处显示实时二维码。</p>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, onBeforeUnmount } from 'vue';
import QrcodeVue from 'qrcode.vue';

// --- 配置信息 ---
const refreshInterval = 2000; // 二维码刷新间隔（毫秒）
const timestampOffset = 4000; // 结束时间戳与开始时间戳的差值（毫秒），取一个平均值
// --- 配置结束 ---

const inputUrl = ref('');        // 绑定输入框的URL
const courseId = ref(null);      // 从URL中解析出的课程ID
const qrCodeUrl = ref('');       // 用于生成二维码的实时URL
const errorMessage = ref('');    // 用于显示错误信息
let intervalId = null;           // 定时器ID

/**
 * 从输入框的URL中解析出课程ID (UUID格式)
 */
const parseUrlAndGenerate = () => {
  if (!inputUrl.value) {
    errorMessage.value = '链接不能为空！';
    return;
  }
  // 使用正则表达式匹配链接中的UUID (课程ID)
  const uuidRegex = /\/([0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12})\//i;
  const match = inputUrl.value.match(uuidRegex);

  if (match && match[1]) {
    courseId.value = match[1]; // 成功匹配，更新课程ID
    errorMessage.value = '';   // 清除错误信息
  } else {
    courseId.value = null;     // 匹配失败，清空课程ID
    errorMessage.value = '无法从此链接中识别出有效的课程ID，请检查链接格式。';
  }
};

/**
 * 根据当前课程ID生成实时签到URL
 */
const generateRealtimeUrl = () => {
  if (!courseId.value) return;

  const baseUrl = "https://acm.sztu.edu.cn:40130/hwsys/signin_op";
  const timestamp2 = Date.now();
  const timestamp1 = timestamp2 + timestampOffset;

  qrCodeUrl.value = `${baseUrl}/${courseId.value}/${timestamp1}/${timestamp2}`;
};

// 监听 courseId 的变化
watch(courseId, (newId) => {
  // 停止上一个课程的定时刷新
  if (intervalId) {
    clearInterval(intervalId);
    intervalId = null;
  }
  
  // 如果有了新的有效课程ID
  if (newId) {
    // 1. 立即生成一次，以便马上显示
    generateRealtimeUrl();
    // 2. 启动新的定时器，为新课程持续刷新
    intervalId = setInterval(generateRealtimeUrl, refreshInterval);
  } else {
    // 如果课程ID被清空了，也清空二维码URL
    qrCodeUrl.value = '';
  }
});

// 在组件卸载前，确保清除定时器，防止内存泄漏
onBeforeUnmount(() => {
  if (intervalId) {
    clearInterval(intervalId);
  }
});
</script>

<style scoped>
.signin-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  font-family: sans-serif;
  padding: 25px;
  background-color: #f4f7f6;
  border-radius: 12px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  max-width: 450px;
  margin: 40px auto;
  text-align: center;
}

.input-section {
  width: 100%;
  margin-bottom: 25px;
}

.input-section label {
  display: block;
  margin-bottom: 8px;
  font-weight: bold;
  color: #555;
}

.input-section input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  box-sizing: border-box; /* 保证padding不会撑大宽度 */
}

.input-section button {
  width: 100%;
  padding: 12px;
  margin-top: 10px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  font-size: 16px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.input-section button:hover {
  background-color: #0056b3;
}

.error-message {
  color: #d93025;
  margin-top: 10px;
  font-size: 0.9em;
}

.qrcode-display {
  width: 100%;
}

.qrcode-wrapper {
  margin: 20px 0;
  padding: 20px;
  background: white;
  border-radius: 8px;
  display: inline-block;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

.info {
  margin-top: 15px;
  text-align: left;
  word-break: break-all;
  background-color: #e9ecef;
  padding: 15px;
  border-radius: 5px;
}

.info p {
  margin: 5px 0;
}

.url-text {
  color: #0056b3;
  font-size: 0.9em;
  font-family: 'Courier New', Courier, monospace;
}

.placeholder {
  margin-top: 20px;
  padding: 40px;
  border: 2px dashed #ccc;
  border-radius: 10px;
  color: #777;
}
</style>