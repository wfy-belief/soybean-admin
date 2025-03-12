<template>
  <n-data-table
    remote
    ref="table"
    :columns="columns"
    :data="data"
    :loading="loading"
    :pagination="pagination"
    :show="true"
    :row-key="rowKey"
    @update:page="handlePageChange"
  />
</template>

<script>
// @update:sorter="handleSorterChange"
// @update:filters="handleFiltersChange"
// @update:page="handlePageChange"
import { defineComponent, h, onMounted, reactive, ref } from 'vue';
import { useRequest } from 'alova/client';
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
      return h(NButton, {
          size: 'small',
          onClick: () => {
            params.a = 123;
            send();
            window.open(row.link, '_blank');
          }
        },
        { default: () => '跳转地址' });
    }
  },
  {
    title: '发布时间',
    key: 'publish_time'
  }, {
    title: '更新时间',
    key: 'update_time'
  }
];

let page_ = 1
let page_size_ = 15

const { loading, data, refresh, error, send, update } = useRequest(
  alovaInstance.Get(`http://localhost:8080/list`, {
    params: {
      page: page_,
      page_size: page_size_,
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
    immediate: false // 是否立即发送请求，默认为true
  }
);

export default defineComponent({
  setup() {
    const dataRef = ref([]);
    const loadingRef = ref(true);
    const columnsRef = ref(columns);
    const paginationReactive = reactive({
      page: page_,
      pageCount: 1,
      pageSize: page_size_,
      showSizePicker: true,
      pageSizes: [20, 50, 100],
      prefix({ itemCount }) {
        return `Total is ${itemCount}.`;
      }
    });

    onMounted(() => {
      send(paginationReactive).then((res) => {
        dataRef.value = res.records;
        paginationReactive.pageCount = res.pages;
        paginationReactive.itemCount = res.total;
        loadingRef.value = loading.value;
      });
    });

    return {
      data: dataRef,
      columns: columnsRef,
      pagination: paginationReactive,
      loading: loadingRef,
      rowKey(rowData) {
        return rowData.id;
      },
      handlePageChange(currentPage) {
        if (!loadingRef.value) {
          page_ = currentPage;
          loadingRef.value = true;
          send().then((res) => {
            dataRef.value = res.records;
            paginationReactive.page = currentPage;
            paginationReactive.pageCount = res.pages;
            paginationReactive.itemCount = res.total;
            loadingRef.value = false;
          });
        }
      }
    };
  }
});
</script>
