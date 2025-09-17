<template>
  <div class="sign-container">
    <h1>二维码签到</h1>

    <div class="upload-section">
      <label for="qr-upload" class="upload-label">上传二维码图片</label>
      <input id="qr-upload" type="file" @change="handleFileChange" accept="image/*" />
      <canvas ref="canvas" style="display: none;"></canvas>
    </div>

    <div class="input-section">
      <label for="qr-link">或者输入二维码链接</label>
      <input id="qr-link" type="text" v-model="manualUrl" placeholder="在此处粘贴链接" />
    </div>

    <div v-if="baseUrl" class="qr-display">
      <h2>生成的二维码</h2>
      <p>原始链接: {{ baseUrl }}</p>
      <p>新链接: {{ newUrl }}</p>
      <img :src="generatedQrCode" alt="Generated QR Code" />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, watch, onUnmounted } from 'vue';
import jsQR from 'jsqr';
import QRCode from 'qrcode';

const canvas = ref<HTMLCanvasElement | null>(null);
const manualUrl = ref('');
const baseUrl = ref('');
const newUrl = ref('');
const generatedQrCode = ref('');

let intervalId: number | null = null;

const handleFileChange = (event: Event) => {
  const target = event.target as HTMLInputElement;
  if (target.files && target.files[0]) {
    const file = target.files[0];
    const reader = new FileReader();
    reader.onload = (e) => {
      const img = new Image();
      img.onload = () => {
        const ctx = canvas.value?.getContext('2d');
        if (ctx && canvas.value) {
          canvas.value.width = img.width;
          canvas.value.height = img.height;
          ctx.drawImage(img, 0, 0, img.width, img.height);
          const imageData = ctx.getImageData(0, 0, img.width, img.height);
          const code = jsQR(imageData.data, imageData.width, imageData.height);
          if (code) {
            baseUrl.value = code.data;
          } else {
            alert('未识别到二维码');
          }
        }
      };
      img.src = e.target?.result as string;
    };
    reader.readAsDataURL(file);
  }
};

const generateQRCode = async (url: string) => {
  try {
    const qrCodeUrl = await QRCode.toDataURL(url);
    generatedQrCode.value = qrCodeUrl;
  } catch (err) {
    console.error('生成二维码失败:', err);
  }
};

const updateUrlAndGenerateQr = () => {
  if (!baseUrl.value) return;

  const parts = baseUrl.value.split('/');
  if (parts.length >= 3) {
    parts[parts.length - 2] = Date.now().toString();
    const updatedUrl = parts.join('/');
    newUrl.value = updatedUrl;
    generateQRCode(updatedUrl);
  }
};

watch(manualUrl, (newVal) => {
  if (newVal) {
    baseUrl.value = newVal;
  }
});

watch(baseUrl, (newVal) => {
  if (intervalId) {
    clearInterval(intervalId);
    intervalId = null;
  }
  if (newVal) {
    updateUrlAndGenerateQr();
    intervalId = window.setInterval(updateUrlAndGenerateQr, 500);
  } else {
    newUrl.value = '';
    generatedQrCode.value = '';
  }
});

onUnmounted(() => {
  if (intervalId) {
    clearInterval(intervalId);
  }
});
</script>

<style scoped>
.sign-container {
  max-width: 500px;
  margin: 40px auto;
  padding: 20px;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  text-align: center;
}

h1, h2 {
  color: #333;
}

.upload-section, .input-section {
  margin-bottom: 25px;
}

.upload-label {
  display: inline-block;
  padding: 10px 15px;
  background-color: #007bff;
  color: white;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.upload-label:hover {
  background-color: #0056b3;
}

input[type="file"] {
  display: none;
}

input[type="text"] {
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  box-sizing: border-box;
  margin-top: 10px;
}

.qr-display {
  margin-top: 20px;
}

.qr-display p {
  word-wrap: break-word;
  background: #f5f5f5;
  padding: 8px;
  border-radius: 4px;
  font-size: 0.9em;
  color: #555;
}

.qr-display img {
  margin-top: 15px;
  border: 1px solid #eee;
  padding: 5px;
  border-radius: 5px;
}
</style>