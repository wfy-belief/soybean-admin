<template>
  <n-data-table
    :columns="columns"
    :data="data.records"
    :row-key="rowKey"
    :pagination="paginationReactive"
  />
</template>

<script setup>
import { useRequest } from 'alova/client';
import { NButton, useMessage } from 'naive-ui';
import { h, reactive } from 'vue';
import { alovaInstance } from '@/utils/service.js';

const message = useMessage();
const params = {
  page: 1,
  page_size: 20
};

const paginationReactive = reactive({
  page: 1,
  pageSize: 10,
  pageTotal: 100,
  showSizePicker: true,
  pageSizes: [20, 50, 100],
  // onChange: (page) => {
  //   paginationReactive.page = page;
  // },
  // onUpdatePageSize: (pageSize) => {
  //   paginationReactive.pageSize = pageSize;
  //   paginationReactive.page = 1;
  //   params.page = paginationReactive.page;
  //   params.pageSize = pageSize;
  // }
});

// 使用alova实例创建method并传给useRequest即可发送请求
const { loading, data, refresh, error, send, update } = useRequest(
  alovaInstance.Get(`http://localhost:8080/list`, {
    params: params,
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
    immediate: true // 是否立即发送请求，默认为true
  }
).onSuccess(event => {

});


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


function rowKey(rowData) {
  return rowData.id
}

</script>

<!--todo 1.按照时间排序-->
<!--todo 2.按照价格排序-->
<!--todo 3.关键字搜索-->
<!--todo 4.订单标记已购买，已删除-->
