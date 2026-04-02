<template>
  <div id="homePage">
    <!-- 英雄区 -->
    <div class="hero-section">
      <h1 class="glow-title">探索 智能云图库</h1>
      <p class="subtitle">基于 AI 的高端云端图片管理平台</p>
      <!-- 搜索框 -->
      <div class="search-bar">
        <a-input-search
          v-model:value="searchParams.searchText"
          placeholder="从海量图片中搜索"
          enter-button="搜索"
          size="large"
          class="tech-search"
          @search="doSearch"
        />
      </div>
    </div>
    <!-- 分类和标签筛选 -->
    <div class="filter-section glass-card">
      <a-tabs v-model:active-key="selectedCategory" @change="doSearch">
        <a-tab-pane key="all" tab="全部" />
        <a-tab-pane v-for="category in categoryList" :tab="category" :key="category" />
      </a-tabs>
      <div class="tag-bar">
        <span class="tag-label">标签：</span>
        <a-space :size="[0, 8]" wrap>
          <a-checkable-tag
            v-for="(tag, index) in tagList"
            :key="tag"
            v-model:checked="selectedTagList[index]"
            class="tech-checkable-tag"
            @change="doSearch"
          >
            {{ tag }}
          </a-checkable-tag>
        </a-space>
      </div>
    </div>
    <!-- 图片列表 -->
    <PictureList :dataList="dataList" :loading="loading" />
    <!-- 分页 -->
    <a-pagination
      style="text-align: right; margin-top: 24px"
      v-model:current="searchParams.current"
      v-model:pageSize="searchParams.pageSize"
      :total="total"
      @change="onPageChange"
    />
  </div>
</template>

<script setup lang="ts">
import { onMounted, reactive, ref } from 'vue'
import {
  listPictureTagCategoryUsingGet,
  listPictureVoByPageUsingPost,
} from '@/api/pictureController.ts'
import { message } from 'ant-design-vue'
import PictureList from '@/components/PictureList.vue' // 定义数据

// 定义数据
const dataList = ref<API.PictureVO[]>([])
const total = ref(0)
const loading = ref(true)

// 搜索条件
const searchParams = reactive<API.PictureQueryRequest>({
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
    ...searchParams,
    tags: [] as string[],
  }
  if (selectedCategory.value !== 'all') {
    params.category = selectedCategory.value
  }
  // [true, false, false] => ['java']
  selectedTagList.value.forEach((useTag, index) => {
    if (useTag) {
      params.tags.push(tagList.value[index])
    }
  })
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
  searchParams.current = page
  searchParams.pageSize = pageSize
  fetchData()
}

// 搜索
const doSearch = () => {
  // 重置搜索条件
  searchParams.current = 1
  fetchData()
}

// 标签和分类列表
const categoryList = ref<string[]>([])
const selectedCategory = ref<string>('all')
const tagList = ref<string[]>([])
const selectedTagList = ref<boolean[]>([])

/**
 * 获取标签和分类选项
 * @param values
 */
const getTagCategoryOptions = async () => {
  const res = await listPictureTagCategoryUsingGet()
  if (res.data.code === 0 && res.data.data) {
    tagList.value = res.data.data.tagList ?? []
    categoryList.value = res.data.data.categoryList ?? []
  } else {
    message.error('获取标签分类列表失败，' + res.data.message)
  }
}

onMounted(() => {
  getTagCategoryOptions()
})
</script>

<style scoped>
#homePage {
  max-width: 1440px;
  margin: 0 auto;
}

.hero-section {
  text-align: center;
  padding: 40px 0 60px;
  animation: fade-in 0.8s ease-out;
}

.glow-title {
  font-size: 42px;
  margin-bottom: 16px;
  background: linear-gradient(135deg, #fff 0%, #165DFF 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  filter: drop-shadow(0 0 10px rgba(22, 93, 255, 0.3));
}

.subtitle {
  color: rgba(255, 255, 255, 0.65);
  font-size: 18px;
  margin-bottom: 32px;
}

.search-bar {
  max-width: 600px;
  margin: 0 auto;
}

.tech-search :deep(.ant-input) {
  background: rgba(255, 255, 255, 0.05) !important;
  border: 1px solid rgba(255, 255, 255, 0.1) !important;
  color: #fff;
  backdrop-filter: blur(4px);
}

.tech-search :deep(.ant-input-search-button) {
  background: #165DFF !important;
  border-color: #165DFF !important;
  box-shadow: 0 0 15px rgba(22, 93, 255, 0.4);
}

.filter-section {
  padding: 16px 24px;
  margin-bottom: 24px;
}

.tag-bar {
  margin-top: 12px;
  display: flex;
  align-items: center;
}

.tag-label {
  color: #FFFFFF;
  margin-right: 12px;
}

.tech-checkable-tag {
  border: 1px solid rgba(255, 255, 255, 0.1);
  background: rgba(255, 255, 255, 0.02);
  color: #FFFFFF;
  transition: all 0.3s;
}

.tech-checkable-tag-checked {
  background-color: #165DFF !important;
  border-color: #165DFF !important;
  color: #fff !important;
  box-shadow: 0 0 10px rgba(22, 93, 255, 0.3);
}

#homePage :deep(.ant-tabs-nav::before) {
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
}

#homePage :deep(.ant-tabs-tab) {
  color: rgba(255, 255, 255, 0.65);
}

#homePage :deep(.ant-tabs-tab-active .ant-tabs-tab-btn) {
  color: #165DFF !important;
  text-shadow: 0 0 8px rgba(22, 93, 255, 0.3);
}

#homePage :deep(.ant-pagination-item),
#homePage :deep(.ant-pagination-prev),
#homePage :deep(.ant-pagination-next) {
  background: rgba(255, 255, 255, 0.05);
  border-color: rgba(255, 255, 255, 0.1);
}

#homePage :deep(.ant-pagination-item a) {
  color: rgba(255, 255, 255, 0.85);
}

#homePage :deep(.ant-pagination-item-active) {
  border-color: #165DFF;
}

#homePage :deep(.ant-pagination-item-active a) {
  color: #165DFF;
}
</style>
