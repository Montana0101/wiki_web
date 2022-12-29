<template>
  <a-layout>
    <a-layout-sider width="200" style="background: #fff">
      <a-menu
        v-model:selectedKeys="selectedKeys2"
        v-model:openKeys="openKeys"
        mode="inline"
        :style="{ height: '100%', borderRight: 0 }"
        @click="handleClick"
      >
        <a-menu-item key="init">
          <MailOutlined />
          <span>欢迎</span>
        </a-menu-item>
        <a-sub-menu v-for="item in level1" :key="item.id">
          <template #icon>
            <MailOutlined />
          </template>
          <template #title>{{ item.name }}</template>

          <a-menu-item v-for="subItem in item.children" :key="subItem.id">
            {{ subItem.name }}
          </a-menu-item>
        </a-sub-menu>
        <!-- <a-sub-menu v-for="item in level1" :key="item.id">
          <template #title>
              {{item.name}}
          </template>
        </a-sub-menu> -->

        <!-- <a-select-option v-for="c in level1" :key="c.id" :value="c.id"
                             :disabled="category.parent == c.id">
              {{c.name}}
            </a-select-option> -->
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
      <div class="welcome" v-show="isShowWelcome">知识库</div>
      <a-list
        v-show="!isShowWelcome"
        item-layout="vertical"
        size="large"
        :grid="{ gutter: 20, column: 3 }"
        :data-source="ebooks"
      >
        <template #renderItem="{ item }">
          <a-list-item key="item.name">
            <template #actions>
              <span v-for="{ type, text } in actions" :key="type">
                <component :is="type" style="margin-right: 8px" />
                {{ text }}
              </span>
            </template>

            <a-list-item-meta :description="item.description">
              <template #title>
                <a :href="item.href">{{ item.name }}</a>
              </template>
              <template #avatar><a-avatar :src="item.cover" /></template>
            </a-list-item-meta>
            {{ item.content }}
          </a-list-item>
        </template>
      </a-list>
    </a-layout-content>
  </a-layout>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted, reactive, toRef } from "vue";
import {
  StarOutlined,
  LikeOutlined,
  MessageOutlined,
  MailOutlined,
} from "@ant-design/icons-vue";
import { Tool } from "@/util/tool";
import { message } from "ant-design-vue";
import axios from "axios";
import About from "../../../../jiawawiki/web/src/views/about.vue";

const listData: Record<string, string>[] = [];

for (let i = 0; i < 23; i++) {
  listData.push({
    href: "https://www.antdv.com/",
    avatar: "https://joeschmoe.io/api/v1/random",
    description:
      "Ant Design, a design language for background applications, is refined by Ant UED Team.",
    content:
      "We supply a series of design principles, practical patterns and high quality design resources (Sketch and Axure), to help people create their product prototypes beautifully and efficiently.",
  });
}
export default defineComponent({
  name: "Home",
  components: {
    StarOutlined,
    LikeOutlined,
    MessageOutlined,
    About,
    MailOutlined,
  },
  setup() {
    console.log("setup");
    const ebooks = ref();
    const ebooks1 = reactive({
      books: [],
    });
    const loading = ref(false);
    const level1 = ref();
    const isShowWelcome = ref(true);
    let categoryId2:any;

    const handleClick = (value: any) => {
      if (value.key == "init") {
        isShowWelcome.value = true;
        return;
      }
      isShowWelcome.value = false;
      categoryId2 = value.key;
      getList();
      // categoryId2
    };
    /**
     * 查询所有分类
     **/
    const handleQueryCategory = () => {
      loading.value = true;
      axios.get("/category/all").then((response) => {
        loading.value = false;
        const data = response.data;
        if (data.success) {
          console.log("原始数组：", data.content);

          level1.value = [];
          level1.value = Tool.array2Tree(data.content, 0);
          console.log("树形结构首页菜单：", level1.value);

          // 加载完分类后，再加载电子书，否则如果分类树加载很慢，则电子书渲染会报错
          // handleQuery({
          //   page: pagination.value.current,
          //   size: pagination.value.pageSize,
          // });
        } else {
          message.error(data.message);
        }
      });
    };

    const getList = () => {
      axios
        .get("/ebook/list", {
          params: {
            categoryId2: categoryId2
          },
        })
        .then((res) => {
          ebooks.value = res.data.content.list;
          ebooks1.books = res.data.content.list;
        });
    };

    onMounted(() => {
      handleQueryCategory();
    });

    const actions: Record<string, string>[] = [
      { type: "StarOutlined", text: "156" },
      { type: "LikeOutlined", text: "156" },
      { type: "MessageOutlined", text: "2" },
    ];

    return {
      listData,
      ebooks,
      ebooks2: toRef(ebooks1, "books"),
      actions,
      level1,
      isShowWelcome,
      handleClick,
    };
  },
});
</script>

<style lang="css" scoped>
.welcome {
  display: flex;
  justify-content: center;
  align-items: center;
  /* background:red; */
  border: 1px solid red;

  height: 100%;
}
</style>