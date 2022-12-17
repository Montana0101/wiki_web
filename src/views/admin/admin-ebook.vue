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
          :data-source="ebooks"
          :pagination="pagination"
          :loading="loading"
          @change="handleTableChange"
        >
          <template #cover="{ text: cover }">
            <img v-if="cover" :src="cover" alt="avatar" />
          </template>
          <!-- <template v-slot:category="{ text, record }">
          <span
            >{{ getCategoryName(record.category1Id) }} /
            {{ getCategoryName(record.category2Id) }}</span
          >
        </template> -->
          <template v-slot:action="{ text, record }">
            <a-space size="small">
              <!-- <router-link :to="'/admin/doc?ebookId=' + record.id">
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

    <a-modal v-model:visible="modalVisible" title="Title" @ok="handleOk">
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
      <a-form
        :layout="formState.layout"
        :model="ebook"
        v-bind="formItemLayout"
      >
        <a-form-item label="封面">
          <a-input v-model:value="ebook.cover" placeholder="请输入封面" />
        </a-form-item>
        <a-form-item label="名称">
          <a-input v-model:value="ebook.name" placeholder="请输入名称" />
        </a-form-item>
        <a-form-item label="分类一">
          <a-input
            v-model:value="ebook.category1Id"
            placeholder="请输入分类一"
          />
        </a-form-item>
        <a-form-item label="分类二">
          <a-input
            v-model:value="ebook.category2Id"
            placeholder="请输入分类二"
          />
        </a-form-item>
        <a-form-item label="描述">
          <a-input v-model:value="ebook.desc" placeholder="请输入描述" />
        </a-form-item>
        <!-- <a-form-item :wrapper-col="buttonItemLayout.wrapperCol">
          <a-button type="primary">Submit</a-button>
        </a-form-item> -->
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

interface FormState {
  layout: "horizontal" | "vertical" | "inline";
  fieldA: string;
  fieldB: string;
}

export default defineComponent({
  name: "AdminEbook",
  setup() {
    const param = ref();
    param.value = {};
    const ebooks = ref();
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
        title: "分类",
        slots: { customRender: "category" },
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
      // 如果不清空现有数据，则编辑保存重新加载数据后，再点编辑，则列表显示的还是编辑前的数据
      ebooks.value = [];
      axios
        .get("/ebook/list", {
          params: {
            page: params.page,
            size: params.size,
            name: param.value.name,
          },
        })
        .then((response) => {
          console.log("打印下别的撒手锏", response);
          loading.value = false;
          const data = response.data;
          if (data.success) {
            ebooks.value = data.content.list;

            // 重置分页按钮
            pagination.value.current = params.page;
            pagination.value.total = data.content.total;
            console.log("打印下数据",data.content.total)
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
    const categoryIds = ref();
    const ebook = ref();
    const modalVisible = ref(false);
    const modalLoading = ref(false);
    const handleModalOk = () => {
      modalLoading.value = true;
      console.log("打印下表单数据",formState)
      // ebook.value.category1Id = categoryIds.value[0];
      // ebook.value.category2Id = categoryIds.value[1];
      // axios.post("/ebook/save", ebook.value).then((response) => {
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

    /**
     * 编辑
     */
    const edit = (record: any) => {
      modalVisible.value = true;
      ebook.value = Tool.copy(record);
      categoryIds.value = [ebook.value.category1Id, ebook.value.category2Id];
    };

    /**
     * 新增
     */
    const add = () => {
      modalVisible.value = true;
      ebook.value = {};
    };

    const handleDelete = (id: number) => {
      axios.delete("/ebook/delete/" + id).then((response) => {
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
    let categorys: any;
    /**
     * 查询所有分类
     **/
    const handleQueryCategory = () => {
      loading.value = true;
      axios.get("/category/all").then((response) => {
        loading.value = false;
        const data = response.data;
        if (data.success) {
          categorys = data.content;
          console.log("原始数组：", categorys);

          level1.value = [];
          level1.value = Tool.array2Tree(categorys, 0);
          console.log("树形结构：", level1.value);

          // 加载完分类后，再加载电子书，否则如果分类树加载很慢，则电子书渲染会报错
          handleQuery({
            page: pagination.value.current,
            size: pagination.value.pageSize,
          });
        } else {
          message.error(data.message);
        }
      });
    };

    const getCategoryName = (cid: number) => {
      // console.log(cid)
      let result = "";
      categorys.forEach((item: any) => {
        if (item.id === cid) {
          // return item.name; // 注意，这里直接return不起作用
          result = item.name;
        }
      });
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
      axios.post('/ebook/save',ebook.value).then(res=>{
        const data = res.data; // commonResp
        if(data.success){
          modalVisible.value = false;
          modalLoading.value = false;

        //  重新加载列表
          handleQuery({
            page:pagination.value.current,
            size:pagination.value.pageSize
          })
        }else {
          message.error(data.message)
        }
      })
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
      // handleQueryCategory();
      // handleQuery()
      handleQuery({
        page: pagination.value.current,
        size: pagination.value.pageSize,
      });
    });

    return {
      param,
      ebooks,
      pagination,
      columns,
      loading,
      handleTableChange,
      handleQuery,
      getCategoryName,
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

      ebook,
      modalVisible,
      modalLoading,
      handleModalOk,
      categoryIds,
      level1,

      handleDelete,
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
