<template>
  <div id="basicLayout">
    <div class="tech-grid-bg"></div>
    <div class="nebula-bg"></div>
    <div class="cloud-lines"></div>
    <a-layout style="min-height: 100vh; background: transparent">
      <a-layout-header class="header">
        <GlobalHeader />
      </a-layout-header>
      <a-layout style="background: transparent">
        <GlobalSider class="sider" />
        <a-layout-content class="content">
          <router-view />
        </a-layout-content>
      </a-layout>
      <a-layout-footer class="footer">
        made by fpr
      </a-layout-footer>
    </a-layout>
  </div>
</template>

<script setup lang="ts">
import GlobalHeader from '@/components/GlobalHeader.vue'
import GlobalSider from '@/components/GlobalSider.vue'
</script>

<style scoped>
#basicLayout {
  min-height: 100vh;
}

#basicLayout .header {
  padding-inline: 20px;
  background: rgba(11, 14, 20, 0.7);
  backdrop-filter: blur(10px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
  color: unset;
  margin-bottom: 1px;
}

#basicLayout .sider {
  background: rgba(11, 14, 20, 0.5);
  backdrop-filter: blur(10px);
  border-right: 1px solid rgba(255, 255, 255, 0.08);
  padding-top: 20px;
}

#basicLayout :deep(.ant-menu-root) {
  border-bottom: none !important;
  border-inline-end: none !important;
}

#basicLayout .content {
  padding: 28px;
  background: transparent;
  margin-bottom: 28px;
}

#basicLayout .footer {
  background: rgba(11, 14, 20, 0.8);
  backdrop-filter: blur(10px);
  border-top: 1px solid rgba(255, 255, 255, 0.08);
  padding: 16px;
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  text-align: center;
  color: rgba(255, 255, 255, 0.45);
}
</style>