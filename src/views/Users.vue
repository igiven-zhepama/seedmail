<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { MessagePlugin, PaginationProps, TableProps } from 'tdesign-vue-next';
import { userApi } from '../services/userApi';
import { useRouter } from 'vue-router';
import { h } from 'vue';
import { Button as TButton, Space as TSpace } from 'tdesign-vue-next';
interface Recipient {
  email: string;
}

const loading = ref(false);
const users = ref<Recipient[]>([]);
const pagination = ref({
  total: 0,
  current: 1,
  pageSize: 10
});

const deleteUser = async (row: Recipient) => {
  try {
    await userApi.deleteUser(row.email);
    MessagePlugin.success('删除成功');
    await fetchUsers({
      current: pagination.value.current,
      pageSize: pagination.value.pageSize,
    });
  } catch (error) {
    MessagePlugin.error('删除失败');
  }
};

const columns = ref([
  { colKey: 'email', title: '邮箱地址', width: 300 },
  {
    colKey: 'operation',
    title: '操作',
    width: 200,
    cell: (d: any, { row }: { row: Recipient }) => {
      return h(TSpace, {}, {
        default: () => [
          h(TButton, {
            theme: 'primary',
            variant: 'text',
            onClick: () => sendMail(row)
          }, {
            default: () => '写信'
          }),
          h(TButton, {
            theme: 'danger',
            variant: 'text',
            onClick: () => deleteUser(row)
          }, {
            default: () => '删除'
          })
        ]
      });
    }
  }
]);
const router = useRouter();
const sendMail = (row: Recipient) => {
  router.push({
    path: '/compose',
    query: {
      to: "",
      from: row.email,
      subject: ``,
    }
  });
};


const fetchUsers = async (paginationInfo: PaginationProps) => {
  try {
    loading.value = true;
    const { current, pageSize } = paginationInfo;
    const response = await userApi.listUsers(current, pageSize);
    users.value = response.users.map((x: string) => {
      return {
        email: x
      };
    });
    pagination.value.total = response.total;
  } catch (error) {
    MessagePlugin.error('获取用户列表失败');
    users.value = [];
  } finally {
    loading.value = false;
  }
};

const onPageChange: TableProps['onPageChange'] = async (pageInfo) => {
  pagination.value.current = pageInfo.current;
  pagination.value.pageSize = pageInfo.pageSize;
  await fetchUsers(pageInfo);
};

onMounted(async () => {
  await fetchUsers({
    current: pagination.value.current,
    pageSize: pagination.value.pageSize,
  });
});
</script>

<template>
  <div class="p-6">
    <div class="flex justify-between items-center mb-6">
      <h1 class="text-2xl font-bold bg-gradient-to-r from-blue-600 to-blue-800 bg-clip-text text-transparent">
        用户列表
      </h1>
    </div>

    <t-card class="shadow-md rounded-xl overflow-hidden">
      <t-table :data="users" :columns="columns" :loading="loading" :pagination="pagination" row-key="email" hover
        stripe lazy-load @page-change="onPageChange" class="recipients-table">
        <template #empty>
          <div class="flex flex-col items-center justify-center py-12 text-gray-500">
            <t-icon name="user-circle" size="48px" class="mb-4 text-gray-300" />
            <p>暂无用户记录</p>
          </div>
        </template>
      </t-table>
    </t-card>
  </div>
</template>

<style scoped>
.recipients-table {
  :deep(.t-table__header) {
    @apply bg-gray-50;
  }

  :deep(.t-table__row) {
    @apply transition-colors duration-200;
  }

  :deep(.t-table__row:hover) {
    @apply bg-blue-50;
  }

  :deep(.t-table__cell) {
    @apply py-4;
  }
}

:deep(.t-pagination) {
  @apply mt-4;
}

:deep(.t-pagination .t-button) {
  @apply hover:bg-blue-50;
}

:deep(.t-pagination .t-is-active) {
  @apply bg-blue-600 text-white hover:bg-blue-700;
}
</style>
