<script setup>
import {
  defineHostBridge,
  hostNotification,
  hostPopup,
  hostThemeParams,
} from '@vbotma/sdk';
import { onMounted, ref, watch, computed } from 'vue';

const iframeRef = ref(null);

const dialog = ref({});
const isVisible = ref(false);
const isIframeVisble = ref(false);

const theme = ref('light');
const lightTheme = {
  bg_color: '#fff',
  text_color: '#000',
  secondary_bg_color: '#ccc',
  button_color: '#42b883',
  accent_text_color: '#42b883',
};

const darkTheme = {
  bg_color: '#000',
  text_color: '#fff',
  secondary_bg_color: '#4c4c4c',
  button_color: '#42b883',
  accent_text_color: '#42b883',
};

const changeTheme = () => {
  if (theme.value == 'light') {
    theme.value = 'dark';
  } else {
    theme.value = 'light';
  }

  if (iframeRef.value) {
    const activeThemeData = theme.value === 'dark' ? darkTheme : lightTheme;
    hostThemeParams.setTheme(activeThemeData);
  }
};

const openDialog = () => {
  isVisible.value = true;
};

const getButtonText = (btn) => {
  if (btn.text) return btn.text;
  if (btn.type === 'cancel') return 'Hủy bỏ'; // Fallback mặc định
  return 'Nút bấm';
};

const handleAction = (btnId, btnType) => {
  console.log(`Bạn đã click vào nút có ID: ${btnId}`);

  hostPopup.close(btnId);
  // Đóng dialog
  isVisible.value = false;
};

const toggleIframe = () => {
  isIframeVisble.value = !isIframeVisble.value;
};

const onIframeLoad = () => {
  // 1. Khởi tạo/Kết nối lại Bridge với iframe mới
  if (iframeRef.value) {
    defineHostBridge(iframeRef.value);
  }

  // 2. Đồng bộ trạng thái theme HIỆN TẠI của Host xuống Mini App
  const activeThemeData = theme.value === 'dark' ? darkTheme : lightTheme;
  hostThemeParams.setTheme(activeThemeData);

  console.log('Đã đồng bộ theme xuống iframe:', theme.value);

  hostNotification.onSend((params) => {
    alert(params.message);
  });

  hostPopup.onOpen((payload) => {
    openDialog();
    dialog.value.buttons = payload.buttons;
    dialog.value.title = payload.title;
    dialog.value.message = payload.message;
    console.log(payload);
  });
};

const themeParams = encodeURIComponent(JSON.stringify(lightTheme));

const iframeSrc = computed(() => {
  return `/mini-app/index.html#/?vbWebAppPlatform=web&vbWebAppVersion=9.2&vbWebAppThemeParams=${themeParams}`;
});
</script>

<template>
  <h2>Host Web</h2>
  <button @click="changeTheme" style="width: fit-content; margin-bottom: 8px">
    Theme: {{ theme !== 'light' ? 'Light' : 'Dark' }}
  </button>
  <button @click="toggleIframe" style="width: fit-content; margin-bottom: 8px">
    Toggle Iframe
  </button>
  <iframe
    v-if="isIframeVisble"
    ref="iframeRef"
    :src="iframeSrc"
    style="width: 100%; border: none; flex: 1"
    sandbox="allow-scripts allow-same-origin allow-forms allow-popups"
    allow="camera; microphone; geolocation"
    @load="onIframeLoad"
  ></iframe>
  <div v-if="isVisible && dialog" class="modal-backdrop">
    <div class="modal-content">
      <h3>{{ dialog.title }}</h3>
      <p>{{ dialog.message }}</p>

      <div class="modal-actions">
        <button
          v-for="btn in dialog.buttons"
          :key="btn.id"
          :id="btn.id"
          :class="['btn', btn.type]"
          @click="handleAction(btn.id, btn.type)"
        >
          {{ getButtonText(btn) }}
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.modal-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background: white;
  padding: 20px 30px;
  border-radius: 8px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
  width: 300px;
  text-align: center;
  animation: fadeIn 0.2s ease-out;
}

.modal-actions {
  margin-top: 20px;
  display: flex;
  justify-content: flex-end;
  gap: 10px; /* Khoảng cách giữa các nút */
}

/* Styling chung cho button */
.btn {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
  transition: background 0.2s;
}

/* Styling dựa trên 'type' trong object */
.btn.default {
  background-color: #42b883; /* Màu xanh Vue */
  color: white;
}
.btn.default:hover {
  background-color: #3aa876;
}
.btn.destructive {
  background-color: rgb(255, 49, 49);
}
.btn.cancel {
  background-color: #f1f1f1;
  color: #333;
}
.btn.cancel:hover {
  background-color: #e0e0e0;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>
