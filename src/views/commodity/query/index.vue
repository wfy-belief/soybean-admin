<script>
// @update:sorter="handleSorterChange"
// @update:filters="handleFiltersChange"
// @update:page="handlePageChange"
import { defineComponent, h, onMounted, reactive, ref } from 'vue';
import { usePagination } from 'alova/client';
import { NButton } from 'naive-ui';
import { alovaInstance } from '@/utils/service.js';

const columns = [
  {
    title: '商品编码',
    key: 'id'
  },
  {
    title: '省份',
    key: 'area'
  },
  {
    title: '价格',
    key: 'price'
  },
  // {
  //   title: '描述信息',
  //   key: 'description'
  // },
  {
    title: '用户名',
    key: 'user_name'
  },
  {
    title: '购买链接',
    key: 'url',
    render(row) {
      return h(
        NButton,
        {
          size: 'small',
          onClick: () => {
            window.open(row.url, '_blank');
          }
        },
        { default: () => '跳转地址' }
      );
    }
  },
  {
    title: '发布时间',
    key: 'publish_time'
  },
  {
    title: '更新时间',
    key: 'update_time'
  }
];

// https://alova.js.org/zh-CN/tutorial/client/strategy/use-pagination#%E6%B8%B2%E6%9F%93%E5%88%97%E8%A1%A8%E6%95%B0%E6%8D%AE
const { loading, data, page, pageSize, total, send } = usePagination(
  (page_param, page_size) =>
    alovaInstance.Get(`http://localhost:8080/list`, {
      params: {
        page: page_param,
        page_size
      },
      cacheFor: 0
    }),
  {
    initialData: {
      records: [],
      total: 0,
      size: 10,
      current: 1,
      pages: 0
    }, // 设置data状态的初始数据
    immediate: false, // 是否立即发送请求，默认为true
    total: response => {
      return response.total;
    },
    data: response => response.records
  }
);

export default defineComponent({
  setup() {
    const columnsRef = ref(columns);
    const paginationReactive = reactive({
      page: page.value,
      pageCount: total.value / pageSize.value,
      pageSize: pageSize.value,
      showSizePicker: true,
      pageSizes: [20, 50, 100],
      prefix({ itemCount }) {
        return `Total is ${itemCount}.`;
      },
      onUpdatePageSize: update_page_size => {
        paginationReactive.pageSize = update_page_size;
        pageSize.value = update_page_size;
        page.value = 1;
        paginationReactive.page = 1;
      }
    });

    onMounted(() => {
      send().then(() => {
        paginationReactive.pageCount = total.value / pageSize.value;
      });
    });

    return {
      data,
      columns: columnsRef,
      pagination: paginationReactive,
      loading,
      rowKey(rowData) {
        return rowData.id;
      },
      handlePageChange(currentPage) {
        page.value = currentPage;
        paginationReactive.page = currentPage;
        paginationReactive.itemCount = total.value;
        paginationReactive.pageCount = total.value / pageSize.value;
      }
    };
  }
});
</script>

<template>
  <NDataTable
    remote
    :columns="columns"
    :data="data"
    :loading="loading"
    :pagination="pagination"
    :show="true"
    :max-height="600"
    :row-key="rowKey"
    virtual-scroll
    @update:page="handlePageChange"
  />
</template>
