<template>
  <p-layout-well v-if="taskRun" class="task-run">
    <template #header>
      <PageHeadingTaskRun :task-run-id="taskRun.id" @delete="goToFlowRun" />
    </template>

    <p-tabs :tabs="tabs">
      <template #details>
        <TaskRunDetails :task-run="taskRun" />
      </template>

      <template #logs>
        <TaskRunLogs :task-run="taskRun" />
      </template>

      <template v-if="taskRun" #task-inputs-heading>
        Task inputs
        <ExtraInfoModal title="Task Inputs">
          {{ localization.info.taskInput }}
        </ExtraInfoModal>
      </template>
      <template #task-inputs>
        <CopyableWrapper v-if="deployment" :text-to-copy="parameters">
          <p-code-highlight lang="json" :text="parameters" class="task-run__inputs" />
        </CopyableWrapper>
      </template>
    </p-tabs>
    <template #well>
      <TaskRunDetails alternate :task-run="taskRun" />
    </template>
  </p-layout-well>
</template>

<script lang="ts" setup>
  import { media } from '@prefecthq/prefect-design'
  import { PageHeadingTaskRun, TaskRunLogs, TaskRunDetails, CopyableWrapper, useFavicon, useWorkspaceApi, localization, ExtraInfoModal } from '@prefecthq/prefect-ui-library'
  import { useRouteParam, useSubscriptionWithDependencies } from '@prefecthq/vue-compositions'
  import { computed } from 'vue'
  import { useRouter } from 'vue-router'
  import { usePageTitle } from '@/compositions/usePageTitle'
  import { routes } from '@/router'

  const router = useRouter()
  const taskRunId = useRouteParam('taskRunId')
  const api = useWorkspaceApi()

  const tabs = computed(() => {
    const values = ['Logs', 'Task Inputs']

    if (!media.xl) {
      values.push('Details')
    }

    return values
  })

  const taskRunIdArgs = computed<[string] | null>(() => taskRunId.value ? [taskRunId.value] : null)
  const taskRunDetailsSubscription = useSubscriptionWithDependencies(api.taskRuns.getTaskRun, taskRunIdArgs, { interval: 30000 })
  const taskRun = computed(() => taskRunDetailsSubscription.response)

  const flowRunId = computed(() => taskRun.value?.flowRunId)
  const flowRunIdArgs = computed<[string] | null>(() => flowRunId.value ? [flowRunId.value] : null)
  const flowRunDetailsSubscription = useSubscriptionWithDependencies(api.flowRuns.getFlowRun, flowRunIdArgs)

  const parameters = computed(() => {
    return taskRun.value?.taskInputs ? JSON.stringify(taskRun.value.taskInputs, undefined, 2) : '{}'
  })

  function goToFlowRun(): void {
    flowRunDetailsSubscription.refresh()
    router.push(routes.flowRun(flowRunId.value!))
  }

  const stateType = computed(() => taskRun.value?.stateType)
  useFavicon(stateType)

  const title = computed(() => {
    if (!taskRun.value) {
      return 'Task Run'
    }
    return `Task Run: ${taskRun.value.name}`
  })
  usePageTitle(title)
</script>

<style>
.task-run__inputs { @apply
  px-4
  py-3
}
</style>