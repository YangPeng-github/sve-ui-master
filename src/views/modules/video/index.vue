<template>
  <div class="tenantManagement-container">

      <el-collapse-transition>
      <div v-show="this.moreQueryFlag" class="more-query">
        <!-- 更多查找 -->
        <vab-query-form>
          <vab-query-form-left-panel :span="24">
            <el-form :inline="true" :model="queryForm" @submit.native.prevent>
              <el-form-item>
                <el-date-picker
                  v-model="createTimePicker"
                  type="datetimerange"
                  start-placeholder="开始时间"
                  end-placeholder="结束时间"
                  :default-time="['00:00:00']">
                </el-date-picker>
               
              </el-form-item>
            </el-form>
          </vab-query-form-left-panel>
        </vab-query-form>
        <el-divider></el-divider>
      </div>
    </el-collapse-transition>

    <!-- 主要操作  -->
    <vab-query-form>
      <vab-query-form-left-panel :span="12">
        <el-button
          v-if="$perms('resource_video_insert')"
          icon="el-icon-plus"
          type="primary"
          @click="handleInsert"
        > 添加 </el-button>

        <el-button
          v-if="$perms('gentest_test_import')"
          icon="el-icon-upload2"
          type="warning"
          @click="handleImportExcel"
        > 导入 </el-button>

        <el-button
          v-if="$perms('gentest_test_export')"
          icon="el-icon-download"
          type="warning"
          @click="handleExportExcel"
        > 导出 </el-button>

        <el-button
          v-if="$perms('gentest_test_delete')"
          :disabled="!selectRows.length > 0"
          icon="el-icon-delete"
          type="danger"
          @click="handleDelete"
        > 批量删除 </el-button>

      </vab-query-form-left-panel>
      <vab-query-form-right-panel :span="12">
        <el-form :inline="true" :model="queryForm" @submit.native.prevent>

          <el-form-item>
            <el-select v-model="queryForm.type_EQ" placeholder="请选择类型" clearable style="width: 100%">
              <el-option
                v-for="item in dict.resource_video_type"
                :key="item.dictValue"
                :label="item.dictName"
                :value="item.dictValue"
              ></el-option>
            </el-select>
          </el-form-item>

          <el-form-item>
            <el-input
              v-model.trim="queryForm.name_LIKE"
              placeholder="请输入名称"
              clearable
            />
          </el-form-item>

          <el-form-item>
            <el-button icon="el-icon-search" type="primary" @click="queryData">
              查询
            </el-button>

            <el-button icon="el-icon-search" @click="moreQuery">
              更多
            </el-button>
          </el-form-item>
        </el-form>
      </vab-query-form-right-panel>
    </vab-query-form>

    <el-table
      v-loading="listLoading"
      :data="list"
      :element-loading-text="elementLoadingText"
      @selection-change="setSelectRows"
      @sort-change="tableSortChange"
    >
      <el-table-column show-overflow-tooltip type="selection"></el-table-column>

      <el-table-column show-overflow-tooltip label="序号" width="95">
        <template slot-scope="scope">
          {{(queryForm.pageNo - 1) * queryForm.pageSize + scope.$index + 1}}
        </template>
      </el-table-column>

      <el-table-column
        show-overflow-tooltip
        prop="name"
        label="名称"
      ></el-table-column>

      <el-table-column
        show-overflow-tooltip
        prop="type"
        label="分类"
      >
        <template slot-scope="scope">
          <span>
            {{ $getDictNameByValue('resource_video_type', scope.row.type) }}
          </span>
        </template>
      </el-table-column>

      <el-table-column
        show-overflow-tooltip
        prop="label"
        label="标签"
      ></el-table-column>

       <el-table-column 
        show-overflow-tooltip 
        label="封面"  width="100">
        <template slot-scope="scope">
          <el-image
            v-if="imgShow"
            :preview-src-list="imageList"
            :src="scope.row.coverImg"
          ></el-image>
        </template>
      </el-table-column>

       <el-table-column
        show-overflow-tooltip
        prop="createTime"
        label="时间"
      >
      </el-table-column>

      <el-table-column
        fixed="right"
        show-overflow-tooltip
        label="操作"
        width="180"
      >
        <template v-slot="scope">
           <el-button
            v-if="$perms('resource_video_select')"
            type="text"
            @click="handleSee(scope.row)"
          > 播放 </el-button>

           <el-divider direction="vertical"></el-divider>

          <el-button
            v-if="$perms('resource_video_update')"
            type="text"
            @click="handleUpdate(scope.row)"
          > 编辑 </el-button>

          <el-divider direction="vertical"></el-divider>

          <el-button
            v-if="$perms('resource_video_delete')"
            type="text"
            @click="handleDelete(scope.row)"
          > 删除 </el-button>
        
          
        </template>

      </el-table-column>
    </el-table>

    <el-pagination
      background
      :current-page="queryForm.pageNo"
      :page-size="queryForm.pageSize"
      :layout="layout"
      :total="total"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    ></el-pagination>
    <see ref="see" @fetchData="fetchData"></see>
    <edit ref="edit" @fetchData="fetchData"></edit>
    <import ref="import" @fetchData="fetchData" ></import>
  </div>
</template>

<script>
  import { getList, doDelete, doDeleteAll, doExportExcel } from "@/api/video/video";
  import See from "./components/videoSee";
  import Edit from "./components/videoEdit";
  import Import from "./components/videoInput";

  import { vueButtonClickBan } from "@/utils";

  import {isNull} from "@/utils/validate";

  import { isNotNull } from "@/utils/valiargs";

  import { formateDate } from "@/utils/format";

  export default {
    name: "Video",
    components: { See,Edit, Import},
    data() {
      return {
        imgShow:true,
        list: null,
        imageList: [],
        listLoading: true,
        layout: "total, sizes, prev, pager, next, jumper",
        total: 0,
        selectRows: "",
        elementLoadingText: "正在加载...",
        createTimePicker:[],
        moreQueryFlag: false,
        queryForm: {
          pageNo: 1,
          pageSize: 10,
          type_EQ:"",
          name_LIKE: "",
          createTime_BEGIN: "",
          createTime_END: "",
        },
        dict:{}
      };
    },
    created() {
      this.fetchData();
    },
    mounted() {
      this.dict.resource_video_type = this.$getDictList("resource_video_type");
    },
    methods: {
      tableSortChange() {
        const imageList = [];
        this.$refs.tableSort.tableData.forEach((item, index) => {
          imageList.push(item.img);
        });
        this.imageList = imageList;
      },
      setSelectRows(val) {
        this.selectRows = val;
      },
      handleInsert(row) {
        this.$refs["edit"].showEdit();
      },
      handleSee(row) {
        if (row.id) {
          this.$refs["see"].showSee(row);
        }
      },
      handleUpdate(row) {
        if (row.id) {
          this.$refs["edit"].showEdit(row);
        }
      },
      handleDelete(row) {
        if (row.id) {
          this.$baseConfirm("你确定要删除当前项吗", null, async () => {
            const { msg } = await doDelete({ id: row.id });
            this.$baseMessage(msg, "success");
            await this.fetchData();
          });
        } else {
          if (this.selectRows.length > 0) {
            const ids = this.selectRows.map((item) => item.id).join();
            this.$baseConfirm("你确定要删除选中项吗", null, async () => {
              const { msg } = await doDeleteAll({ ids });
              this.$baseMessage(msg, "success");
              await this.fetchData();
            });
          } else {
            this.$baseMessage("未选中任何行", "error");
            return false;
          }
        }
      },
      // 导出excel
      handleExportExcel(el){
        // 导出按钮防抖处理 默认限制为10秒
        vueButtonClickBan(el, 10);

        // 执行导出
        doExportExcel(this.queryForm);
      },
      // 导入excel
      handleImportExcel(){
        this.$refs["import"].show();
      },

      handleSizeChange(val) {
        this.queryForm.pageSize = val;
        this.fetchData();
      },
      handleCurrentChange(val) {
        this.queryForm.pageNo = val;
        this.fetchData();
      },
      moreQuery(){
        this.moreQueryFlag = !this.moreQueryFlag;
      },
      queryData() {
        if (
          isNotNull(this.createTimePicker) &&
          this.createTimePicker.length === 2
        ) {
          this.queryForm.createTime_BEGIN =
            this.createTimePicker.length === 0
              ? ""
              : formateDate(
                  this.createTimePicker[0],
                  "yyyy-MM-dd hh:mm:ss"
                );
          this.queryForm.createTime_END =
            this.createTimePicker.length === 0
              ? ""
              : formateDate(
                  this.createTimePicker[1],
                  "yyyy-MM-dd hh:mm:ss"
                );
        } else {
          this.queryForm.createTime_BEGIN = "";
          this.queryForm.createTime_END = "";
        }
        this.queryForm.pageNo = 1;
        this.fetchData();
      },
      async fetchData() {
        this.listLoading = true;
        const { data } = await getList(this.queryForm);
        if(!isNull(data)){
            this.list = data.rows;
            const imageList = [];
            data.rows.forEach((item, index) => {
              imageList.push(item.coverImg);
            });
            this.imageList = imageList;
            this.total = data.total;
          }

        setTimeout(() => {
          this.listLoading = false;
        }, 300);
      },
    },
  };
</script>
