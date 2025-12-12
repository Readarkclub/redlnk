<template>
  <!-- 图片画廊模态框 -->
  <div v-if="visible && record" class="modal-fullscreen" @click="$emit('close')">
    <div class="modal-body" @click.stop>
      <!-- 头部区域 -->
      <div class="modal-header">
        <div style="flex: 1;">
          <!-- 标题区域 -->
          <div class="title-section">
            <h3
              class="modal-title"
              :class="{ 'collapsed': !titleExpanded && record.title.length > 80 }"
            >
              {{ record.title }}
            </h3>
            <button
              v-if="record.title.length > 80"
              class="title-expand-btn"
              @click="titleExpanded = !titleExpanded"
            >
              {{ titleExpanded ? '收起' : '展开' }}
            </button>
          </div>

          <div class="modal-meta">
            <span>{{ record.outline.pages.length }} 张图片 · {{ formattedDate }}</span>
            <button
              class="view-outline-btn"
              @click="$emit('showOutline')"
              title="查看完整大纲"
            >
              <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path>
                <polyline points="14 2 14 8 20 8"></polyline>
                <line x1="16" y1="13" x2="8" y2="13"></line>
                <line x1="16" y1="17" x2="8" y2="17"></line>
              </svg>
              查看大纲
            </button>
          </div>
        </div>

        <div class="header-actions">
          <button class="btn download-btn" @click="$emit('downloadAll')">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path>
              <polyline points="7 10 12 15 17 10"></polyline>
              <line x1="12" y1="15" x2="12" y2="3"></line>
            </svg>
            打包下载
          </button>
          <button class="close-icon" @click="$emit('close')">×</button>
        </div>
      </div>

      <!-- 图片网格 -->
      <div class="modal-gallery-grid">
        <div
          v-for="(img, idx) in record.images.generated"
          :key="idx"
          class="modal-img-item"
        >
          <div
            class="modal-img-preview"
            v-if="img"
            :class="{ 'regenerating': regeneratingImages.has(idx) }"
          >
            <img
              :src="`/redink-api/images/${record.images.task_id}/${img}`"
              loading="lazy"
              decoding="async"
            />
            <div class="modal-img-overlay">
              <button
                class="modal-overlay-btn"
                @click="$emit('regenerate', idx)"
                :disabled="regeneratingImages.has(idx)"
              >
                <svg class="regenerate-icon" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M23 4v6h-6"></path>
                  <path d="M20.49 15a9 9 0 1 1-2.12-9.36L23 10"></path>
                </svg>
                {{ regeneratingImages.has(idx) ? '重绘中...' : '重新生成' }}
              </button>
            </div>
          </div>
          <div class="placeholder" v-else>Waiting...</div>

          <div class="img-footer">
            <span>Page {{ idx + 1 }}</span>
            <span
              v-if="img"
              class="download-link"
              @click="$emit('download', img, idx)"
            >
              下载
            </span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

/**
 * 图片画廊模态框组件
 *
 * 功能：
 * - 展示历史记录的所有生成图片
 * - 支持重新生成单张图片
 * - 支持下载单张/全部图片
 * - 可展开查看完整大纲
 */

// 定义记录类型
interface ViewingRecord {
  id: string
  title: string
  updated_at: string
  outline: {
    raw: string
    pages: Array<{ type: string; content: string }>
  }
  images: {
    task_id: string
    generated: string[]
  }
}

// 定义 Props
const props = defineProps<{
  visible: boolean
  record: ViewingRecord | null
  regeneratingImages: Set<number>
}>()

// 定义 Emits
defineEmits<{
  (e: 'close'): void
  (e: 'showOutline'): void
  (e: 'downloadAll'): void
  (e: 'download', filename: string, index: number): void
  (e: 'regenerate', index: number): void
}>()

// 标题展开状态
const titleExpanded = ref(false)

// 格式化日期
const formattedDate = computed(() => {
  if (!props.record) return ''
  const d = new Date(props.record.updated_at)
  return `${d.getMonth() + 1}/${d.getDate()}`
})
</script>

<style scoped>
/* 全屏模态框遮罩 */
.modal-fullscreen {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.9);
  z-index: 999;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 40px;
}

/* 模态框主体 */
.modal-body {
  background: white;
  width: 100%;
  max-width: 1000px;
  height: 90vh;
  border-radius: 16px;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

/* 头部区域 */
.modal-header {
  padding: 20px;
  border-bottom: 1px solid #eee;
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  flex-shrink: 0;
  gap: 20px;
}

/* 标题区域 */
.title-section {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  margin-bottom: 4px;
}

.modal-title {
  flex: 1;
  margin: 0;
  font-size: 18px;
  font-weight: 600;
  line-height: 1.4;
  color: #1a1a1a;
  word-break: break-word;
  transition: max-height 0.3s ease;
}

.modal-title.collapsed {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  text-overflow: ellipsis;
}

.title-expand-btn {
  flex-shrink: 0;
  padding: 2px 8px;
  background: #f0f0f0;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 11px;
  color: #666;
  transition: all 0.2s;
  margin-top: 2px;
}

.title-expand-btn:hover {
  background: var(--primary, #ff2442);
  color: white;
}

/* 元信息 */
.modal-meta {
  font-size: 12px;
  color: #999;
  display: flex;
  align-items: center;
  gap: 12px;
  margin-top: 8px;
}

/* 查看大纲按钮 */
.view-outline-btn {
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 4px 10px;
  background: white;
  border: 1px solid #dee2e6;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
  color: #495057;
  transition: all 0.2s;
}

.view-outline-btn:hover {
  background: var(--primary, #ff2442);
  color: white;
  border-color: var(--primary, #ff2442);
}

/* 头部操作区 */
.header-actions {
  display: flex;
  gap: 12px;
  align-items: center;
}

.download-btn {
  padding: 8px 16px;
  font-size: 14px;
  display: flex;
  align-items: center;
  gap: 6px;
}

.close-icon {
  background: none;
  border: none;
  font-size: 24px;
  cursor: pointer;
  color: #666;
  padding: 0;
  line-height: 1;
}

.close-icon:hover {
  color: #333;
}

/* 图片网格 */
.modal-gallery-grid {
  flex: 1;
  overflow-y: auto;
  padding: 20px;
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 20px;
}

/* 单个图片项 */
.modal-img-item {
  display: flex;
  flex-direction: column;
}

/* 图片预览容器 */
.modal-img-preview {
  position: relative;
  width: 100%;
  aspect-ratio: 3/4;
  overflow: hidden;
  border-radius: 8px;
  contain: layout style paint;
}

.modal-img-preview img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

/* 悬浮遮罩 */
.modal-img-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  transition: opacity 0.15s ease-out;
  pointer-events: none;
  will-change: opacity;
}

.modal-img-preview:hover .modal-img-overlay {
  opacity: 1;
  pointer-events: auto;
}

/* 重绘中状态 */
.modal-img-preview.regenerating .modal-img-overlay {
  opacity: 1;
  pointer-events: auto;
}

.modal-img-preview.regenerating .regenerate-icon {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

/* 遮罩层按钮 */
.modal-overlay-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 8px 16px;
  background: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 13px;
  color: #333;
  transition: background-color 0.2s, color 0.2s, transform 0.1s;
  will-change: transform;
}

.modal-overlay-btn:hover {
  background: var(--primary, #ff2442);
  color: white;
  transform: scale(1.05);
}

.modal-overlay-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
  transform: none;
}

/* 占位符 */
.placeholder {
  width: 100%;
  aspect-ratio: 3/4;
  background: #f5f5f5;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #999;
  font-size: 14px;
}

/* 图片底部信息 */
.img-footer {
  margin-top: 8px;
  display: flex;
  justify-content: space-between;
  font-size: 12px;
  color: #666;
}

.download-link {
  cursor: pointer;
  color: var(--primary, #ff2442);
  transition: opacity 0.2s;
}

.download-link:hover {
  opacity: 0.7;
}

/* 响应式 */
@media (max-width: 768px) {
  .modal-fullscreen {
    padding: 20px;
  }

  .modal-gallery-grid {
    grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
    gap: 12px;
    padding: 12px;
  }
}
</style>
