<script setup lang="ts">
import DefaultTheme from 'vitepress/theme'
import { useData, inBrowser, useRoute } from 'vitepress'
import { watchEffect, onMounted, watch } from 'vue'

const { lang } = useData()
watchEffect(() => {
  if (inBrowser) {
    document.cookie = `nf_lang=${lang.value}; expires=Mon, 1 Jan 2030 00:00:00 UTC; path=/`
  }
})

const route = useRoute()

const loadGiscus = () => {
  const script = document.createElement('script')
  script.src = 'https://giscus.app/client.js'
  script.setAttribute('data-repo', 'jasoneri/awesome-nijigen-ai-exp')  // 替换为你的仓库
  script.setAttribute('data-repo-id', 'R_kgDOOuGPIw')          // 替换为仓库ID
  script.setAttribute('data-category', 'General')
  script.setAttribute('data-category-id', 'DIC_kwDOOuGPI84CqdJd')
  script.setAttribute('data-mapping', 'title')
  script.setAttribute('data-strict', '0')
  script.setAttribute('data-reactions-enabled', '1')
  script.setAttribute('data-emit-metadata', '0')
  script.setAttribute('data-input-position', 'top')
  script.setAttribute('data-theme', 'preferred_color_scheme')
  script.setAttribute('data-lang', 'zh-CN')
  script.setAttribute('data-loading', 'lazy')
  script.crossOrigin = 'anonymous'
  script.async = true
  
  const container = document.querySelector('.giscus-container')
  if (container) {
    container.innerHTML = '' // 清空旧内容
    container.appendChild(script)
  }
}

onMounted(() => {
  loadGiscus()
})

watch(() => route.path, () => {
  loadGiscus()
})
</script>

<template>
  <DefaultTheme.Layout>
    <template #doc-after>
      <div class="giscus-container" /> <!-- Giscus 挂载点 -->
    </template>
  </DefaultTheme.Layout>
</template>