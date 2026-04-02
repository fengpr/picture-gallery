<template>
  <div id="pictureDetailPage" class="animate-fade-in">
    <a-row :gutter="[16, 16]">
      <!-- 图片预览 -->
      <a-col :sm="24" :md="16" :xl="18">
        <a-card class="glass-card preview-card">
          <template #title>
            <span class="glow-title">图片预览</span>
          </template>
          <div class="image-container">
            <a-image :src="picture.url" class="main-image" />
          </div>
        </a-card>
      </a-col>
      <!-- 图片信息区域 -->
      <a-col :sm="24" :md="8" :xl="6">
        <a-card class="glass-card info-card">
          <template #title>
            <span class="glow-title">图片信息</span>
          </template>
          <a-descriptions :column="1" class="tech-descriptions">
            <a-descriptions-item label="作者">
              <a-space>
                <a-avatar :size="24" :src="picture.user?.userAvatar" class="tech-avatar" />
                <div class="user-name">{{ picture.user?.userName }}</div>
              </a-space>
            </a-descriptions-item>
            <a-descriptions-item label="名称">
              <span class="info-value">{{ picture.name ?? '未命名' }}</span>
            </a-descriptions-item>
            <a-descriptions-item label="简介">
              <span class="info-value">{{ picture.introduction ?? '-' }}</span>
            </a-descriptions-item>
            <a-descriptions-item label="分类">
              <a-tag color="blue" class="tech-tag">{{ picture.category ?? '默认' }}</a-tag>
            </a-descriptions-item>
            <a-descriptions-item label="标签">
              <a-space wrap>
                <a-tag v-for="tag in picture.tags" :key="tag" class="tech-tag">
                  {{ tag }}
                </a-tag>
              </a-space>
            </a-descriptions-item>
            <a-descriptions-item label="格式">
              <span class="info-value">{{ picture.picFormat ?? '-' }}</span>
            </a-descriptions-item>
            <a-descriptions-item label="分辨率">
              <span class="info-value">{{ picture.picWidth }} x {{ picture.picHeight }}</span>
            </a-descriptions-item>
            <a-descriptions-item label="大小">
              <span class="info-value">{{ formatSize(picture.picSize) }}</span>
            </a-descriptions-item>
            <a-descriptions-item label="主色调">
              <a-space>
                <span class="info-value">{{ picture.picColor ?? '-' }}</span>
                <div
                  v-if="picture.picColor"
                  class="color-box"
                  :style="{
                    backgroundColor: toHexColor(picture.picColor),
                  }"
                />
              </a-space>
            </a-descriptions-item>
          </a-descriptions>
          <!-- 图片操作 -->
          <div class="action-bar">
            <a-space wrap direction="vertical" style="width: 100%">
              <a-button type="primary" class="flow-button action-btn" @click="doDownload">
                免费下载
                <template #icon>
                  <DownloadOutlined />
                </template>
              </a-button>
              <a-button :icon="h(ShareAltOutlined)" class="action-btn secondary-btn" @click="doShare">
                分享
              </a-button>
              <a-flex gap="8">
                <a-button v-if="canEdit" :icon="h(EditOutlined)" class="flex-btn" @click="doEdit">
                  编辑
                </a-button>
                <a-button v-if="canDelete" :icon="h(DeleteOutlined)" danger class="flex-btn" @click="doDelete">
                  删除
                </a-button>
              </a-flex>
            </a-space>
          </div>
        </a-card>
      </a-col>
    </a-row>
    <ShareModal ref="shareModalRef" :link="shareLink" />
  </div>
</template>

<style scoped>
#pictureDetailPage {
  padding: 24px;
}

.preview-card, .info-card {
  height: 100%;
}

.image-container {
  display: flex;
  justify-content: center;
  align-items: center;
  background: rgba(0, 0, 0, 0.2);
  border-radius: 8px;
  overflow: hidden;
  min-height: 400px;
}

.main-image {
  max-height: 600px;
  object-fit: contain;
}

.tech-descriptions :deep(.ant-descriptions-item-label) {
  color: rgba(255, 255, 255, 0.75) !important;
}

.info-value {
  color: #fff;
}

.user-name {
  color: #fff;
  font-weight: 500;
}

.tech-avatar {
  border: 1px solid rgba(22, 93, 255, 0.3);
  box-shadow: 0 0 8px rgba(22, 93, 255, 0.2);
}

.tech-tag {
  background: rgba(22, 93, 255, 0.1);
  border: 1px solid rgba(22, 93, 255, 0.2);
  color: #165DFF;
}

.color-box {
  width: 16px;
  height: 16px;
  border-radius: 4px;
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.action-bar {
  margin-top: 24px;
  padding-top: 24px;
  border-top: 1px solid rgba(255, 255, 255, 0.05);
}

.action-btn {
  width: 100%;
  height: 40px;
  font-weight: 600;
}

.secondary-btn {
  background: rgba(255, 255, 255, 0.05);
  border-color: rgba(255, 255, 255, 0.1);
  color: #fff;
}

.secondary-btn:hover {
  background: rgba(255, 255, 255, 0.1);
  border-color: #165DFF;
  color: #165DFF;
}

.flex-btn {
  flex: 1;
}
</style>

<script setup lang="ts">
import { computed, h, onMounted, ref } from 'vue'
import { deletePictureUsingPost, getPictureVoByIdUsingGet } from '@/api/pictureController.ts'
import { message } from 'ant-design-vue'
import {
  DeleteOutlined,
  DownloadOutlined,
  EditOutlined,
  ShareAltOutlined,
} from '@ant-design/icons-vue'
import { useRouter } from 'vue-router'
import { downloadImage, formatSize, toHexColor } from '@/utils'
import ShareModal from '@/components/ShareModal.vue'
import { SPACE_PERMISSION_ENUM } from '@/constants/space.ts'

interface Props {
  id: string | number
}

const props = defineProps<Props>()
const picture = ref<API.PictureVO>({})

// 通用权限检查函数
function createPermissionChecker(permission: string) {
  return computed(() => {
    return (picture.value.permissionList ?? []).includes(permission)
  })
}

// 定义权限检查
const canEdit = createPermissionChecker(SPACE_PERMISSION_ENUM.PICTURE_EDIT)
const canDelete = createPermissionChecker(SPACE_PERMISSION_ENUM.PICTURE_DELETE)

// 获取图片详情
const fetchPictureDetail = async () => {
  try {
    const res = await getPictureVoByIdUsingGet({
      id: props.id,
    })
    if (res.data.code === 0 && res.data.data) {
      picture.value = res.data.data
    } else {
      message.error('获取图片详情失败，' + res.data.message)
    }
  } catch (e: any) {
    message.error('获取图片详情失败：' + e.message)
  }
}

onMounted(() => {
  fetchPictureDetail()
})

const router = useRouter()

// 编辑
const doEdit = () => {
  router.push({
    path: '/add_picture',
    query: {
      id: picture.value.id,
      spaceId: picture.value.spaceId,
    },
  })
}

// 删除数据
const doDelete = async () => {
  const id = picture.value.id
  if (!id) {
    return
  }
  const res = await deletePictureUsingPost({ id })
  if (res.data.code === 0) {
    message.success('删除成功')
  } else {
    message.error('删除失败')
  }
}

// 下载图片
const doDownload = () => {
  downloadImage(picture.value.url)
}

// ----- 分享操作 ----
const shareModalRef = ref()
// 分享链接
const shareLink = ref<string>()
// 分享
const doShare = () => {
  shareLink.value = `${window.location.protocol}//${window.location.host}/picture/${picture.value.id}`
  if (shareModalRef.value) {
    shareModalRef.value.openModal()
  }
}
</script>

<style scoped>
#pictureDetailPage {
  margin-bottom: 16px;
}
</style>
