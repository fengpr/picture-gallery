<template>
  <div id="app">
    <a-config-provider :locale="zhCN" :theme="themeConfig">
      <BasicLayout />
    </a-config-provider>
  </div>
</template>

<script setup lang="ts">
import zhCN from 'ant-design-vue/es/locale/zh_CN'
import { theme } from 'ant-design-vue'
import dayjs from 'dayjs'
import 'dayjs/locale/zh-cn'
dayjs.locale('zh-cn')

import BasicLayout from '@/layouts/BasicLayout.vue'

const themeConfig = {
  algorithm: theme.darkAlgorithm,
  token: {
    colorPrimary: '#165DFF',
    colorBgContainer: 'rgba(20, 24, 33, 0.8)',
    colorBgLayout: '#0B0E14',
    borderRadius: 8,
    colorBorder: 'rgba(255, 255, 255, 0.08)',
    fontFamily: 'Inter, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif',
  },
}
</script>

<style scoped></style>
