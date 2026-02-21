<script setup>
import { ref, computed, onMounted } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'

// URL 输入
const url = ref('')

// 视频格式预设
const formatPreset = ref('bestvideo+bestaudio')
const customFormat = ref('')

// 分辨率选择
const resolution = ref('')

// 字幕选项
const subtitleOptions = ref({
  writeSub: false,
  embedSub: false,
  subLangs: '',
  autoSubs: false
})

// 输出路径
const outputDir = ref('~/Downloads')

// 播放列表选项
const playlistOptions = ref({
  yesPlaylist: true,
  playlistItems: '',
  playlistReverse: false
})

// 其他常用选项
const otherOptions = ref({
  writeDescription: false,
  writeThumbnail: false,
  extractAudio: false,
  audioFormat: 'mp3'
})

// 保存配置到 localStorage
const saveConfig = () => {
  try {
    const config = {
      formatPreset: formatPreset.value,
      customFormat: customFormat.value,
      resolution: resolution.value,
      subtitleOptions: { ...subtitleOptions.value },
      outputDir: outputDir.value,
      playlistOptions: { ...playlistOptions.value },
      otherOptions: { ...otherOptions.value }
    }
    localStorage.setItem('yt-dlp-config', JSON.stringify(config))
    ElMessage.success('配置已保存')
  } catch (e) {
    console.error('保存配置失败:', e)
    ElMessage.error('保存配置失败')
  }
}

// 从 localStorage 加载配置
const loadConfig = () => {
  try {
    const saved = localStorage.getItem('yt-dlp-config')
    if (saved) {
      const config = JSON.parse(saved)
      formatPreset.value = config.formatPreset || 'bestvideo+bestaudio'
      customFormat.value = config.customFormat || ''
      resolution.value = config.resolution || ''
      subtitleOptions.value = { ...subtitleOptions.value, ...config.subtitleOptions }
      outputDir.value = config.outputDir || '~/Downloads'
      playlistOptions.value = { ...playlistOptions.value, ...config.playlistOptions }
      otherOptions.value = { ...otherOptions.value, ...config.otherOptions }
      ElMessage.success('已加载上次的配置')
    }
  } catch (e) {
    console.error('加载配置失败:', e)
  }
}

// 初始化时加载配置
onMounted(() => {
  loadConfig()
})

// 计算生成的命令
const command = computed(() => {
  if (!url.value.trim()) {
    return '请输入视频 URL'
  }

  let cmd = ['yt-dlp']

  // 格式选择
  if (customFormat.value) {
    cmd.push(`-f ${customFormat.value}`)
  } else if (resolution.value) {
    cmd.push(`-f "bestvideo[height<=?${resolution.value}]+bestaudio/best[height<=?${resolution.value}]"`)
  } else if (formatPreset.value) {
    cmd.push(`-f ${formatPreset.value}`)
  }

  // 字幕选项
  if (subtitleOptions.value.writeSub) {
    cmd.push('--write-subs')
  }
  if (subtitleOptions.value.embedSub) {
    cmd.push('--embed-subs')
  }
  if (subtitleOptions.value.subLangs) {
    cmd.push(`--sub-langs ${subtitleOptions.value.subLangs}`)
  }
  if (subtitleOptions.value.autoSubs) {
    cmd.push('--write-auto-subs')
  }

  // 输出路径
  if (outputDir.value) {
    cmd.push(`-P "${outputDir.value}"`)
  }

  // 播放列表选项
  if (!playlistOptions.value.yesPlaylist) {
    cmd.push('--no-playlist')
  }
  if (playlistOptions.value.playlistItems) {
    cmd.push(`--playlist-items ${playlistOptions.value.playlistItems}`)
  }
  if (playlistOptions.value.playlistReverse) {
    cmd.push('--playlist-reverse')
  }

  // 其他选项
  if (otherOptions.value.writeDescription) {
    cmd.push('--write-description')
  }
  if (otherOptions.value.writeThumbnail) {
    cmd.push('--write-thumbnail')
  }
  if (otherOptions.value.extractAudio) {
    cmd.push('-x')
    cmd.push(`--audio-format ${otherOptions.value.audioFormat}`)
  }

  // URL
  cmd.push(`"${url.value.trim()}"`)

  return cmd.join(' ')
})

// 复制命令到剪贴板
const copyCommand = async () => {
  try {
    await navigator.clipboard.writeText(command.value)
    ElMessage.success('命令已复制到剪贴板！')
  } catch (err) {
    ElMessage.error('复制失败，请手动复制')
  }
}

// 清空所有选项
const resetOptions = () => {
  ElMessageBox.confirm(
    '确定要重置所有选项吗？',
    '提示',
    {
      confirmButtonText: '确定',
      cancelButtonText: '取消',
      type: 'warning',
    }
  ).then(() => {
    url.value = ''
    formatPreset.value = 'bestvideo+bestaudio'
    customFormat.value = ''
    resolution.value = ''
    subtitleOptions.value = {
      writeSub: false,
      embedSub: false,
      subLangs: '',
      autoSubs: false
    }
    outputDir.value = '~/Downloads'
    playlistOptions.value = {
      yesPlaylist: true,
      playlistItems: '',
      playlistReverse: false
    }
    otherOptions.value = {
      writeDescription: false,
      writeThumbnail: false,
      extractAudio: false,
      audioFormat: 'mp3'
    }
    ElMessage.success('已重置所有选项')
  }).catch(() => {})
}
</script>

<template>
  <div class="app-container">
    <div class="header">
      <h1>🎬 yt-dlp 辅助助手</h1>
      <p class="subtitle">轻松生成 yt-dlp 下载命令</p>
      <el-button
        type="success"
        @click="saveConfig"
        style="margin-top: 10px"
      >
        💾 保存配置
      </el-button>
    </div>

    <div class="main-content">
      <!-- 左侧配置区 -->
      <div class="config-panel">
        <!-- 视频地址和路径 -->
        <el-card class="mb-12">
          <template #header>
            <span class="card-title">🔗 视频与路径</span>
          </template>

          <div class="input-group">
            <el-input
              v-model="url"
              placeholder="粘贴视频 URL..."
              clearable
              style="width: 100%"
            >
              <template #prepend>URL</template>
            </el-input>

            <el-input
              v-model="outputDir"
              placeholder="下载保存路径，如: ~/Downloads"
              clearable
              style="width: 100%; margin-top: 10px"
            >
              <template #prepend>路径</template>
            </el-input>
          </div>
        </el-card>

        <!-- 视频质量 -->
        <el-card class="mb-12">
          <template #header>
            <span class="card-title">🎬 视频质量</span>
          </template>

          <el-form label-position="top" :label-width="'80px'">
            <el-row :gutter="10">
              <el-col :span="12">
                <el-form-item label="预设格式" style="margin-bottom: 10px">
                  <el-select
                    v-model="formatPreset"
                    :disabled="!!customFormat || !!resolution"
                    style="width: 100%"
                  >
                    <el-option label="最佳质量" value="best" />
                    <el-option label="最佳视频+音频" value="bestvideo+bestaudio" />
                    <el-option label="MP4 格式" value="mp4" />
                    <el-option label="WebM 格式" value="webm" />
                    <el-option label="最低质量" value="worst" />
                  </el-select>
                </el-form-item>
              </el-col>
              <el-col :span="12">
                <el-form-item label="分辨率限制" style="margin-bottom: 10px">
                  <el-select
                    v-model="resolution"
                    :disabled="!!customFormat"
                    clearable
                    placeholder="不限制"
                    style="width: 100%"
                  >
                    <el-option label="4K (2160p)" value="2160" />
                    <el-option label="2K (1440p)" value="1440" />
                    <el-option label="Full HD (1080p)" value="1080" />
                    <el-option label="HD (720p)" value="720" />
                    <el-option label="SD (480p)" value="480" />
                    <el-option label="360p" value="360" />
                  </el-select>
                </el-form-item>
              </el-col>
            </el-row>

            <el-form-item label="自定义格式" style="margin-bottom: 0">
              <el-input
                v-model="customFormat"
                placeholder='如: bestvideo[height<=1080]+bestaudio'
                clearable
              />
            </el-form-item>
          </el-form>
        </el-card>

        <!-- 字幕选项 -->
        <el-card class="mb-12">
          <template #header>
            <span class="card-title">📝 字幕选项</span>
          </template>

          <el-space direction="vertical" :size="10" style="width: 100%">
            <el-space wrap :size="8">
              <el-checkbox v-model="subtitleOptions.writeSub" border>
                下载字幕
              </el-checkbox>
              <el-checkbox v-model="subtitleOptions.embedSub" border>
                嵌入字幕
              </el-checkbox>
              <el-checkbox v-model="subtitleOptions.autoSubs" border>
                自动字幕
              </el-checkbox>
            </el-space>

            <el-input
              v-model="subtitleOptions.subLangs"
              placeholder="字幕语言，如: zh-Hans,en,ja"
              clearable
            >
              <template #prepend>语言</template>
            </el-input>
          </el-space>
        </el-card>

        <!-- 其他选项 -->
        <el-card class="mb-12">
          <template #header>
            <span class="card-title">⚙️ 其他选项</span>
          </template>

          <el-space wrap :size="8">
            <el-checkbox v-model="otherOptions.writeDescription" border>
              写入描述
            </el-checkbox>
            <el-checkbox v-model="otherOptions.writeThumbnail" border>
              下载缩略图
            </el-checkbox>
            <el-checkbox v-model="otherOptions.extractAudio" border>
              仅提取音频
            </el-checkbox>
          </el-space>

          <el-select
            v-if="otherOptions.extractAudio"
            v-model="otherOptions.audioFormat"
            placeholder="音频格式"
            style="width: 100%; margin-top: 10px"
          >
            <el-option label="MP3" value="mp3" />
            <el-option label="M4A" value="m4a" />
            <el-option label="WAV" value="wav" />
            <el-option label="FLAC" value="flac" />
            <el-option label="Opus" value="opus" />
          </el-select>
        </el-card>

        <!-- 播放列表 -->
        <el-card class="mb-12">
          <template #header>
            <span class="card-title">📋 播放列表</span>
          </template>

          <el-space direction="vertical" :size="10" style="width: 100%">
            <el-space wrap :size="8">
              <el-checkbox v-model="playlistOptions.yesPlaylist" border>
                下载整个播放列表
              </el-checkbox>
              <el-checkbox v-model="playlistOptions.playlistReverse" border>
                反向下载
              </el-checkbox>
            </el-space>

            <el-input
              v-model="playlistOptions.playlistItems"
              placeholder="下载范围，如: 1-5, 8, 10-15"
              clearable
            >
              <template #prepend>范围</template>
            </el-input>
          </el-space>
        </el-card>

        <!-- 操作按钮 -->
        <el-button
          type="info"
          @click="resetOptions"
          style="width: 100%"
        >
          重置所有选项
        </el-button>
      </div>

      <!-- 右侧命令预览区 -->
      <div class="command-panel">
        <el-card shadow="always">
          <template #header>
            <div class="command-header">
              <span class="card-title">⚡ 生成的命令</span>
              <el-button
                type="primary"
                @click="copyCommand"
              >
                复制命令
              </el-button>
            </div>
          </template>

          <div class="command-display">
            <pre>{{ command }}</pre>
          </div>

          <el-divider />

          <el-alert
            title="使用提示"
            type="info"
            :closable="false"
            show-icon
          >
            <ul class="tips-list">
              <li>粘贴视频 URL 后，命令会自动生成</li>
              <li>使用分辨率限制可以快速控制下载质量</li>
              <li>自定义格式支持完整的 yt-dlp 格式语法</li>
              <li>输出模板支持 %(title)s、%(uploader)s 等变量</li>
            </ul>
          </el-alert>
        </el-card>
      </div>
    </div>
  </div>
</template>

<style scoped>
.app-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 20px;
}

.header {
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
  text-align: center;
  padding: 20px;
  border-radius: 12px;
  margin-bottom: 20px;
}

.header h1 {
  margin: 0;
  font-size: 2em;
  color: #2c3e50;
  font-weight: 600;
}

.subtitle {
  margin: 8px 0 0 0;
  font-size: 1em;
  color: #666;
}

.main-content {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
  max-width: 1600px;
  margin: 0 auto;
}

.config-panel {
  display: flex;
  flex-direction: column;
}

.mb-12 {
  margin-bottom: 12px;
}

.card-title {
  font-weight: 600;
  font-size: 15px;
}

/* 紧凑模式样式 */
:deep(.el-card__body) {
  padding: 15px;
}

:deep(.el-form-item) {
  margin-bottom: 10px;
}

:deep(.el-form-item__label) {
  font-size: 13px;
  margin-bottom: 4px;
}

:deep(.el-checkbox.is-bordered) {
  padding: 6px 10px;
  font-size: 13px;
}

:deep(.el-input__inner) {
  font-size: 14px;
}

:deep(.el-select .el-input__inner) {
  font-size: 14px;
}

.command-panel {
  position: sticky;
  top: 20px;
  height: fit-content;
}

.command-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-weight: 600;
  font-size: 18px;
}

.command-display {
  background: #1e1e1e;
  border-radius: 8px;
  padding: 15px;
  margin-bottom: 12px;
  min-height: 100px;
  max-height: 350px;
  overflow-y: auto;
}

.command-display pre {
  margin: 0;
  color: #a9b7c6;
  font-family: 'Monaco', 'Menlo', 'Courier New', monospace;
  font-size: 13px;
  line-height: 1.5;
  white-space: pre-wrap;
  word-break: break-all;
}

.tips-list {
  margin: 10px 0 0 0;
  padding-left: 20px;
  font-size: 14px;
  line-height: 1.8;
}

.tips-list li {
  margin-bottom: 5px;
}

/* 自定义滚动条 */
.command-display::-webkit-scrollbar {
  width: 8px;
}

.command-display::-webkit-scrollbar-track {
  background: #2d2d2d;
  border-radius: 4px;
}

.command-display::-webkit-scrollbar-thumb {
  background: #555;
  border-radius: 4px;
}

.command-display::-webkit-scrollbar-thumb:hover {
  background: #666;
}

/* 响应式设计 */
@media (max-width: 968px) {
  .main-content {
    grid-template-columns: 1fr;
  }

  .command-panel {
    position: static;
  }

  .header h1 {
    font-size: 1.8em;
  }

  .command-header {
    flex-direction: column;
    gap: 15px;
    align-items: stretch;
  }

  .command-header .el-button {
    width: 100%;
  }
}
</style>
