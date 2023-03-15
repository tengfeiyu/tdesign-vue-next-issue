<template>
  <!-- 表格 -->
  <t-table :columns="columns" :data="data" row-key="id" hover>
    <template #disabled="scope">
      <!-- <t-button
          :theme="`${scope.row.disabled ? 'primary' : 'default'}`"
          size="small"
          :loading="toggleDisabled[scope.row.id]"
          @click="handleRowDisabledChange(scope.row)"
        >
          {{ scope.row.disabled ? '启用' : '禁用' }}
        </t-button> -->
      <t-switch
        :value="scope.row.enabled"
        :label="['正常', '禁用']"
        :loading="toggleDisabled[scope.row.id]"
        @change="handleRowDisabledChange(scope.row)"
      />
    </template>
    <template #locked="scope">
      <!-- <t-button
          :theme="`${scope.row.locked ? 'primary' : 'default'}`"
          size="small"
          :loading="toggleLocked[scope.row.id]"
          @click="handleRowLockedChange(scope.row)"
        >
          {{ scope.row.locked ? '解锁' : '锁定' }}
        </t-button> -->
      <t-switch
        :value="scope.row.unLocked"
        :label="['正常', '锁定']"
        :loading="toggleLocked[scope.row.id]"
        @change="handleRowLockedChange(scope.row, scope.row.unLocked)"
      />
      选中状态 {{ scope.row.unLocked }}
    </template>
  </t-table>
</template>

<script setup lang="ts">
import lodash from 'lodash';
import { Ref, ref, shallowRef } from 'vue';

const listData = [
  { id: 1, disabled: 0, locked: 0 },
  { id: 2, disabled: 0, locked: 0 },
  { id: 3, disabled: 0, locked: 0 },
];

const list = () => {
  return new Promise<Array<any>>((resolve) => {
    setTimeout(() => {
      resolve(listData);
    }, lodash.random(333, 666));
  });
};

const columns = shallowRef([
  {
    colKey: 'disabled',
    title: '禁用状态',
    width: 110,
  },
  {
    colKey: 'locked',
    title: '锁定状态',
    width: 110,
  },
]);
const data: Ref<Array<any>> = ref([]);
const toggleDisabled: Ref<Record<string, boolean>> = ref({});
const toggleLocked: Ref<Record<string, boolean>> = ref({});

const handleTableQuery = () => {
  list().then((d) => {
    console.log('获取最新数据', d);
    data.value = d.map((row: any) => {
      const { disabled, locked, ...others } = row;
      return {
        ...others,
        enabled: lodash.isNumber(disabled) && disabled === 0,
        unLocked: lodash.isNumber(locked) && locked === 0,
        _origin: row,
      };
    });
  });
};
// operations methods
const handleRowDisabledChange = async (row: any) => {
  toggleDisabled.value = {
    ...toggleDisabled.value,
    [row.id]: true,
  };
  const matchedRow: any = listData.find((item) => item.id === row.id) || {};
  try {
    if (row.enabled) {
      matchedRow.disabled = 1;
    } else {
      matchedRow.disabled = 0;
    }
    handleTableQuery();
  } finally {
    toggleDisabled.value = {
      ...toggleDisabled.value,
      [row.id]: false,
    };
  }
};
const handleRowLockedChange = async (row: any, passed: boolean) => {
  console.log('操作传递数据', row);
  console.log('操作传递状态', passed);
  // ### 此处获取的数据状态在第一次操作成功后，刷新列表再次操作之后就一直无法正确获取
  // ### 取而代之的用button方式，不存在此问题
  toggleLocked.value = {
    ...toggleLocked.value,
    [row.id]: true,
  };
  const matchedRow: any = listData.find((item) => item.id === row.id) || {};
  try {
    if (row.unLocked) {
      matchedRow.locked = 1;
    } else {
      matchedRow.locked = 0;
    }
    handleTableQuery();
  } finally {
    toggleLocked.value = {
      ...toggleLocked.value,
      [row.id]: false,
    };
  }
};

handleTableQuery();
</script>
