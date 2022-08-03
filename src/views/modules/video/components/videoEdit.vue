<template>
  <el-dialog
    :title="title"
    :visible.sync="dialogFormVisible"
    :close-on-click-modal="false"
    width="800px"
    @close="close"
  >
    <el-form ref="form" :model="form" :rules="rules" label-width="105px">
      <el-row :gutter="10" >
        <el-col :xs="24" :sm="24" :md="24" :lg="12" :xl="12">
          <el-form-item label="名称" prop="name">
            <el-input v-model="form.name" autocomplete="off"></el-input>
          </el-form-item>
        </el-col>

        <el-col :xs="24" :sm="24" :md="24" :lg="12" :xl="12">
          <el-form-item label="类型" prop="type">
            <el-select v-model="form.type" placeholder="请选择" style="width: 100%">
              <el-option
                v-for="item in dict.resource_video_type"
                :key="item.dictValue"
                :label="item.dictName"
                :value="item.dictValue"
              ></el-option>
            </el-select>
          </el-form-item>
        </el-col>

        <el-col :xs="24" :sm="24" :md="24" :lg="12" :xl="12">
                <el-form-item label="名称" prop="label">
                  <el-input v-model="form.label" autocomplete="off" placeholder="请输入标签,多个标签用逗号隔开"></el-input>
                </el-form-item>
        </el-col>

        <el-col :xs="24" :sm="24" :md="24" :lg="12" :xl="12">
          <el-form-item label="封面链接" prop="coverImg">
            <el-input v-model="form.coverImg" autocomplete="off" placeholder="请输入封面的图片url"></el-input>
          </el-form-item>
        </el-col>
          <el-col :xs="24" :sm="24" :md="24" :lg="12" :xl="12">
          <el-form-item label="视频链接" prop="url">
            <el-input v-model="form.url" autocomplete="off" placeholder="请输入视频的url"></el-input>
          </el-form-item>
        </el-col>
      </el-row>

    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button @click="close">取 消</el-button>
      <el-button type="primary" @click="save">确 定</el-button>
    </div>
  </el-dialog>
</template>

<script>
  import { doInsert, doUpdate } from "@/api/video/video";
  import { isName, isNull,isUrl} from "@/utils/validate";

  export default {
    name: "VideoEdit",
    data() {

      const validateName = (rule, value, callback) => {
        if (isNull(value)) {
          callback(new Error("请输入名称"));
        } else {
          callback();
        }
      };

      const validateUrl = (rule, value, callback) => {
        if (isNull(value)) {
          callback(new Error("请输入url"));
        } else if(!isUrl(value)){
          callback(new Error("请输入正确的url"));
        }else {
          callback();
        }
      };

      return {
        form: {
          type: '',
          // 设置默认值
          version: 0
        },
        dict: {},
        rules: {
          name: [
            { required: true, trigger: "blur", validator: validateName },
          ], 
          url: [
            { required: true, trigger: "blur", validator: validateUrl },
          ],
          coverImg: [
            { required: true, trigger: "blur", validator: validateUrl },
          ]
        },
        title: "",
        dialogFormVisible: false,
      };
    },
    created() {

    },
    mounted() {
      // 如果不是每次开启时查询 在created中 有可能会短暂查不到
      this.dict.resource_video_type = this.$getDictList("resource_video_type");
    },
    methods: {
      showEdit(row) {
        if (!row) {
          this.title = "添加";
        } else {
          this.title = "编辑";
          this.form = Object.assign({}, row);
        }
        this.dialogFormVisible = true;
      },
      close() {
        this.dialogFormVisible = false;
        this.$refs["form"].resetFields();
        this.form = this.$options.data().form;
      },
      save() {
        this.$refs["form"].validate(async (valid) => {
          if (valid) {
            // 修改
            if (!isNull(this.form.id)) {
              const { success, msg } = await doUpdate(this.form);
              if(success){
                this.$baseMessage(msg, "success");
              }
            } else {
              const { success, msg } = await doInsert(this.form);
              if(success){
                this.$baseMessage(msg, "success");
              }
            }

            await this.$emit("fetchData");
            this.close();
          } else {
            return false;
          }
        });
      },
    },
  };
</script>
