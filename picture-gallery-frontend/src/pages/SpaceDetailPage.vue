<template>
  <div id="spaceDetailPage">
    <!-- 空间信息 -->
    <a-flex justify="space-between" align="center" class="space-header">
      <div class="header-left">
        <h2 class="glow-title">{{ space.spaceName }}（{{ SPACE_TYPE_MAP[space.spaceType] }}）</h2>
        <div v-if="space.spaceType === SPACE_TYPE_ENUM.TEAM" class="collaboration-status">
          <a-tag color="blue">
            <template #icon>
              <SyncOutlined :spin="true" />
            </template>
            正在实时协作
          </a-tag>
        </div>
        <div class="space-usage">
          <a-tooltip
            :title="`占用空间 ${formatSize(space.totalSize)} / ${formatSize(space.maxSize)}`"
          >
            <a-progress
              :percent="Number(((space.totalSize * 100) / space.maxSize).toFixed(1))"
              size="small"
              stroke-color="#165DFF"
              :show-info="false"
              class="usage-progress"
            />
          </a-tooltip>
          <span class="usage-text">
             {{ formatSize(space.totalSize) }} / {{ formatSize(space.maxSize) }}
           </span>
         </div>
       </div>
       <div class="header-right">
         <a-space size="middle">
           <a-button
             v-if="canUploadPicture"
             type="primary"
             :icon="h(PlusOutlined)"
             :href="`/add_picture?spaceId=${id}`"
             class="flow-button"
           >
             上传图片
           </a-button>
           <a-button
             v-if="canManageSpaceUser && space.spaceType === SPACE_TYPE_ENUM.TEAM"
             type="primary"
             ghost
             :icon="h(TeamOutlined)"
             :href="`/spaceUserManage/${id}`"
           >
             成员管理
           </a-button>
           <a-button
             v-if="canManageSpaceUser"
             type="primary"
             ghost
             :icon="h(BarChartOutlined)"
             :href="`/space_analyze?spaceId=${id}`"
           >
             空间分析
           </a-button>
           <a-button v-if="canEditPicture" :icon="h(EditOutlined)" @click="doBatchEdit">
             批量编辑
           </a-button>
         </a-space>
       </div>
     </a-flex>
      <div style="margin-bottom: 16px" />
    <!-- 搜索表单 -->
    <PictureSearchForm :onSearch="onSearch" />
    <div style="margin-bottom: 16px" />
    <!-- 按颜色搜索，跟其他搜索条件独立 -->
    <a-form-item label="按颜色搜索">
      <color-picker format="hex" @pureColorChange="onColorChange" />
    </a-form-item>
    <!-- 图片列表 -->
    <PictureList
      v-if="dataList.length > 0"
      :dataList="dataList"
      :loading="loading"
      :showOp="true"
      :canEdit="canEditPicture"
      :canDelete="canDeletePicture"
      :onReload="fetchData"
    />
    <a-empty v-else-if="!loading" description="暂无图片，快来上传吧！">
      <a-button
        v-if="canUploadPicture"
        type="primary"
        :icon="h(PlusOutlined)"
        :href="`/add_picture?spaceId=${id}`"
        class="flow-button"
      >
        立即上传
      </a-button>
    </a-empty>
    <!-- 分页 -->
    <a-pagination
      style="text-align: right"
      v-model:current="searchParams.current"
      v-model:pageSize="searchParams.pageSize"
      :total="total"
      @change="onPageChange"
    />
    <BatchEditPictureModal
      ref="batchEditPictureModalRef"
      :spaceId="id"
      :pictureList="dataList"
      :onSuccess="onBatchEditPictureSuccess"
    />
  </div>
</template>

<script setup lang="ts">
import { computed, h, onMounted, ref, watch } from 'vue'
import { getSpaceVoByIdUsingGet } from '@/api/spaceController.ts'
import { message } from 'ant-design-vue'
import {
  listPictureVoByPageUsingPost,
  searchPictureByColorUsingPost,
} from '@/api/pictureController.ts'
import { formatSize } from '@/utils'
import PictureList from '@/components/PictureList.vue'
import PictureSearchForm from '@/components/PictureSearchForm.vue'
import { ColorPicker } from 'vue3-colorpicker'
import 'vue3-colorpicker/style.css'
import BatchEditPictureModal from '@/components/BatchEditPictureModal.vue'
import { BarChartOutlined, EditOutlined, TeamOutlined, PlusOutlined, SyncOutlined } from '@ant-design/icons-vue'
import { SPACE_PERMISSION_ENUM, SPACE_TYPE_MAP, SPACE_TYPE_ENUM } from '../constants/space.ts'
import { useLoginUserStore } from '@/stores/useLoginUserStore.ts'

interface Props {
  id: string | number
}

const props = defineProps<Props>()
const space = ref<API.SpaceVO>({})
const loginUserStore = useLoginUserStore()

// 通用权限检查函数
function createPermissionChecker(permission: string) {
  return computed(() => {
    const loginUser = loginUserStore.loginUser
    if (!loginUser.id) {
      return false
    }
    // 空间创建者拥有所有权限
    if (space.value.userId && space.value.userId === loginUser.id) {
      return true
    }
    return (space.value.permissionList ?? []).includes(permission)
  })
}

// 定义权限检查
const canManageSpaceUser = createPermissionChecker(SPACE_PERMISSION_ENUM.SPACE_USER_MANAGE)
const canUploadPicture = createPermissionChecker(SPACE_PERMISSION_ENUM.PICTURE_UPLOAD)
const canEditPicture = createPermissionChecker(SPACE_PERMISSION_ENUM.PICTURE_EDIT)
const canDeletePicture = createPermissionChecker(SPACE_PERMISSION_ENUM.PICTURE_DELETE)

// -------- 获取空间详情 --------
const fetchSpaceDetail = async () => {
  try {
    const res = await getSpaceVoByIdUsingGet({
      id: props.id,
    })
    if (res.data.code === 0 && res.data.data) {
      space.value = res.data.data
    } else {
      message.error('获取空间详情失败，' + res.data.message)
    }
  } catch (e: any) {
    message.error('获取空间详情失败：' + e.message)
  }
}

onMounted(() => {
  fetchSpaceDetail()
})

// --------- 获取图片列表 --------

// 定义数据
const dataList = ref<API.PictureVO[]>([])
const total = ref(0)
const loading = ref(true)

// 搜索条件
const searchParams = ref<API.PictureQueryRequest>({
  current: 1,
  pageSize: 12,
  sortField: 'createTime',
  sortOrder: 'descend',
})

// 获取数据
const fetchData = async () => {
  loading.value = true
  // 转换搜索参数
  const params = {
    spaceId: props.id,
    ...searchParams.value,
  }
  const res = await listPictureVoByPageUsingPost(params)
  if (res.data.code === 0 && res.data.data) {
    dataList.value = res.data.data.records ?? []
    total.value = res.data.data.total ?? 0
  } else {
    message.error('获取数据失败，' + res.data.message)
  }
  loading.value = false
}

// 页面加载时获取数据，请求一次
onMounted(() => {
  fetchData()
})

// 分页参数
const onPageChange = (page: number, pageSize: number) => {
  searchParams.value.current = page
  searchParams.value.pageSize = pageSize
  fetchData()
}

// 搜索
const onSearch = (newSearchParams: API.PictureQueryRequest) => {
  console.log('new', newSearchParams)

  searchParams.value = {
    ...searchParams.value,
    ...newSearchParams,
    current: 1,
  }
  console.log('searchparams', searchParams.value)
  fetchData()
}

// 按照颜色搜索
const onColorChange = async (color: string) => {
  loading.value = true
  const res = await searchPictureByColorUsingPost({
    picColor: color,
    spaceId: props.id,
  })
  if (res.data.code === 0 && res.data.data) {
    const data = res.data.data ?? []
    dataList.value = data
    total.value = data.length
  } else {
    message.error('获取数据失败，' + res.data.message)
  }
  loading.value = false
}

// ---- 批量编辑图片 -----
const batchEditPictureModalRef = ref()

// 批量编辑图片成功
const onBatchEditPictureSuccess = () => {
  fetchData()
}

// 打开批量编辑图片弹窗
const doBatchEdit = () => {
  if (batchEditPictureModalRef.value) {
    batchEditPictureModalRef.value.openModal()
  }
}

// 空间 id 改变时，必须重新获取数据
watch(
  () => props.id,
  (newSpaceId) => {
    fetchSpaceDetail()
    fetchData()
  },
)
</script>

<style scoped>
#spaceDetailPage {
  margin-bottom: 16px;
}

.space-header {
  margin-bottom: 24px;
  padding: 16px 24px;
  background: rgba(255, 255, 255, 0.02);
  border-radius: 12px;
  border: 1px solid rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(10px);
}

.header-left .glow-title {
  margin-bottom: 8px;
  font-size: 24px;
}

.collaboration-status {
  margin-bottom: 12px;
}

.space-usage {
  display: flex;
  align-items: center;
  gap: 12px;
}

.usage-progress {
  width: 120px;
  margin: 0;
}

.usage-text {
  font-size: 12px;
  color: rgba(255, 255, 255, 0.45);
}

.header-right :deep(.ant-btn-ghost) {
  border-color: rgba(255, 255, 255, 0.15);
  color: rgba(255, 255, 255, 0.85);
}

.header-right :deep(.ant-btn-ghost:hover) {
  border-color: #165DFF;
  color: #165DFF;
}
</style>
