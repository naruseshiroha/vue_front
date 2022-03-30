<template>
  <a-layout>
    <a-layout-content>
      <div
        class="doc"
        :style="{
          background: '#fff',
          padding: '24px',
          margin: 0,
          minHeigh: '280px',
        }"
      >
        <a-row>
          <a-col :span="6">
            <a-tree
              v-if="level1.length > 0"
              :tree-data="level1"
              @select="onSelect"
              :replaceFields="{ title: 'name', key: 'id', value: 'id' }"
              :defaultExpandAll="true"
            >
            </a-tree>
          </a-col>
          <a-col :span="18">
            <div :innerHTML="html"></div>
          </a-col>
        </a-row>
      </div>
    </a-layout-content>
  </a-layout>
</template>
<script lang="ts">
import { defineComponent, onMounted, ref, createVNode } from "vue";
import axios from "axios";
import { message } from "ant-design-vue";
import { Tool } from "@/utils/tool";
import { useRoute } from "vue-router";
export default defineComponent({
  name: "AdminDoc",
  setup() {
    // 通过路由内置函数得到一个路由的变量route，而这个变量里面就有路由的信息，包括路径参数
    const route = useRoute();
    const docs = ref();
    const html = ref();

    /**
     * 一级文档树的数据层级说明，children属性就是二级文档
     * [{
     *   id: "",
     *   name: "",
     *   children: [{
     *     id: "",
     *     name: "",
     *   }]
     * }]
     */
    const level1 = ref(); // 一级文档树，children属性就是二级文档
    level1.value = [];
    /**
     * 数据查询
     **/
    const handleQuery = () => {
      axios.get("/doc/all/" + route.query.ebookId).then((response) => {
        const { data } = response;
        if (data.code == 200) {
          docs.value = data.data;

          level1.value = [];
          level1.value = Tool.array2Tree(docs.value, 0);
        } else {
          message.error(data.message);
        }
      });
    };

    /**
     * 数据内容
     **/
    const handleQueryContent = (id: number) => {
      axios.get("/doc/find-content/" + id).then((response) => {
        const { data } = response;
        if (data.code == 200) {
          html.value = data.data;
          console.log(html);
          console.log(html.value);
        } else {
          message.error(data.message);
        }
      });
    };

    const onSelect = (selectedKeys: any, info: any) => {
      console.log("selected", selectedKeys, info);
      if (Tool.isNotEmpty(selectedKeys)) {
        handleQueryContent(selectedKeys[0]);
      }
    };

    onMounted(() => {
      handleQuery();
    });
    return {
      level1,
      html,
      onSelect,
    };
  },
});
</script>
