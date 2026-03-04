<script setup lang="ts">
import { onMounted, onUnmounted } from 'vue'
import { usePhotobooth } from '@/composables/usePhotobooth'
import { useLiveView } from '@/composables/useLiveView'
import ScreenIdle from '@/components/photobooth/ScreenIdle.vue'
import ScreenTemplate from '@/components/photobooth/ScreenTemplate.vue'
import ScreenShoot from '@/components/photobooth/shoot/ScreenShoot.vue'
import ScreenResult from '@/components/photobooth/ScreenResult.vue'
import ScreenUploading from '@/components/photobooth/ScreenUploading.vue'
import ScreenProcessing from '@/components/photobooth/ScreenProcessing.vue'
import TestPanel from '@/components/photobooth/TestPanel.vue'
import ScreenDbViewer from '@/components/photobooth/ScreenDbViewer.vue'
import CameraTestPage from '@/components/photobooth/CameraTestPage.vue'
import TestFilterPage from '@/components/testfillter/TestFilterPage.vue'
import LoadingOverlay from '@/components/photobooth/LoadingOverlay.vue'
import Footer from '@/components/photobooth/Footer.vue'

const photobooth = usePhotobooth()
const { currentScreen, showScreen, runDevStartPage, buildFinalOutput, selectTemplate, templates, callHost } = photobooth
const { setHostLiveViewDataUrl, liveViewFrameCount } = useLiveView()

onMounted(() => {
  // 鎖住右鍵選單
  document.addEventListener('contextmenu', (e) => {
    e.preventDefault()
    return false
  })
  document.addEventListener('dragstart', (e) => {
    e.preventDefault()
    return false
  })
  // 禁止 Ctrl + 滾輪／Ctrl + +/- 放大縮小網頁
  document.addEventListener('wheel', (e) => {
    if (e.ctrlKey) {
      e.preventDefault()
      return false
    }
  }, { passive: false })
  // 禁止觸控雙指放大縮小（需 passive: false 才能 preventDefault）
  document.addEventListener('touchmove', (e) => {
    if (e.touches.length >= 2) {
      e.preventDefault()
    }
  }, { passive: false })
  document.addEventListener('keydown', (e) => {
    // if (e.key === 'F12') {
    //   e.preventDefault()
    //   return false
    // }
    if (e.ctrlKey && (e.key === '+' || e.key === '=' || e.key === '-')) {
      e.preventDefault()
      return false
    }
    if (e.ctrlKey && e.shiftKey && e.key === 'I') {
      e.preventDefault()
      return false
    }
    if (e.ctrlKey && e.shiftKey && e.key === 'J') {
      e.preventDefault()
      return false
    }
    if (e.ctrlKey && e.key === 'U') {
      e.preventDefault()
      return false
    }
    if (e.ctrlKey && e.key === 'S') {
      e.preventDefault()
      return false
    }
  })

  const win = window as unknown as {
    chrome?: {
      webview?: {
        addEventListener: (type: string, handler: (ev: { data: string }) => void) => void
      }
    }
  }
  if (win.chrome?.webview) {
    win.chrome.webview.addEventListener('message', (ev: { data: string }) => {
      try {
        const msg = JSON.parse(ev.data) as {
          '@event'?: string
          event?: string
          amount?: number
          dataUrl?: string
          templateId?: string
        }
        const eventType = msg['@event'] ?? msg.event
        if (eventType === 'wpf_shoot_done' && typeof msg.templateId === 'string') {
          const templateId = msg.templateId
          const tpl = templates.value.find((t: { id: string }) => t.id === templateId) ?? templates.value[0]
          if (tpl) selectTemplate(tpl)
          showScreen('uploading')
          buildFinalOutput()
          return
        }
        if (eventType === 'liveview_frame' && typeof msg.dataUrl === 'string') {
          setHostLiveViewDataUrl(msg.dataUrl)
          if (liveViewFrameCount.value === 1) {
            console.log('[Live View] 第一幀已收到')
          } else if (liveViewFrameCount.value % 100 === 0) {
            console.log(`[Live View] 已收到 ${liveViewFrameCount.value} 幀`)
          }
          return
        }
      } catch {
        // ignore
      }
    })
  }

  runDevStartPage()
})

onUnmounted(() => {
})
</script>

<template>
  <div id="app" class="photobooth-app">
    <ScreenIdle :class="{ active: currentScreen === 'idle' }" />
    <CameraTestPage
      v-if="currentScreen === 'camera-test'"
      @close="showScreen('idle')"
    />
    <TestFilterPage
      v-if="currentScreen === 'test-filter'"
      :class="{ active: currentScreen === 'test-filter' }"
      @close="showScreen('idle')"
    />
    <ScreenDbViewer
      v-if="currentScreen === 'db-view'"
      class="active"
      @close="showScreen('idle')"
    />
    <TestPanel />
    <ScreenTemplate :class="{ active: currentScreen === 'template' }" />
    <ScreenShoot
      :class="{ active: currentScreen === 'shoot' }"
      :is-active="currentScreen === 'shoot'"
    />
    <ScreenResult :class="{ active: currentScreen === 'result' }" />
    <ScreenUploading :class="{ active: currentScreen === 'uploading' }" />
    <ScreenProcessing :class="{ active: currentScreen === 'processing' }" />
    <LoadingOverlay />
    <Footer />
  </div>
</template>

<style>
/* 全域由 main.scss 提供 */
</style>
