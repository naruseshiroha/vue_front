<template>
  <a-layout>
    <a-layout-sider width="200" style="background: #fff">
      <a-menu
        mode="inline"
        v-model:selectedKeys="selectedKeys2"
        v-model:openKeys="openKeys"
        :style="{ height: '100%', borderRight: 0 }"
        @click="handleClick"
      >
        <a-menu-item key="welcome">
          <MailOutlined />
          <span>欢迎</span>
        </a-menu-item>
        <a-sub-menu v-for="item in level1" :key="item.id">
          <template v-slot:title>
            <span><user-outlined />{{ item.name }}</span>
          </template>
          <a-menu-item v-for="child in item.children" :key="child.id">
            <MailOutlined /><span>{{ child.name }}</span>
          </a-menu-item>
        </a-sub-menu>
      </a-menu>
    </a-layout-sider>
    <a-layout-content
      :style="{
        background: '#fff',
        padding: '24px',
        margin: 0,
        minHeight: '280px',
      }"
    >
      <div class="welcome" v-show="isShowWelcome">
        <h1>欢迎来到我的知识库</h1>
      </div>
      <!-- {{ ebooks }} -->
      <a-list
        v-show="!isShowWelcome"
        item-layout="vertical"
        size="large"
        :grid="{ gutter: 20, column: 3 }"
        :pagination="pagination"
        :data-source="ebooks"
      >
        <template #renderItem="{ item }">
          <a-list-item key="item.name">
            <template #actions>
              <span v-for="{ type, text } in actions" :key="type">
                <component v-bind:is="type" style="margin-right: 8px" />
                {{ text }}
              </span>
            </template>
            <a-list-item-meta :description="item.description">
              <template #title>
                <a :href="item.href">{{ item.name }}</a>
              </template>
              <template #avatar><a-avatar :src="item.cover" /></template>
            </a-list-item-meta>
          </a-list-item>
        </template>
      </a-list>
    </a-layout-content>
  </a-layout>
</template>

<script lang="ts">
import { defineComponent, onMounted, ref } from "vue";
import HelloWorld from "@/components/HelloWorld.vue"; // @ is an alias to /src
import axios from "axios";
import { message } from "ant-design-vue";
import { Tool } from "@/utils/tool";

export default defineComponent({
  name: "HomeView",
  components: {
    HelloWorld,
  },
  setup() {
    console.log("setup");
    const ebooks = ref();

    const pagination = {
      onChange: (pageNum: number) => {
        axios
          .get("/ebook/findAllEbook", {
            params: { pageNum, pageSize: pagination.pageSize },
          })
          .then((response) => {
            const { data } = response.data;
            ebooks.value = data.list;
            pagination.total = data.total;
            pagination.current = data.pageNum;
          });
      },
      pageSize: 9,
      pageSizeOptions: [2, 3, 4, 5, 6],
      current: 1,
      total: "",
      hideOnSinglePage: true,
    };

    const level1 = ref();
    let categorys: any;
    /**
     * 查询所有分类
     **/
    const handleQueryCategory = () => {
      axios.get("/category/all").then((response) => {
        const { data } = response;
        if (data.code === 200) {
          categorys = data.data;
          console.log("原始数组：", categorys);
          level1.value = [];
          level1.value = Tool.array2Tree(categorys, 0);
          console.log("树形结构：", level1.value);
        } else {
          message.error(data.message);
        }
      });
    };

    const isShowWelcome = ref(true);
    let category2Id = 0;

    const handleQueryEbook = () => {
      axios
        .get("/ebook/findAllEbook", {
          params: {
            pageNum: 1,
            pageSize: 1000,
            category2Id: category2Id,
          },
        })
        .then((response) => {
          const { data } = response.data;
          console.log("data", data);
          ebooks.value = data.list;

          pagination.pageSize = data.pageSize;
          pagination.total = data.total;
          pagination.current = data.pageNum;
        });
    };

    const handleClick = (value: any) => {
      console.log("首页左边分类被点击啦啦啦", value);
      if (value.key === "welcoome") {
        isShowWelcome.value = true;
      } else {
        category2Id = value.key;
        isShowWelcome.value = false;
        handleQueryEbook();
      }
    };

    const actions: Record<string, string>[] = [
      { type: "StarOutlined", text: "156" },
      { type: "LikeOutlined", text: "156" },
      { type: "MessageOutlined", text: "2" },
    ];
    onMounted(() => {
      handleQueryCategory();

      axios
        .get("/ebook/findAllEbook", { params: { pageNum: 1, pageSize: 9 } })
        .then((response) => {
          console.log(response);
          const { data } = response.data;
          ebooks.value = data.list;
          pagination.total = data.total;
          pagination.current = data.pageNum;
        });
    });
    return {
      ebooks,
      pagination,
      actions,

      handleClick,
      level1,

      isShowWelcome,
    };
  },
});
</script>
