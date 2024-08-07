<script setup lang="ts">
import ImportSettingsDialog from '@/components/Dialogs/ImportSettingsDialog.vue'
import { defaultDateFormat, TitleOptions } from '@/constants/vuetorrent'
import { LOCALES } from '@/locales'
import { Github } from '@/services/Github'
import { useAppStore, useDialogStore, useHistoryStore, useVueTorrentStore } from '@/stores'
import { DarkLegacy, DarkRedesigned, LightLegacy, LightRedesigned } from '@/themes'
import { computed, readonly, ref } from 'vue'
import { useI18n } from 'vue-i18n'
import { toast } from 'vue3-toastify'

const { t } = useI18n()
const appStore = useAppStore()
const historyStore = useHistoryStore()
const vueTorrentStore = useVueTorrentStore()
const dialogStore = useDialogStore()

const github = new Github()

const titleOptionsList = readonly([
  { title: t('constants.titleOptions.default'), value: TitleOptions.DEFAULT },
  { title: t('constants.titleOptions.global_speed'), value: TitleOptions.GLOBAL_SPEED },
  { title: t('constants.titleOptions.first_torrent_speed'), value: TitleOptions.FIRST_TORRENT_STATUS },
  { title: t('constants.titleOptions.custom'), value: TitleOptions.CUSTOM }
])

const lightVariants = readonly([
  { title: t('constants.themes.light.legacy'), value: LightLegacy.id },
  { title: t('constants.themes.light.redesigned'), value: LightRedesigned.id }
])

const darkVariants = readonly([
  { title: t('constants.themes.dark.legacy'), value: DarkLegacy.id },
  { title: t('constants.themes.dark.redesigned'), value: DarkRedesigned.id }
])

const paginationSizes = ref([{ title: t('settings.vuetorrent.general.paginationSize.infinite_scroll'), value: -1 }, 5, 15, 30, 50, 100, 250, 500])

const vueTorrentVersion = computed(() => {
  if (import.meta.env.PROD) {
    return import.meta.env.VITE_PACKAGE_VERSION
  } else if (import.meta.env.DEV) {
    return 'DEV'
  }

  return null
})

const paginationSize = computed({
  get: () => {
    if (vueTorrentStore.paginationSize === -1) return t('settings.vuetorrent.general.paginationSize.infinite_scroll')
    return vueTorrentStore.paginationSize.toString()
  },
  set: (v: string) => {
    const input = parseInt(v, 10)
    if (isNaN(input)) return

    if (input <= 0 && input !== -1) {
      vueTorrentStore.paginationSize = -1
    } else {
      vueTorrentStore.paginationSize = input
    }
  }
})

const paginationSizeMessages = computed(() => (vueTorrentStore.paginationSize > 1000 ? t('settings.vuetorrent.general.paginationSize.warning') : ''))

const resetSettings = () => {
  localStorage.clear()
  sessionStorage.clear()
  location.reload()
}

const downloadSettings = () => {
  const settings = localStorage.getItem('vuetorrent_webuiSettings')
  if (!settings) return

  const jsonString = JSON.stringify(JSON.parse(settings), null, 2)
  const blob = new Blob([jsonString], { type: 'application/json' })
  const blobUrl = URL.createObjectURL(blob)

  const a = document.createElement('a')
  a.href = blobUrl
  a.download = 'settings.json'

  document.body.appendChild(a)
  a.click()
  document.body.removeChild(a)
}

const importSettings = () => {
  dialogStore.createDialog(ImportSettingsDialog)
}

const registerMagnetHandler = () => {
  if (typeof navigator.registerProtocolHandler !== 'function') {
    toast.error(t('toast.magnet_handler.not_supported'))
    return
  }

  const templateUrl = location.href.replace('/settings', '/magnet/%s')
  navigator.registerProtocolHandler('magnet', templateUrl)
  toast.success(t('toast.magnet_handler.registered'))
}

const checkNewVersion = async () => {
  if (vueTorrentVersion.value === 'DEV') return

  const latest = await github.getVersion()
  if (`v${vueTorrentVersion.value}` === latest) {
    toast.success(t('toast.version.latest'))
    return
  }

  toast.info(t('toast.version.new'))
}

function openBackendHelp() {
  window.open('https://github.com/VueTorrent/vuetorrent-backend/wiki/Installation', '_blank', 'noreferrer')
}
</script>

<template>
  <v-list>
    <v-list-subheader>{{ t('settings.vuetorrent.general.tip') }}</v-list-subheader>

    <v-list-item>
      <v-row>
        <v-col cols="12" sm="6">
          <v-checkbox v-model="vueTorrentStore.showCurrentSpeed" hide-details density="compact" :label="t('settings.vuetorrent.general.showCurrentSpeed')" />
        </v-col>
        <v-col cols="12" sm="6">
          <v-checkbox v-model="vueTorrentStore.showSpeedGraph" hide-details density="compact" :label="t('settings.vuetorrent.general.showSpeedGraph')" />
        </v-col>

        <v-col cols="12" sm="6">
          <v-checkbox v-model="vueTorrentStore.showAlltimeStat" hide-details density="compact" :label="t('settings.vuetorrent.general.showAlltimeStat')" />
        </v-col>
        <v-col cols="12" sm="6">
          <v-checkbox v-model="vueTorrentStore.showSessionStat" hide-details density="compact" :label="t('settings.vuetorrent.general.showSessionStat')" />
        </v-col>

        <v-col cols="12" sm="6">
          <v-checkbox v-model="vueTorrentStore.showFreeSpace" hide-details density="compact" :label="t('settings.vuetorrent.general.showFreeSpace')" />
        </v-col>
        <v-col cols="12" sm="6">
          <v-checkbox v-model="vueTorrentStore.isDrawerRight" hide-details density="compact" :label="t('settings.vuetorrent.general.isDrawerRight')" />
        </v-col>

        <v-col cols="12" sm="6">
          <v-checkbox v-model="vueTorrentStore.enableRatioColors" hide-details density="compact" :label="t('settings.vuetorrent.general.enableRatioColors')" />
        </v-col>
        <v-col cols="12" sm="6">
          <v-checkbox v-model="vueTorrentStore.enableHashColors" hide-details density="compact" :label="t('settings.vuetorrent.general.enableHashColors')" />
        </v-col>

        <v-col cols="12" sm="6">
          <v-checkbox v-model="vueTorrentStore.hideChipIfUnset" hide-details density="compact" :label="t('settings.vuetorrent.general.hideChipIfUnset')" />
        </v-col>
        <v-col cols="12" sm="6">
          <v-checkbox v-model="vueTorrentStore.hideColoredChip" hide-details density="compact" :label="t('settings.vuetorrent.general.hideColoredChip')" />
        </v-col>

        <v-col cols="12" sm="6">
          <v-checkbox v-model="vueTorrentStore.isShutdownButtonVisible" hide-details density="compact" :label="t('settings.vuetorrent.general.isShutdownButtonVisible')" />
        </v-col>
        <v-col cols="12" sm="6"></v-col>

        <v-col cols="12" sm="6">
          <v-checkbox v-model="vueTorrentStore.useBinarySize" hide-details density="compact" :label="t('settings.vuetorrent.general.useBinarySize')" />
        </v-col>
        <v-col cols="12" sm="6">
          <v-checkbox v-model="vueTorrentStore.useBitSpeed" hide-details density="compact" :label="t('settings.vuetorrent.general.useBitSpeed')" />
        </v-col>
      </v-row>
    </v-list-item>

    <v-list-item class="mt-3">
      <v-row>
        <v-col cols="12" md="4">
          <v-text-field v-model.number="vueTorrentStore.refreshInterval" type="number" hide-details suffix="ms" :label="t('settings.vuetorrent.general.refreshInterval')" />
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field v-model.number="vueTorrentStore.fileContentInterval" type="number" hide-details suffix="ms" :label="t('settings.vuetorrent.general.fileContentInterval')" />
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field v-model.number="historyStore.historySize" type="number" hide-details :label="t('settings.vuetorrent.general.historySize')" />
        </v-col>

        <v-col cols="12" md="6">
          <v-select v-model="vueTorrentStore.language" flat hide-details :items="LOCALES" :label="t('settings.vuetorrent.general.language')" />
        </v-col>
        <v-col cols="12" md="6">
          <v-combobox
            v-model="paginationSize"
            :messages="paginationSizeMessages"
            flat
            hide-details
            :items="paginationSizes"
            :return-object="false"
            :label="t('settings.vuetorrent.general.paginationSize.label')" />
        </v-col>

        <v-col cols="12" md="4">
          <v-select v-model="vueTorrentStore.uiTitleType" flat hide-details :items="titleOptionsList" :label="t('settings.vuetorrent.general.vueTorrentTitle')" />
        </v-col>
        <v-col cols="12" md="8">
          <v-text-field
            :disabled="vueTorrentStore.uiTitleType !== TitleOptions.CUSTOM"
            v-model="vueTorrentStore.uiTitleCustom"
            hide-details
            :label="t('settings.vuetorrent.general.customTitle')" />
        </v-col>

        <v-col cols="12" md="6">
          <v-select v-model="vueTorrentStore.theme.light" flat hide-details :items="lightVariants" :label="$t('settings.vuetorrent.general.lightVariants')" />
        </v-col>
        <v-col cols="12" md="6">
          <v-select v-model="vueTorrentStore.theme.dark" flat hide-details :items="darkVariants" :label="$t('settings.vuetorrent.general.darkVariants')" />
        </v-col>
      </v-row>

      <v-row>
        <v-col cols="12" md="6">
          <v-text-field v-model="vueTorrentStore.dateFormat" :placeholder="defaultDateFormat" hint="using Dayjs" :label="t('settings.vuetorrent.general.dateFormat')" />
        </v-col>

        <v-col cols="12" md="6">
          <v-text-field
            v-model="vueTorrentStore.backendUrl"
            :label="t('settings.vuetorrent.general.backendUrl')"
            placeholder="https://YOUR-HOST:PORT/"
            append-inner-icon="mdi-help-circle"
            @click:appendInner="openBackendHelp" />
        </v-col>
      </v-row>
    </v-list-item>

    <v-list-item>
      <v-row>
        <v-col cols="6" class="d-flex align-center justify-center">
          <h3>
            {{ t('settings.vuetorrent.general.currentVersion') }}
            <span v-if="!vueTorrentVersion">undefined</span>
            <a v-else-if="vueTorrentVersion === 'DEV'" target="_blank" href="https://github.com/VueTorrent/VueTorrent/">{{ vueTorrentVersion }}</a>
            <a v-else target="_blank" :href="`https://github.com/VueTorrent/VueTorrent/releases/tag/v${vueTorrentVersion}`">{{ vueTorrentVersion }}</a>
          </h3>
        </v-col>

        <v-col cols="6" class="d-flex align-center justify-center">
          <v-btn color="primary" @click="registerMagnetHandler">{{ t('settings.vuetorrent.general.registerMagnet') }} </v-btn>
        </v-col>
      </v-row>
    </v-list-item>

    <v-list-item>
      <v-row>
        <v-col cols="12" sm="6" class="d-flex align-center justify-center">
          <h3>
            {{ t('settings.vuetorrent.general.qbittorrentVersion') }}
            <a target="_blank" :href="`https://github.com/qbittorrent/qBittorrent/releases/tag/release-${appStore.version}`">{{ appStore.version }}</a>
          </h3>
        </v-col>
        <v-col cols="12" sm="6" class="d-flex align-center justify-center">
          <v-btn color="primary" @click="checkNewVersion">{{ t('settings.vuetorrent.general.check_new') }} </v-btn>
        </v-col>
      </v-row>
    </v-list-item>

    <v-list-item>
      <v-row>
        <v-col cols="12" sm="4" class="d-flex align-center justify-center">
          <v-btn color="primary" @click="importSettings">{{ t('settings.vuetorrent.general.import') }}</v-btn>
        </v-col>
        <v-col cols="12" sm="4" class="d-flex align-center justify-center">
          <v-btn color="primary" @click="downloadSettings">{{ t('settings.vuetorrent.general.download') }}</v-btn>
        </v-col>
        <v-col cols="12" sm="4" class="d-flex align-center justify-center">
          <v-btn color="red" @click="resetSettings">{{ t('settings.vuetorrent.general.resetSettings') }}</v-btn>
        </v-col>
      </v-row>
    </v-list-item>
  </v-list>
</template>
