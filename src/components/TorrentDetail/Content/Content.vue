<script setup lang="ts">
import ContentFilterDialog from '@/components/Dialogs/ContentFilterDialog.vue'
import { useContentStore, useDialogStore } from '@/stores'
import { Torrent, TreeNode } from '@/types/vuetorrent'
import { storeToRefs } from 'pinia'
import { computed, nextTick, onBeforeUnmount, onMounted, ref, watch } from 'vue'
import { useDisplay } from 'vuetify'
import ContentNode from './ContentNode.vue'

const props = defineProps<{ torrent: Torrent; isActive: boolean }>()

const { height: deviceHeight } = useDisplay()
const contentStore = useContentStore()
const { rightClickProperties, filenameFilter, openedItems, flatTree, internalSelection, timerForcedPause, isTimerActive } = storeToRefs(contentStore)
const dialogStore = useDialogStore()

const height = computed(() => {
  // 48px for the tabs and page title
  // 64px for the toolbar
  // 12px for the padding (top and bottom)
  // 56px for the filter text field
  // 8px for its top margin
  return deviceHeight.value - 48 * 2 - 64 - 12 * 2 - 56 - 8
})

async function onRightClick(e: MouseEvent | Touch, node: TreeNode) {
  if (rightClickProperties.value.isVisible) {
    rightClickProperties.value.isVisible = false
    await nextTick()
  }

  Object.assign(rightClickProperties.value, {
    isVisible: true,
    offset: [e.pageX, e.pageY],
    hash: props.torrent.hash
  })

  if (internalSelection.value.size <= 1) {
    internalSelection.value = new Set([node.fullName])
  }
}

// mobile long press
const timer = ref<NodeJS.Timeout>()

function startPress(e: Touch, node: TreeNode) {
  timer.value = setTimeout(() => {
    onRightClick(e, node)
  }, 500)
}

function endPress() {
  clearTimeout(timer.value)
}

// END mobile long press

watch(
  () => props.isActive,
  isActive => {
    if (isActive && !timerForcedPause.value) contentStore.resumeTimer()
    else contentStore.pauseTimer()
  }
)

onMounted(() => {
  props.isActive && contentStore.resumeTimer()
})
onBeforeUnmount(() => {
  contentStore.$reset()
})

function pause() {
  timerForcedPause.value = true
  contentStore.pauseTimer()
}

function resume() {
  timerForcedPause.value = false
  contentStore.resumeTimer()
}

function openFilterDialog() {
  dialogStore.createDialog(ContentFilterDialog)
}
</script>

<template>
  <v-card>
    <div class="mt-2 mx-3 d-flex flex-gap align-center">
      <v-text-field v-model="filenameFilter" hide-details clearable :placeholder="$t('torrentDetail.content.filter_placeholder')" />

      <v-tooltip :text="$t('torrentDetail.content.filter.activator')" location="bottom">
        <template #activator="{ props }">
          <v-btn v-bind="props"
                 icon="mdi-select-multiple"
                 color="primary"
                 @click="openFilterDialog" />
        </template>
      </v-tooltip>

      <v-tooltip :text="isTimerActive ? $t('common.pause') : $t('common.resume')" location="bottom">
        <template #activator="{ props }">
          <v-btn v-bind="props"
                 :icon="isTimerActive ? 'mdi-timer-pause' : 'mdi-timer-play'"
                 color="primary"
                 @click="isTimerActive ? pause() : resume()" />
        </template>
      </v-tooltip>
    </div>

    <v-virtual-scroll id="tree-root" :items="flatTree" :height="height" item-height="68" class="pa-2">
      <template #default="{ item }">
        <ContentNode
          :opened-items="openedItems"
          :node="item"
          @setFilePrio="(fileIdx, prio) => contentStore.setFilePriority(fileIdx, prio)"
          @touchcancel="endPress"
          @touchend="endPress"
          @touchmove="endPress"
          @touchstart="startPress($event.touches.item(0)!, item)"
          @onRightClick="(e, node) => onRightClick(e, node)" />
      </template>
    </v-virtual-scroll>
  </v-card>
</template>

<style lang="scss">
#_tree-root {
  font-size: medium;
  list-style-type: none;

  div.v-virtual-scroll__item {
    padding-top: 8px;

    &:first-child {
      padding-top: 0;
    }

    &:last-child {
      padding-bottom: 8px;
    }
  }
}
</style>
