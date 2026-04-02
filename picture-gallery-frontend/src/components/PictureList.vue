<template>
  <div class="picture-list">
    <!-- 图片列表 -->
    <a-list
      :grid="{ gutter: 16, xs: 1, sm: 2, md: 3, lg: 4, xl: 5, xxl: 6 }"
      :data-source="dataList"
      :loading="loading"
    >
      <template #renderItem="{ item: picture }">
        <a-list-item style="padding: 0">
          <!-- 单张图片 -->
          <a-card class="glass-card" hoverable @click="doClickPicture(picture)">
            <template #cover>
              <div class="image-wrapper">
                <img
                  :alt="picture.name"
                  :src="picture.thumbnailUrl ?? picture.url"
                  class="card-image"
                />
                <div class="image-overlay"></div>
              </div>
            </template>
            <a-card-meta>
              <template #title>
                <span class="card-title">{{ picture.name }}</span>
              </template>
              <template #description>
                <a-flex gap="4" wrap="wrap">
                  <a-tag color="blue" class="tech-tag">
                    {{ picture.category ?? '默认' }}
                  </a-tag>
                  <a-tag v-for="tag in picture.tags" :key="tag" class="tech-tag">
                    {{ tag }}
                  </a-tag>
                </a-flex>
              </template>
            </a-card-meta>
            <template v-if="showOp" #actions>
              <a-tooltip title="分享">
                <ShareAltOutlined @click="(e) => doShare(picture, e)" />
              </a-tooltip>
              <a-tooltip title="编辑">
                <EditOutlined v-if="canEdit" @click="(e) => doEdit(picture, e)" />
              </a-tooltip>
              <a-tooltip title="删除">
                <DeleteOutlined v-if="canDelete" @click="(e) => doDelete(picture, e)" />
              </a-tooltip>
            </template>
          </a-card>
        </a-list-item>
      </template>
    </a-list>
    <ShareModal ref="shareModalRef" :link="shareLink" />
  </div>
</template>

<script setup lang="ts">
import { useRouter } from 'vue-router'
import {
  DeleteOutlined,
  EditOutlined,
  ShareAltOutlined,
} from '@ant-design/icons-vue'
import { deletePictureUsingPost } from '@/api/pictureController.ts'
import { message } from 'ant-design-vue'
import ShareModal from '@/components/ShareModal.vue'
import { ref } from 'vue'

interface Props {
  dataList?: API.PictureVO[]
  loading?: boolean
  showOp?: boolean
  canEdit?: boolean
  canDelete?: boolean
  onReload?: () => void
}

const props = withDefaults(defineProps<Props>(), {
  dataList: () => [],
  loading: false,
  showOp: false,
  canEdit: false,
  canDelete: false,
})

const router = useRouter()
// 跳转至图片详情页
const doClickPicture = (picture: API.PictureVO) => {
  router.push({
    path: `/picture/${picture.id}`,
  })
}

// 编辑
const doEdit = (picture, e) => {
  // 阻止冒泡
  e.stopPropagation()
  // 跳转时一定要携带 spaceId
  router.push({
    path: '/add_picture',
    query: {
      id: picture.id,
      spaceId: picture.spaceId,
    },
  })
}

// 删除数据
const doDelete = async (picture, e) => {
  // 阻止冒泡
  e.stopPropagation()
  const id = picture.id
  if (!id) {
    return
  }
  const res = await deletePictureUsingPost({ id })
  if (res.data.code === 0) {
    message.success('删除成功')
    props.onReload?.()
  } else {
    message.error('删除失败')
  }
}

// ----- 分享操作 ----
const shareModalRef = ref()
// 分享链接
const shareLink = ref<string>()
// 分享
const doShare = (picture, e) => {
  // 阻止冒泡
  e.stopPropagation()
  shareLink.value = `${window.location.protocol}//${window.location.host}/picture/${picture.id}`
  if (shareModalRef.value) {
    shareModalRef.value.openModal()
  }
}
</script>

<style scoped>
.picture-list :deep(.ant-card) {
  background: rgba(20, 24, 33, 0.6);
  border: 1px solid rgba(255, 255, 255, 0.08);
}

.image-wrapper {
  position: relative;
  overflow: hidden;
  height: 180px;
}

.card-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.5s ease;
}

.glass-card:hover .card-image {
  transform: scale(1.1);
}

.image-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(to bottom, transparent 60%, rgba(11, 14, 20, 0.8));
  opacity: 0.6;
}

.card-title {
  color: #fff;
  font-weight: 600;
  font-size: 15px;
}

.tech-tag {
  border: 1px solid rgba(22, 93, 255, 0.3);
  background: rgba(22, 93, 255, 0.1);
  color: #165DFF;
  border-radius: 4px;
}

.picture-list :deep(.ant-card-actions) {
  background: rgba(255, 255, 255, 0.02);
  border-top: 1px solid rgba(255, 255, 255, 0.05);
}

.picture-list :deep(.ant-card-actions > li > span:not(.anticon)) {
  color: rgba(255, 255, 255, 0.45);
}

.picture-list :deep(.ant-card-actions > li:hover) {
  background: rgba(22, 93, 255, 0.1);
}
</style>
