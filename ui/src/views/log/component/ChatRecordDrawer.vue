<template>
  <el-drawer v-model="visible" size="60%" @close="closeHandle" class="chat-record-drawer">
    <template #header>
      <h4>{{ currentAbstract }}</h4>
    </template>
    <div
      v-loading="paginationConfig.current_page === 1 && loading"
      class="h-full"
      style="padding: 24px 0"
    >
      <InfiniteScroll
        :size="recordList.length"
        :total="paginationConfig.total"
        :page_size="paginationConfig.page_size"
        v-model:current_page="paginationConfig.current_page"
        @load="getChatRecord"
        :loading="loading"
      >
        <AiChat :data="application" :record="recordList" log></AiChat>
      </InfiniteScroll>
    </div>
    <template #footer>
      <div>
        <el-button @click="pre" :disabled="pre_disable || loading">上一条</el-button>
        <el-button @click="next" :disabled="next_disable || loading">下一条</el-button>
      </div>
    </template>
  </el-drawer>
</template>

<script setup lang="ts">
import { ref, reactive, computed, watch } from 'vue'
import { useRoute } from 'vue-router'
import logApi from '@/api/log'
import { type chatType } from '@/api/type/application'
import { type ApplicationFormType } from '@/api/type/application'
const props = withDefaults(
  defineProps<{
    /**
     * 应用信息
     */
    application?: ApplicationFormType
    /**
     * 对话 记录id
     */
    chartId: string
    currentAbstract: string
    /**
     * 下一条
     */
    next: () => void
    /**
     * 上一条
     */
    pre: () => void

    pre_disable: boolean

    next_disable: boolean
  }>(),
  {}
)

const emit = defineEmits(['update:chartId', 'update:currentAbstract', 'refresh'])

const route = useRoute()
const {
  params: { id }
} = route
const loading = ref(false)
const visible = ref(false)
const recordList = ref<chatType[]>([])

const paginationConfig = reactive({
  current_page: 1,
  page_size: 20,
  total: 0
})

function closeHandle() {
  recordList.value = []
  paginationConfig.total = 0
  paginationConfig.current_page = 1
}

function getChatRecord() {
  if (props.chartId && visible.value) {
    logApi.getChatRecordLog(id as string, props.chartId, paginationConfig, loading).then((res) => {
      paginationConfig.total = res.data.total
      recordList.value = [...recordList.value, ...res.data.records]
    })
  }
}

watch(
  () => props.chartId,
  () => {
    recordList.value = []
    paginationConfig.total = 0
    paginationConfig.current_page = 1
    getChatRecord()
  }
)

watch(visible, (bool) => {
  if (!bool) {
    emit('update:chartId', '')
    emit('update:currentAbstract', '')
    emit('refresh')
  }
})

const open = () => {
  getChatRecord()
  visible.value = true
}

defineExpose({
  open
})
</script>
<style lang="scss">
.chat-record-drawer {
  .el-drawer__body {
    background: var(--app-layout-bg-color);
    padding: 0;
  }
  :deep(.el-divider__text) {
    background: var(--app-layout-bg-color);
  }
}
</style>
