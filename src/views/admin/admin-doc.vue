<template>
  <a-layout>
    <a-layout>
      <a-layout-content
        :style="{
          background: '#fff',
          padding: '24px',
          margin: 0,
          minHeight: '280px',
        }"
      >
        <p>
          <a-form layout="inline" :model="param">
            <a-form-item>
              <a-input v-model:value="param.name" placeholder="名称"> </a-input>
            </a-form-item>
            <a-form-item>
              <a-button
                type="primary"
                @click="handleQuery({ page: 1, size: pagination.pageSize })"
              >
                查询
              </a-button>
            </a-form-item>
            <a-form-item>
              <a-button type="primary" @click="add()"> 新增 </a-button>
            </a-form-item>
          </a-form>
        </p>
        <a-table
          :columns="columns"
          :row-key="(record) => record.id"
          :data-source="level1"
          :pagination="pagination"
          :loading="loading"
          @change="handleTableChange"
        >
          <template #cover="{ text: cover }">
            <img v-if="cover" :src="cover" alt="avatar" />
          </template>
          <!-- <template v-slot:doc="{ text, record }">
          <span
            >{{ getDocName(record.doc1Id) }} /
            {{ getDocName(record.doc2Id) }}</span
          >
        </template> -->
          <template v-slot:action="{ text, record }">
            <a-space size="small">
              <!-- <router-link :to="'/admin/doc?docId=' + record.id">
              <a-button type="primary"> 文档管理 </a-button>
            </router-link> -->
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

    <a-modal v-model:visible="modalVisible" title="文档选择器" @ok="handleOk">
      <template #footer>
        <a-button key="back" @click="handleCancel">取消</a-button>
        <a-button
          key="submit"
          type="primary"
          :loading="mloading"
          @click="modalHandleOk"
          >确定</a-button
        >
      </template>
      <a-form :layout="formState.layout" :model="doc" v-bind="formItemLayout">
        <a-form-item label="名称">
          <a-input v-model:value="doc.name" placeholder="请输入名称" />
        </a-form-item>
        <a-form-item label="父文档">
          <!-- <a-select
              ref="select"
              v-model:value="doc.parent"
          >
            <a-select-option value="0">无</a-select-option>
            <a-select-option v-for="c in level1" :key="c.id" :value="c.id"
                             :disabled="doc.parent == c.id">
              {{c.name}}
            </a-select-option>
          </a-select> -->
          <a-tree-select
            v-model:value="doc.parent"
            style="width: 100%"
            :dropdown-style="{ maxHeight: '400px', overflow: 'auto' }"
            placeholder="请选择父文档"
            tree-default-expand-all
            :tree-data="treeSelectData"
            :replaceFields="{ label: 'name', key: 'id', value: 'id' }"
          >
            <!-- <template #title="{ key, value }">
              <span style="color:#08c" v-if="key == '0-0-1' ">
                Childdsa {{value}}
              </span>
            </template> -->
          </a-tree-select>
          <!--          <a-input v-model:value="doc.parent" placeholder="请输入名称" />-->
        </a-form-item>
        <a-form-item label="顺序">
          <a-input v-model:value="doc.sort" placeholder="请输入顺序" />
        </a-form-item>
      </a-form>
    </a-modal>
  </a-layout>
</template>

<script lang="ts">
import { defineComponent, onMounted, ref, reactive, computed } from "vue";
import axios from "axios";
import { message } from "ant-design-vue";
import { Tool } from "@/util/tool";
import type { UnwrapRef } from "vue";
import { useRoute } from "vue-router";
interface FormState {
  layout: "horizontal" | "vertical" | "inline";
  fieldA: string;
  fieldB: string;
}

export default defineComponent({
  name: "AdminDoc",
  setup() {
    const param = ref();
    const treeSelectData = ref();
    treeSelectData.value = [];
    const route: any = useRoute();
    console.log("路由：", route);
    console.log("route.path：", route.path);
    console.log("route.query：", route.query);
    console.log("route.param：", route.params);
    console.log("route.fullPath：", route.fullPath);
    console.log("route.name：", route.name);
    console.log("route.meta：", route.meta);

    param.value = {};
    const docs = ref();
    const pagination = ref({
      current: 1,
      pageSize: 10,
      total: 0,
    });
    const loading = ref(false);

    const columns = [
      {
        title: "名称",
        dataIndex: "name",
      },
      {
        title: "父文档",
        dataIndex: "parent",
      },
      {
        title: "顺序",
        dataIndex: "sort",
      },
      {
        title: "操作",
        key: "action",
        slots: { customRender: "action" },
      },
    ];

    /**
     * 数据查询
     **/
    const handleQuery = (params: any) => {
      loading.value = true;
      // 如果不清空现有数据，则编辑保存重新加载数据后，再点编辑，则列表显示的还是编辑前的数据
      docs.value = [];
      axios.get("/doc/all").then((response) => {
        loading.value = false;
        const data = response.data;
        if (data.success) {
          // docs.value = data.content
          // 重置分页按钮
          // pagination.value.current = params.page;
          // pagination.value.total = data.content.total;
          // console.log("打印下数据",data.content.total)
          // console.log("dashdsanasjkdnsaasjkdsa", data.content);

          level1.value = [];
          level1.value = Tool.array2Tree(data.content, 0);
          console.log("树形结构", level1.value);
        } else {
          message.error(data.message);
        }
      });
    };

    /**
     * 表格点击页码时触发
     */
    const handleTableChange = (pagination: any) => {
      console.log("看看自带的分页参数都有啥：" + pagination);
      handleQuery({
        page: pagination.current,
        size: pagination.pageSize,
      });
    };

    // -------- 表单 ---------
    /**
     * 数组，[100, 101]对应：前端开发 / Vue
     */
    const docIds = ref();
    const doc = ref();
    const modalVisible = ref(false);
    const modalLoading = ref(false);
    const handleModalOk = () => {
      modalLoading.value = true;
      console.log("打印下表单数据", formState);
      // doc.value.doc1Id = docIds.value[0];
      // doc.value.doc2Id = docIds.value[1];
      // axios.post("/doc/save", doc.value).then((response) => {
      //   modalLoading.value = false;
      //   const data = response.data; // data = commonResp
      //   if (data.success) {
      //     modalVisible.value = false;
      //
      //     // 重新加载列表
      //     handleQuery({
      //       page: pagination.value.current,
      //       size: pagination.value.pageSize,
      //     });
      //   } else {
      //     message.error(data.message);
      //   }
      // });
    };

    let deleteIds: Array<string> = [];
    const deleteNames: Array<string> = [];
    /**
     * 查找整根树枝
     */
    const getDeleteIds = (treeSelectData: any, id: any) => {
      // console.log(treeSelectData, id);
      // 遍历数组，即遍历某一层节点
      for (let i = 0; i < treeSelectData.length; i++) {
        const node = treeSelectData[i];
        if (node.id === id) {
          // 如果当前节点就是目标节点
          console.log("delete", node);
          // 将目标ID放入结果集ids
          // node.disabled = true;
          deleteIds.push(id);
          deleteNames.push(node.name);

          // 遍历所有子节点
          const children = node.children;
          if (Tool.isNotEmpty(children)) {
            for (let j = 0; j < children.length; j++) {
              getDeleteIds(children, children[j].id);
            }
          }
        } else {
          // 如果当前节点不是目标节点，则到其子节点再找找看。
          const children = node.children;
          if (Tool.isNotEmpty(children)) {
            getDeleteIds(children, id);
          }
        }
      }
    };

    /**
     * 将某节点及其子孙节点全部置为disabled
     */
    const setDisable = (treeSelectData: any, id: any) => {
      // console.log(treeSelectData, id);
      // 遍历数组，即遍历某一层节点
      for (let i = 0; i < treeSelectData.length; i++) {
        const node = treeSelectData[i];
        if (node.id === id) {
          // 如果当前节点就是目标节点
          console.log("disabled", node);
          // 将目标节点设置为disabled
          node.disabled = true;

          // 遍历所有子节点，将所有子节点全部都加上disabled
          const children = node.children;
          if (Tool.isNotEmpty(children)) {
            for (let j = 0; j < children.length; j++) {
              setDisable(children, children[j].id);
            }
          }
        } else {
          // 如果当前节点不是目标节点，则到其子节点再找找看。
          const children = node.children;
          if (Tool.isNotEmpty(children)) {
            setDisable(children, id);
          }
        }
      }
    };

    /**
     * 编辑
     */
    const edit = (record: any) => {
      modalVisible.value = true;
      doc.value = Tool.copy(record);
      // docIds.value = [doc.value.doc1Id, doc.value.doc2Id];
      treeSelectData.value = Tool.copy(level1.value);
      setDisable(treeSelectData.value, record.id);
      treeSelectData.value.unshift({ id: 0, name: "根节点" });
      console.log("打印下当权选项id", doc);
    };

    /**
     * 新增
     */
    const add = () => {
      // 清空富文本框
      // editor.txt.html("");
      modalVisible.value = true;
      doc.value = {
        // ebookId: route.query.ebookId
      };

      treeSelectData.value = Tool.copy(level1.value) || [];

      // 为选择树添加一个"无"
      treeSelectData.value.unshift({ id: 0, name: "无" });
    };

    const handleDelete = (id: number) => {
      deleteIds = [];
      getDeleteIds(level1.value,id)
      axios.delete("/doc/delete/" + deleteIds.join(',')).then((response) => {
        const data = response.data; // data = commonResp
        if (data.success) {
          // 重新加载列表
          handleQuery({
            page: pagination.value.current,
            size: pagination.value.pageSize,
          });
        } else {
          message.error(data.message);
        }
      });
    };

    const level1 = ref();
    // let docs: any;
    /**
     * 查询所有文档
     **/
    const handleQueryDoc = () => {
      loading.value = true;
      axios.get("/doc/all").then((response) => {
        loading.value = false;
        const data = response.data;
        if (data.success) {
          // docs = data.content;
          console.log("原始数组：", docs);

          level1.value = [];
          level1.value = Tool.array2Tree(docs, 0);
          console.log("树形结构：", level1.value);

          // 加载完文档后，再加载电子书，否则如果文档树加载很慢，则电子书渲染会报错
          handleQuery({
            page: pagination.value.current,
            size: pagination.value.pageSize,
          });
        } else {
          message.error(data.message);
        }
      });
    };

    const getDocName = (cid: number) => {
      // console.log(cid)
      let result = "";
      // docs.forEach((item: any) => {
      //   if (item.id === cid) {
      //     // return item.name; // 注意，这里直接return不起作用
      //     result = item.name;
      //   }
      // });
      return result;
    };

    const mloading = ref<boolean>(false);
    const visible = ref<boolean>(false);

    const showModal = () => {
      visible.value = true;
    };

    // modal区域
    const handleCancel = () => {
      modalVisible.value = false;
    };

    const modalHandleOk = () => {
      modalLoading.value = true;
      doc.value.ebookId = route.query.ebookId * 1;
      axios.post("/doc/save", doc.value).then((res) => {
        const data = res.data; // commonResp
        if (data.success) {
          modalVisible.value = false;
          modalLoading.value = false;

          //  重新加载列表
          handleQuery({
            page: pagination.value.current,
            size: pagination.value.pageSize,
          });
        } else {
          message.error(data.message);
        }
      });
    };

    // 表单区域
    const formState: UnwrapRef<FormState> = reactive({
      layout: "horizontal",
      fieldA: "",
      fieldB: "",
    });
    const formItemLayout = computed(() => {
      const { layout } = formState;
      return layout === "horizontal"
        ? {
            labelCol: { span: 4 },
            wrapperCol: { span: 14 },
          }
        : {};
    });
    const buttonItemLayout = computed(() => {
      const { layout } = formState;
      return layout === "horizontal"
        ? {
            wrapperCol: { span: 14, offset: 4 },
          }
        : {};
    });

    onMounted(() => {
      // handleQueryDoc();
      // handleQuery()
      handleQuery({
        page: pagination.value.current,
        size: pagination.value.pageSize,
      });
    });

    return {
      param,
      docs,
      pagination,
      columns,
      loading,
      handleTableChange,
      handleQuery,
      getDocName,
      mloading,
      visible,
      showModal,
      modalHandleOk,
      handleCancel,
      formState,
      formItemLayout,
      buttonItemLayout,
      edit,
      add,

      doc,
      modalVisible,
      modalLoading,
      handleModalOk,
      docIds,
      level1,

      handleDelete,
      treeSelectData,
    };
  },
});
</script>

<style scoped>
img {
  width: 50px;
  height: 50px;
}
</style>
