<template>
  <dao-table
    id="local-volume-event-list"
    :data="state.items"
    :columns="columns"
    :sort="initialSort"
    :page-size="pagination.pageSize"
    :current-page="pagination.page"
    :total="pagination.total"
    :loading="isLoading"
    @sort-change="handleSortChange"
    @page-change="handleChangePage"
    @size-change="handleChangePageSize"
    @refresh="handleRefresh"
  >
    <template #search>
      <dao-input
        v-model="filterData.action"
        type="search"
        block
        borderless
        @keyup.enter="handleSearch"
        @after-reset="handleSearch"
      />
    </template>

    <template #td-actionType="{row}">
      {{ row.eventRecord?.action }}
    </template>

    <template #td-status="{row}">
      {{ row.eventRecord?.state ?? '-' }}
    </template>

    <template #td-time="{row}">
      {{ useDateFormat(row.eventRecord?.time) }}
    </template>

    <template #td-content="{row}">
      <dao-text-button
        :prop="{
          type:'action',
        }"
        @click="viewEventContent(row)"
      >
        {{ $t('views.local-volumes.components.LocalVolumeEvents.view') }}
      </dao-text-button>
    </template>
  </dao-table>
</template>

<script setup lang="ts">
import { ref, computed, defineProps } from 'vue';
import { useI18n } from 'vue-i18n';
import { createDialog, useDateFormat, useQueryTable } from '@dao-style/extend';
import { Volume } from '@/services/Volume';
import type {
  ApiEventAction,
  VolumesEventsDetailParams,
} from '@/services/data-contracts';
import MonacoEditorDialog from '@/components/dialogs/MonacoEditorDialog.vue';
import type { DaoTableSort } from '@dao-style/core';

const props = defineProps({
  volume: {
    type: String,
    default: '',
  },
});

const VolumeAPi = new Volume();
const { t } = useI18n();

const columns = computed(() => [
  {
    id: 'actionType',
    header: t('views.local-volumes.components.LocalVolumeEvents.actionType'),
  },
  {
    id: 'status',
    header: t('views.local-volumes.components.LocalVolumeEvents.status'),
  },
  {
    id: 'time',
    header: t('views.local-volumes.components.LocalVolumeEvents.operateTime'),
    sortable: true,
  },
  {
    id: 'content',
    header: t('views.local-volumes.components.LocalVolumeEvents.eventContent'),
  },
]);

const initialSort = ref<DaoTableSort>({
  id: 'time',
  desc: true,
});

const queryEvents = async (req: VolumesEventsDetailParams) => {
  const { data } = await VolumeAPi.volumesEventsDetail({
    ...req,
  });

  return data;
};

const [{
  state,
  pagination,
  handleChangePage,
  handleChangePageSize,
  handleRefresh,
  isLoading,
}, {
  filterData,
  handleSearch,
}] = useQueryTable(queryEvents, {
  page: 1,
  pageSize: 10,
  sort: 'time',
  sortDir: 'DESC',
  volumeName: props.volume,
});

const handleSortChange = ({ id, desc }: { id: string, desc: boolean }) => {
  initialSort.value.id = id;
  initialSort.value.desc = desc;

  filterData.sort = id;
  filterData.sortDir = initialSort.value.desc ? 'DESC' : 'ASC';

  handleRefresh();
};

const viewEventContent = (row: ApiEventAction) => {
  const dialog = createDialog(MonacoEditorDialog);
  let content = row.eventRecord?.actionContent ?? '';

  try {
    content = JSON.stringify(JSON.parse(content), null, 2);
  } finally {
    dialog.show({
      modelValue: content,
      header: t('views.local-volumes.components.LocalVolumeEvents.eventContent'),
      readOnly: true,
    });
  }
};
</script>
