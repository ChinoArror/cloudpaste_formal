<template>
  <div v-if="isOpen" class="fixed inset-0 z-[60] overflow-auto bg-black bg-opacity-50 flex items-center justify-center p-2 sm:p-4 pt-20 sm:pt-4">
    <div
      class="relative w-full max-w-sm sm:max-w-3xl lg:max-w-5xl h-auto min-h-[400px] sm:min-h-[500px] max-h-[85vh] sm:max-h-[80vh] rounded-lg shadow-xl flex flex-col"
      :class="darkMode ? 'bg-gray-800' : 'bg-white'"
    >
      <!-- 弹窗标题栏 -->
      <div class="p-4 flex justify-between items-center border-b" :class="darkMode ? 'border-gray-700' : 'border-gray-200'">
        <h3 class="text-lg font-semibold" :class="darkMode ? 'text-gray-100' : 'text-gray-900'">{{ t("mount.uploadModal.title") }}</h3>
        <button
          @click="closeModal"
          class="p-1 rounded-full transition-colors"
          :class="darkMode ? 'hover:bg-gray-700 text-gray-400 hover:text-gray-200' : 'hover:bg-gray-200 text-gray-500 hover:text-gray-700'"
        >
          <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
          </svg>
        </button>
      </div>

      <!-- 弹窗内容区 -->
      <div class="flex-1 p-4 overflow-y-auto">
        <!-- 上传方式选择 -->
        <div class="mb-4 flex items-center justify-between p-3 rounded-lg" :class="darkMode ? 'bg-gray-700/50' : 'bg-gray-100'">
          <div class="flex items-center">
            <span class="text-sm font-medium mr-2" :class="darkMode ? 'text-gray-300' : 'text-gray-700'">{{ t("mount.uploadModal.uploadMethod") }}</span>
            <div class="flex space-x-4">
              <label class="flex items-center cursor-pointer">
                <input
                  type="radio"
                  name="uploadMethod"
                  value="presigned"
                  v-model="uploadMethod"
                  class="w-4 h-4 text-blue-600 bg-gray-100 border-gray-300 dark:bg-gray-700 dark:border-gray-600"
                  :disabled="isUploading"
                />
                <span class="ml-2 text-sm" :class="darkMode ? 'text-gray-300' : 'text-gray-600'">
                  {{ t("mount.uploadModal.presignedUpload") }}
                  <span class="text-xs px-1 py-0.5 rounded" :class="darkMode ? 'bg-green-900/50 text-green-300' : 'bg-green-100 text-green-700'">
                    {{ t("mount.uploadModal.recommended") }}
                  </span>
                </span>
              </label>
              <label class="flex items-center cursor-pointer">
                <input
                  type="radio"
                  name="uploadMethod"
                  value="direct"
                  v-model="uploadMethod"
                  class="w-4 h-4 text-blue-600 bg-gray-100 border-gray-300 dark:bg-gray-700 dark:border-gray-600"
                  :disabled="isUploading"
                />
                <span class="ml-2 text-sm" :class="darkMode ? 'text-gray-300' : 'text-gray-600'">{{ t("mount.uploadModal.directUpload") }}</span>
              </label>
              <label class="flex items-center cursor-pointer">
                <input
                  type="radio"
                  name="uploadMethod"
                  value="multipart"
                  v-model="uploadMethod"
                  class="w-4 h-4 text-blue-600 bg-gray-100 border-gray-300 dark:bg-gray-700 dark:border-gray-600"
                  :disabled="isUploading"
                />
                <span class="ml-2 text-sm" :class="darkMode ? 'text-gray-300' : 'text-gray-600'">{{ t("mount.uploadModal.multipartUpload") }}</span>
              </label>
            </div>
          </div>
          <div>
            <span
              class="text-xs px-2 py-1 rounded-full cursor-help"
              :class="
                uploadMethod === 'direct'
                  ? darkMode
                    ? 'bg-green-900/30 text-green-300'
                    : 'bg-green-100 text-green-700'
                  : uploadMethod === 'presigned'
                  ? darkMode
                    ? 'bg-blue-900/30 text-blue-300'
                    : 'bg-blue-100 text-blue-700'
                  : darkMode
                  ? 'bg-amber-900/30 text-amber-300'
                  : 'bg-amber-100 text-amber-700'
              "
              :title="
                uploadMethod === 'direct'
                  ? t('mount.uploadModal.directModeDesc')
                  : uploadMethod === 'presigned'
                  ? t('mount.uploadModal.presignedModeDesc')
                  : t('mount.uploadModal.multipartModeDesc')
              "
            >
              {{
                uploadMethod === "direct"
                  ? t("mount.uploadModal.directMode")
                  : uploadMethod === "presigned"
                  ? t("mount.uploadModal.presignedMode")
                  : t("mount.uploadModal.multipartMode")
              }}
            </span>
          </div>
        </div>

        <!-- 文件拖放区域 -->
        <div
          class="drop-zone mb-4 border-2 border-dashed rounded-lg flex flex-col items-center justify-center py-8 px-4 cursor-pointer transition-all duration-300"
          :class="[
            darkMode ? 'border-gray-600 hover:border-gray-500 bg-gray-800/30' : 'border-gray-300 hover:border-gray-400 bg-gray-50',
            isDragging ? (darkMode ? 'border-blue-500 bg-blue-500/10 pulsing-border' : 'border-blue-500 bg-blue-50 pulsing-border') : '',
          ]"
          @dragenter.prevent="onDragOver"
          @dragover.prevent="onDragOver"
          @dragleave.prevent="onDragLeave"
          @drop.prevent="onDrop"
          @click="triggerFileInput"
        >
          <div class="icon-container mb-3">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              class="h-12 w-12 transition-colors duration-300"
              :class="[darkMode ? 'text-gray-400' : 'text-gray-500', isDragging ? (darkMode ? 'text-blue-400' : 'text-blue-500') : '']"
              fill="none"
              viewBox="0 0 24 24"
              stroke="currentColor"
            >
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M7 16a4 4 0 01-.88-7.903A5 5 0 1115.9 6L16 6a5 5 0 011 9.9M15 13l-3-3m0 0l-3 3m3-3v12" />
            </svg>
          </div>
          <div class="text-center">
            <p
              class="text-base font-medium transition-colors duration-300"
              :class="[darkMode ? 'text-gray-300' : 'text-gray-700', isDragging ? (darkMode ? 'text-blue-300' : 'text-blue-700') : '']"
            >
              {{ isDragging ? t("mount.uploadModal.dragDropHere") : t("mount.uploadModal.clickOrDragToUpload") }}
            </p>
            <p class="text-sm mt-1 transition-colors duration-300" :class="[darkMode ? 'text-gray-400' : 'text-gray-500']">
              <span class="px-1.5 py-0.5 rounded text-xs" :class="darkMode ? 'bg-gray-700 text-blue-300' : 'bg-gray-200 text-blue-600'">
                {{ t("mount.uploadModal.multiFileSupport") }}
              </span>
            </p>
            <p class="text-xs mt-2 transition-colors duration-300" :class="darkMode ? 'text-gray-500' : 'text-gray-500'">
              <span class="inline-flex items-center">
                <svg class="h-3 w-3 mr-1" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                    d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2"
                  />
                </svg>
                {{ t("mount.uploadModal.pasteSupport") }}
              </span>
            </p>
          </div>
          <input ref="fileInput" type="file" class="hidden" multiple @change="onFileSelected" />
        </div>

        <!-- 已选文件列表 -->
        <div v-if="selectedFiles.length > 0" class="selected-files mb-4">
          <div class="files-header flex justify-between items-center mb-3">
            <h3 class="text-base font-medium" :class="darkMode ? 'text-gray-200' : 'text-gray-700'">{{ t("mount.uploadModal.selectedFiles", { count: selectedFiles.length }) }}</h3>
            <button
              type="button"
              @click="clearAllFiles"
              class="text-sm px-2 py-1 rounded transition-colors flex items-center"
              :class="darkMode ? 'bg-gray-700 hover:bg-gray-600 text-gray-300' : 'bg-gray-100 hover:bg-gray-200 text-gray-600'"
              :disabled="isUploading"
            >
              <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 mr-1" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"
                />
              </svg>
              {{ t("mount.uploadModal.clearAll") }}
            </button>
          </div>
          <div class="files-list max-h-60 overflow-y-auto">
            <div
              v-for="(file, index) in selectedFiles"
              :key="index"
              class="selected-file mb-3 flex items-center p-3 rounded-md"
              :class="[
                darkMode ? 'bg-gray-700/50' : 'bg-gray-100',
                fileItems[index]?.status === 'error' ? (darkMode ? 'border-l-4 border-red-500' : 'border-l-4 border-red-500') : '',
                fileItems[index]?.status === 'success' ? (darkMode ? 'border-l-4 border-green-500' : 'border-l-4 border-green-500') : '',
                fileItems[index]?.status === 'uploading' ? (darkMode ? 'border-l-4 border-blue-500' : 'border-l-4 border-blue-500') : '',
              ]"
            >
              <div class="file-icon mr-3">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8" :class="darkMode ? 'text-gray-300' : 'text-gray-600'" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                    d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"
                  />
                </svg>
              </div>
              <div class="file-info flex-grow mr-3">
                <div class="font-medium truncate" :class="darkMode ? 'text-white' : 'text-gray-900'">
                  {{ file.name }}
                </div>
                <div class="flex justify-between">
                  <span class="text-sm" :class="darkMode ? 'text-gray-400' : 'text-gray-500'">
                    {{ formatFileSize(file.size) }}
                  </span>

                  <!-- 文件状态显示 -->
                  <span
                    v-if="fileItems[index]"
                    class="text-xs ml-2 px-2 py-0.5 rounded-full"
                    :class="{
                      'bg-yellow-100 text-yellow-800 dark:bg-yellow-900/30 dark:text-yellow-300': fileItems[index].status === 'pending',
                      'bg-blue-100 text-blue-800 dark:bg-blue-900/30 dark:text-blue-300': fileItems[index].status === 'uploading',
                      'bg-green-100 text-green-800 dark:bg-green-900/30 dark:text-green-300': fileItems[index].status === 'success',
                      'bg-red-100 text-red-800 dark:bg-red-900/30 dark:text-red-300': fileItems[index].status === 'error',
                    }"
                  >
                    {{
                      fileItems[index].status === "pending"
                        ? t("mount.uploadModal.fileStatus.pending")
                        : fileItems[index].status === "uploading"
                        ? t("mount.uploadModal.fileStatus.uploading", { progress: fileItems[index].progress })
                        : fileItems[index].status === "success"
                        ? t("mount.uploadModal.fileStatus.success")
                        : fileItems[index].status === "error"
                        ? t("mount.uploadModal.fileStatus.error")
                        : ""
                    }}
                  </span>
                </div>

                <!-- 单个文件进度条 -->
                <div v-if="fileItems[index]?.status === 'uploading'" class="w-full bg-gray-200 rounded-full h-1.5 mt-1 dark:bg-gray-700">
                  <div
                    class="h-1.5 rounded-full transition-all duration-200"
                    :class="fileItems[index].progress >= 95 ? 'bg-green-500' : 'bg-blue-500'"
                    :style="{ width: `${fileItems[index].progress}%` }"
                  ></div>
                </div>
                <!-- 错误信息 -->
                <div v-if="fileItems[index]?.status === 'error' && fileItems[index]?.message" class="text-xs mt-1 text-red-500">
                  {{ fileItems[index].message }}
                </div>
              </div>
              <!-- 取消上传按钮，仅在上传状态显示 -->
              <button
                v-if="fileItems[index]?.status === 'uploading'"
                type="button"
                @click="cancelSingleUpload(index)"
                class="p-1 rounded-full hover:bg-opacity-20 transition-colors mr-1"
                :class="darkMode ? 'hover:bg-red-900/60 text-gray-400 hover:text-red-300' : 'hover:bg-red-100 text-gray-500 hover:text-red-500'"
                :title="t('mount.uploadModal.cancelSingleUpload')"
              >
                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 19V5M5 12l7-7 7 7" />
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4L20 20" stroke="red" />
                </svg>
              </button>
              <!-- 重试按钮，仅在错误状态显示 -->
              <button
                v-if="fileItems[index]?.status === 'error'"
                type="button"
                @click="handleRetryUpload(index)"
                class="p-1 rounded-full hover:bg-opacity-20 transition-colors mr-1"
                :class="darkMode ? 'hover:bg-gray-600 text-gray-400' : 'hover:bg-gray-200 text-gray-500'"
                :title="t('mount.uploadModal.retryUpload')"
              >
                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                    d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15"
                  />
                </svg>
              </button>
              <!-- 删除按钮 -->
              <button
                type="button"
                @click="clearSelectedFile(index)"
                class="p-1 rounded-full hover:bg-opacity-20 transition-colors"
                :class="darkMode ? 'hover:bg-gray-600 text-gray-400' : 'hover:bg-gray-200 text-gray-500'"
                :title="t('mount.uploadModal.removeFile')"
                :disabled="fileItems[index]?.status === 'uploading'"
              >
                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                </svg>
              </button>
            </div>
          </div>
        </div>

        <!-- 总上传进度 -->
        <div v-if="isUploading" class="mb-4">
          <div class="flex justify-between items-center mb-1">
            <span class="text-sm font-medium" :class="darkMode ? 'text-gray-300' : 'text-gray-700'">{{ t("mount.uploadModal.totalProgress") }}</span>
            <span class="text-sm" :class="darkMode ? 'text-gray-400' : 'text-gray-600'">{{ totalProgress }}%</span>
          </div>
          <div class="w-full bg-gray-200 rounded-full h-2.5 dark:bg-gray-700">
            <div
              class="h-2.5 rounded-full transition-all duration-200 progress-stripes animate-progress-stripes"
              :class="totalProgress >= 95 ? 'bg-green-500' : 'bg-blue-500'"
              :style="{ width: `${totalProgress}%` }"
            ></div>
          </div>
          <div class="flex justify-between items-center mt-1">
            <span class="text-xs" :class="darkMode ? 'text-gray-400' : 'text-gray-500'"> {{ t("mount.uploadModal.uploadSpeed") }} {{ uploadSpeed }} </span>
            <span class="text-xs" :class="darkMode ? 'text-gray-400' : 'text-gray-500'">
              {{ currentUploadInfo }}
            </span>
          </div>
        </div>

        <!-- 上传消息 -->
        <div
          v-if="message && message.content"
          class="mb-4 p-3 rounded-md text-sm"
          :class="
            message.type === 'error'
              ? darkMode
                ? 'bg-red-900/40 border border-red-800 text-red-200'
                : 'bg-red-50 border border-red-200 text-red-800'
              : message.type === 'info'
              ? darkMode
                ? 'bg-blue-900/40 border border-blue-800 text-blue-200'
                : 'bg-blue-50 border border-blue-200 text-blue-800'
              : darkMode
              ? 'bg-green-900/40 border border-green-800 text-green-200'
              : 'bg-green-50 border border-green-200 text-green-800'
          "
        >
          <div class="flex items-center">
            <!-- 成功图标 -->
            <svg
              v-if="message.type === 'success'"
              class="h-5 w-5 mr-2"
              :class="darkMode ? 'text-green-300' : 'text-green-500'"
              fill="none"
              viewBox="0 0 24 24"
              stroke="currentColor"
            >
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" />
            </svg>

            <!-- 信息图标 -->
            <svg
              v-else-if="message.type === 'info'"
              class="h-5 w-5 mr-2"
              :class="darkMode ? 'text-blue-300' : 'text-blue-500'"
              fill="none"
              viewBox="0 0 24 24"
              stroke="currentColor"
            >
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
            </svg>

            <!-- 错误图标 -->
            <svg v-else class="h-5 w-5 mr-2" :class="darkMode ? 'text-red-300' : 'text-red-500'" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
            </svg>

            {{ message.content }}
          </div>
        </div>
      </div>

      <!-- 弹窗底部按钮 -->
      <div class="p-4 flex justify-end items-center gap-3 border-t" :class="darkMode ? 'border-gray-700' : 'border-gray-200'">
        <button
          @click="closeModal"
          class="px-4 py-2 rounded-md transition-colors"
          :class="darkMode ? 'text-gray-300 hover:bg-gray-700' : 'text-gray-700 hover:bg-gray-100'"
          :disabled="isUploading"
        >
          {{ t("mount.uploadModal.cancel") }}
        </button>
        <button v-if="isUploading" @click="cancelUpload" class="px-4 py-2 rounded-md text-white transition-colors bg-red-500 hover:bg-red-600">
          {{ t("mount.uploadModal.cancelUpload") }}
        </button>
        <button
          v-else
          @click="submitUpload"
          class="px-4 py-2 rounded-md text-white transition-colors"
          :class="darkMode ? 'bg-blue-600 hover:bg-blue-700' : 'bg-blue-500 hover:bg-blue-600'"
          :disabled="!hasFilesToUpload"
        >
          {{ t("mount.uploadModal.startUpload") }}
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch, onBeforeUnmount } from "vue";
import { useI18n } from "vue-i18n";
import { useFileUpload } from "../../../../composables/index.js";

const { t } = useI18n();

// 使用文件上传 Composable
const uploadModal = useFileUpload();

// 解构状态和方法
const {
  selectedFiles,
  fileItems,
  isDragging,
  message,
  isUploading,
  totalProgress,
  uploadSpeed,
  hasFilesToUpload,
  currentUploadInfo,
  handleDrop,
  handleFileSelect,
  handlePaste,
  removeFile,
  clearAllFiles,
  startUpload,
  cancelUpload,
  cancelSingleUpload,
  retryUpload,
} = uploadModal;

// 组件属性
const props = defineProps({
  isOpen: {
    type: Boolean,
    default: false,
  },
  darkMode: {
    type: Boolean,
    default: false,
  },
  currentPath: {
    type: String,
    required: true,
  },
  isAdmin: {
    type: Boolean,
    default: false,
  },
});

// 事件
const emit = defineEmits(["close", "upload-success", "upload-error"]);

// 组件特有的状态
const fileInput = ref(null);
const uploadMethod = ref("presigned");

// 包装函数，适配模板中的方法调用
const onDrop = handleDrop;
const onFileSelected = handleFileSelect;
const clearSelectedFile = removeFile;

// 重试上传包装函数
const handleRetryUpload = async (index) => {
  await retryUpload(index, props.currentPath, uploadMethod.value, props.isAdmin);
};

// 生命周期钩子
onMounted(async () => {
  try {
    // 添加全局粘贴事件监听器
    window.addEventListener("paste", handlePasteWrapper);
  } catch (error) {
    console.error("获取最大上传大小失败:", error);
  }
});

// 组件卸载时移除事件监听器
onBeforeUnmount(() => {
  window.removeEventListener("paste", handlePasteWrapper);
});

// 包装粘贴事件处理（添加弹窗状态检查）
const handlePasteWrapper = (event) => {
  // 如果弹窗未打开，不处理粘贴事件
  if (!props.isOpen) return;

  // 调用 composable 中的处理函数
  handlePaste(event);
};

// 方法
const closeModal = () => {
  if (isUploading.value) {
    if (confirm(t("mount.uploadModal.confirmCancelUpload"))) {
      cancelUpload();
      emit("close");
    }
  } else {
    clearAllFiles();
    emit("close");
  }
};

// 拖拽处理
const onDragOver = () => {
  isDragging.value = true;
};

const onDragLeave = (event) => {
  // 只有当鼠标离开拖拽区域而不是其内部元素时才重置
  const rect = event.currentTarget.getBoundingClientRect();
  const x = event.clientX;
  const y = event.clientY;

  // 判断鼠标是否真的离开了整个区域
  if (x <= rect.left || x >= rect.right || y <= rect.top || y >= rect.bottom) {
    isDragging.value = false;
  }
};

// 点击触发文件选择
const triggerFileInput = () => {
  fileInput.value.click();
};

// 导入统一的工具函数
import { formatFileSize } from "../../../../utils/fileUtils.js";

// 上传文件（使用新的 composable）
const submitUpload = async () => {
  const result = await startUpload(props.currentPath, uploadMethod.value, props.isAdmin);

  if (result.success) {
    emit("upload-success", {
      count: result.uploadResults.length,
      results: result.uploadResults,
    });
  } else {
    emit("upload-error", new Error(result.errors?.[0]?.error || "Upload failed"));
  }
};
</script>

<style scoped>
/* 脉动边框动画 */
@keyframes pulseBorder {
  0% {
    border-color: rgba(59, 130, 246, 0.5); /* 淡蓝色 */
  }
  50% {
    border-color: rgba(59, 130, 246, 1); /* 全蓝色 */
  }
  100% {
    border-color: rgba(59, 130, 246, 0.5); /* 淡蓝色 */
  }
}

/* 进度条条纹动画 */
@keyframes progressStripes {
  0% {
    background-position: 0 0;
  }
  100% {
    background-position: 30px 0;
  }
}

.pulsing-border {
  animation: pulseBorder 1.5s ease-in-out infinite;
}

/* 提升动画性能 */
.drop-zone {
  will-change: border-color, background-color, transform;
  transform: translateZ(0);
}

/* 拖动元素进入时的缩放效果 */
.drop-zone.pulsing-border {
  transform: scale(1.01);
  transition: transform 0.3s ease;
}

/* 进度条条纹样式 */
.progress-stripes {
  background-image: linear-gradient(
    45deg,
    rgba(255, 255, 255, 0.15) 25%,
    transparent 25%,
    transparent 50%,
    rgba(255, 255, 255, 0.15) 50%,
    rgba(255, 255, 255, 0.15) 75%,
    transparent 75%,
    transparent
  );
  background-size: 30px 30px;
}

/* 进度条条纹动画 */
.animate-progress-stripes {
  animation: progressStripes 1s linear infinite;
}
</style>
