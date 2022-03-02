<template>
  <a-layout>
    <a-layout-content
      :style="{
        background: '#fff',
        padding: '24px',
        margin: 0,
        minHeight: '280px',
      }"
    >
      <!-- <div class="about">
      <h1>管理电子书页面</h1>
    </div> -->
      <a-table
        :columns="columns"
        :row-key="(record) => record.id"
        :data-source="ebooks"
        :pagination="pagination"
        :loading="loading"
        @change="handleTableChange"
      >
        <template #cover="{ text: cover }">
          <img v-if="cover" :src="cover" alt="avatar" />
        </template>
        <!-- <template v-slot:category="{ text, record }"> -->
        <template v-slot:category="{ record }">
          <span
            >{{ getCategoryName(record.category1Id) }} /
            {{ getCategoryName(record.category2Id) }}</span
          >
        </template>
        <!-- <template v-slot:action="{ text, record }"> -->
        <template v-slot:action="{ record }">
          <a-space size="small">
            <router-link :to="'/admin/doc?ebookId=' + record.id">
              <a-button type="primary"> 文档管理 </a-button>
            </router-link>
            <a-button type="primary" @click="edit(record)"> 编辑 </a-button>
            <a-popconfirm
              title="删除后不可恢复，确认删除?"
              ok-text="是"
              cancel-text="否"
              @confirm="handleDelete(record.id)"
            >
              <a-button type="danger"> 删除 </a-button>
            </a-popconfirm>
          </a-space>
        </template>
      </a-table>
    </a-layout-content>
  </a-layout>
</template>
<script lang="ts">
import { defineComponent, onMounted, ref } from "vue";
import axios from "axios";
export default defineComponent({
  name: "AdminEbook",
  setup() {
    const ebooks = ref();
    // const pagination = ref();
    const pagination = ref({
      current: 1,
      pageSize: 10,
      total: 0,
    });
    const loading = ref(false);

    const columns = [
      {
        title: "封面",
        dataIndex: "cover",
        slots: { customRender: "cover" },
      },
      {
        title: "名称",
        dataIndex: "name",
      },
      {
        title: "分类一",
        Key: "category1Id",
        dataIndex: "category1Id",
      },
      {
        title: "分类二",
        dataIndex: "category2Id",
      },
      {
        title: "文档数",
        dataIndex: "docCount",
      },
      {
        title: "阅读数",
        dataIndex: "viewCount",
      },
      {
        title: "点赞数",
        dataIndex: "voteCount",
      },
      {
        title: "Action",
        key: "action",
        slots: { customRender: "action" },
      },
    ];
    /**
     * 数据查询
     **/
    const handleQuery = (params: any) => {
      loading.value = true;
      axios.get("/ebook/findAllEbook", params).then((response) => {
        loading.value = false;
        const { data } = response.data;
        ebooks.value = data.list;

        console.log("data=", data);

        // 重置分页按钮
        pagination.value.current = params.pageNum;
        pagination.value.total = data.navigatePages;
      });
    };

    /**
     * 表格点击页码时触发
     */
    const handleTableChange = (pagination: any) => {
      console.log("看看自带的分页参数都有啥：" + pagination);
      handleQuery({
        pageNum: pagination.current,
        pageSize: pagination.pageSize,
      });
    };
    onMounted(() => {
      handleQuery({
        pageNum: 1,
        pageSize: pagination.value.pageSize,
      });
    });
    return {
      ebooks,
      pagination,
      columns,
      loading,
      handleTableChange,
    };
  },
});
</script>