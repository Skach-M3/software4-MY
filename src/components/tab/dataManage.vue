<template>
  <div>
    <!--==========================     头部按钮     ==============================================================-->
    <div id="top_buttons">
      <div id="DiseaseFilter">
        <span>涉及病种：</span>
        <el-select v-model="disease" placeholder="请选择">
          <el-option
            v-for="item in dataDisList"
            :key="item"
            :label="item"
            :value="item"
          >
          </el-option>
        </el-select>
      </div>

      <div id="creatorFilter">
        <span>创建人：</span>
        <el-select v-model="creator" placeholder="请选择">
          <el-option
            v-for="item in dataCreatorList"
            :key="item"
            :label="item"
            :value="item"
          >
          </el-option>
        </el-select>
      </div>
      <el-button @click="clearFilter">清除</el-button>

      <div id="top_right_buttons">
        <el-button type="primary">数据拉取</el-button>
      </div>
      <div id="top_right_buttons">
        <el-button type="primary" @click="importData">导入数据表</el-button>
      </div>
    </div>

    <!--===============================    表格     ==============================================================-->
    <div id="table">
      <el-table
        :data="
          dataList.filter(
            (data) =>
              !(disease || creator) ||
              (data.disease.includes(disease) && data.creator.includes(creator))
          )
        "
        style="width: 100%"
        stripe
        :header-cell-style="{ background: '#eef1f6', color: '#606266' }"
      >
        <el-table-column label="数据表" prop="table_name"> </el-table-column>
        <el-table-column label="涉及病种" prop="disease"> </el-table-column>
        <el-table-column label="样本个数" prop="samplesize"> </el-table-column>
        <el-table-column label="字段个数" prop="featurenumber">
        </el-table-column>
        <el-table-column label="创建人" prop="creator"> </el-table-column>
        <el-table-column label="UID" prop="uid"> </el-table-column>
        <!-- <el-table-column label="创建时间" prop="create_time"> </el-table-column> -->
        <el-table-column align="center">
          <template slot="header">
            <span>操作</span>
          </template>
          <template slot-scope="scope">
            <el-button slot="reference" size="mini" type="primary"  style="margin-right: 10px" @click="getData(scope.row.table_name)"
                >查看</el-button
              >
            <el-popconfirm
              title="删除后无法恢复"
              icon="el-icon-warning"
              icon-color="red"
              @confirm="handleDelete(scope.row)"
            >
              <el-button slot="reference" size="mini" type="danger" :disabled="scope.row.uid != loginUserID"
                >删除</el-button
              >
            </el-popconfirm>
          </template>
        </el-table-column>
      </el-table>
    </div>
      <!-- ==============================     查看数据的对话框 ================================================================ -->
    
<el-dialog
  title="详细数据"
  :visible.sync="DatadialogVisible"
  v-if="DatadialogVisible"
  width="70%">
  <el-table
      :data="patientTable"
      style="width: 100%; margin-top: 20px"
      max-height="450px"
      border
      stripe
      v-loading="getData_loading"
      element-loading-text="正在获取数据"
      element-loading-spinner="el-icon-loading"
    >
      <el-table-column type="index"> </el-table-column>
      <el-table-column
        v-for="(key, index) in Object.keys(patientTable[0])"
        :key="index"
        :label="key"
        :prop="key"
      >
      </el-table-column>
    </el-table>
  <span slot="footer" class="dialog-footer">
    <el-button type="primary" @click="DatadialogVisible = false">确 定</el-button>
  </span>
</el-dialog>
    <!--===============================     导入数据表单   ===================================================================-->
    <el-dialog
      v-loading="loading"
      :element-loading-text="loadText"
      id="importDataTable"
      title="导入数据表"
      :visible.sync="dialogFormVisible"
      width="40%"
    >
      <el-form
        :model="dialogForm"
        ref="dialogFormRef"
        :rules="dialogForm.rules"
        label-width="110px"
      >
        <el-form-item label="选择数据表" prop="filesInfo">
          <el-upload
            action=""
            class="upload"
            ref="uploadRef"
            :on-preview="handlePreview"
            :on-remove="handleRemove"
            :on-change="changeFile"
            :auto-upload="false"
            accept=".csv"
            :limit="1"
            :multiple="false"
            :file-list="dialogForm.filesInfo"
            :http-request="
              (data) => {
                upRequest(data);
              }
            "
          >
            <el-button slot="trigger" size="small" type="success"
              >选取文件</el-button
            >
            <div slot="tip" class="el-upload__tip">只能上传csv文件</div>
          </el-upload>
        </el-form-item>

        <el-form-item label="数据表名称" prop="tableName">
          <el-input
            v-model="dialogForm.tableName"
            placeholder="请输入数据表名称"
          ></el-input>
        </el-form-item>
        <el-form-item label="涉及疾病" prop="dataDisease">
          <el-select
            v-model="dialogForm.dataDisease"
            filterable
            placeholder="请选择或搜索"
          >
            <el-option
              v-for="item in disOptions"
              :key="item.name"
              :label="item.name"
              :value="item.name"
            >
            </el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button
          @click="
            dialogFormVisible = false;
            resetForm('dialogFormRef');
          "
          >取消</el-button
        >
        <el-button @click="resetForm('dialogFormRef')">重置</el-button>
        <el-button type="primary" @click="uploadFile">下一步</el-button>
      </div>

      <el-dialog
        v-loading="loading2"
        :element-loading-text="loadText2"
        append-to-body
        title="请选择特征类型"
        :visible.sync="featuresVision"
      >
        <el-form class="featureLabel" label-width="auto">
          <el-form-item
            v-for="(name, index) in Object.keys(featuresMap)"
            :key="index"
            :label="name"
          >
            <el-select
              v-model="featuresMap[name]"
              placeholder="请选择特征类型"
              @change="changeLabel(name, featuresMap[name])"
            >
              <el-option
                label="人口学特征"
                value="people"
                key="people"
              ></el-option>
              <el-option
                label="社会学特征"
                value="social"
                key="social"
              ></el-option>
              <el-option
                label="生理学特征"
                value="medical"
                key="medical"
              ></el-option>
              <el-option label="标签特征" value="label" key="label"></el-option>
            </el-select>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="primary" @click="compelete">完成上传</el-button>
        </div>
      </el-dialog>
    </el-dialog>
  </div>
</template>

<script>
import { getRequest, postRequest } from "@/api/user";
import { mapGetters, mapMutations, mapState, mapActions } from "vuex";
import { disOptions } from "@/components/tab/constData.js";
import { resetForm, debounce } from "../mixins/mixin";

export default {
  mixins: [resetForm, debounce],
  computed: {
    ...mapGetters(["dataDisList", "dataCreatorList"]),
    ...mapState(["dataList"]),
    loginUserID(){
      return sessionStorage.getItem("userid")
    }
  },

  watch: {
    "dialogForm.tableName"() {
      this.checkTableName();
    },
  },

  data() {
    return {
      loading: false,
      loading2: false,
      getData_loading: false,
      loadText: "拼命加载中",
      loadText2: "拼命加载中",
      disease: "",
      creator: "",
      disOptions,
      featuresVision: false,
      DatadialogVisible:false,
      featuresMap: {},
      patientTable:[],
      dialogForm: {
        filesInfo: [],
        tableName: "",
        isOnly: true,
        dataDisease: "",
        featuresNum: 1,
        fields: [{ name: "", type: "" }],
        rules: {
          tableName: [
            {
              required: true,
              message: "数据表名称不能为空",
              trigger: "change",
            },
          ],
          dataDisease: [
            {
              required: true,
              message: "涉及疾病不能为空",
              trigger: "blur",
            },
          ],
          numFeatures: [
            { required: true, message: "特征个数不能为空", trigger: "blur" },
            { type: "integer", message: "特征个数需为整数", trigger: "blur" },
            { min: 1, message: "特征个数需>1", trigger: "blur" },
          ],
        },
      },
      dialogFormVisible: false,
      options: {
        method: "post",
        data: {},
        url: "/DataTable/upload",
        headers: {
          "Content-Type": "multipart/form-data",
        },
      },
    };
  },

  created() {
    // 检查重名的防抖函数
    this.checkTableName = this.debounce(() => {
      getRequest("/DataTable/inspection", {
        newname: this.dialogForm.tableName,
      }).then((res) => {
        // 上一次重复了这一次不重复就要提醒一下
        if (!this.dialogForm.isOnly && res) {
          this.$message({
            showClose: true,
            message: "表名可用",
            type: "success",
          });
        }
        if (typeof res === "boolean") {
          this.dialogForm.isOnly = res;
        }
        if (!res) {
          this.$message({
            showClose: true,
            message: "数据表重名，请重新填写",
            type: "warning",
          });
          return false;
        }
        return true;
      });
    }, 200);
  },

  methods: {
    ...mapMutations(["SetDataList"]),
    ...mapActions(["getDataList"]),
    getData(tablename) {
      this.DatadialogVisible = true;
      this.getData_loading = true;
      getRequest("/DataTable/getInfoByTableName", {
        tableName: tablename,
      }).then((res) => {
        this.patientTable = res.data;
        console.log(this.patientTable);
        this.getData_loading = false;
      });
    },
    changeLabel(name, label) {
      this.featuresMap[name] = label;
    },

    handleEdit(index, row) {
      console.log(index, row);
    },
    handleDelete(row) {
      postRequest(`DataTable/delete/${row.id}`).then((res) => {
        this.SetDataList(res.reverse());
      });
    },
    clearFilter() {
      this.disease = "";
      this.creator = "";
    },
    importData() {
      this.dialogFormVisible = true;
    },

    handleSubmit() {
      console.log("文件上传中...");
    },

    handleRemove(file, fileList) {
      console.log(file, fileList);
    },
    handlePreview(file) {
      console.log(file);
    },
    beforeUpload(file) {
      //上传前的验证
      const isCSV =
        file.type === "text/csv" || file.type === "application/vnd.ms-excel";
      // const isLt2M = file.size / 1024 / 1024 < 2;

      if (!isCSV) {
        this.$message.error("上传文件仅支持 CSV 格式");
      }
      // if (!isLt2M) {
      //     this.$message.error('上传文件大小不能超过 2MB!');
      // }
      // return isJPG && isLt2M;
      return isCSV;
    },

    removeFileExtension(fileName) {
      // 寻找最后一个点号的位置
      const lastDotIndex = fileName.lastIndexOf(".");

      // 如果找到了点号并且不在文件名的开头或结尾
      if (
        lastDotIndex !== -1 &&
        lastDotIndex < fileName.length - 1 &&
        lastDotIndex > 0
      ) {
        // 返回删除了后缀的文件名部分
        return fileName.substring(0, lastDotIndex);
      } else {
        // 如果没有找到点号或者点号在文件名的开头或结尾，直接返回原文件名
        return fileName;
      }
    },

    changeFile() {
      console.log(this.$refs["uploadRef"].uploadFiles);
      if (this.dialogForm.tableName.length < 1) {
        this.dialogForm.tableName = this.removeFileExtension(
          this.$refs["uploadRef"].uploadFiles[0].name
        );
      }
    },

    // 数据表上传函数
    upRequest(data) {
      const payload = new FormData();
      payload.append("file", data.file);
      payload.append("newName", this.dialogForm.tableName);
      payload.append("disease", this.dialogForm.dataDisease);
      payload.append("user", sessionStorage.getItem("username"));
      payload.append("uid", sessionStorage.getItem("userid")-0);
      this.options = {
        method: "post",
        data: payload,
        url: "/DataTable/upload",
        headers: {
          "Content-Type": "multipart/form-data",
        },
      };
      this.$axios(this.options).then((res) => {
        this.loading = false;
        console.log(res);
        if (res?.code == "200") {
          this.$message({
            showClose: true,
            type: "success",
            message: "解析成功",
          });
          console.log(payload);
          let featureList = res.tableHeaders;
          // 把特征存为map的键
          for (const item of featureList) {
            this.$set(this.featuresMap, item, "people");
          }
          this.featuresVision = true;
        } else {
          this.$message({
            showClose: true,
            type: "error",
            message: "解析失败",
          });
        }
      });
    },

    uploadFile() {
      if (this.$refs["uploadRef"].uploadFiles.length < 1) {
        this.$message({
          showClose: true,
          type: "warning",
          message: "请选择数据表",
        });
        return false;
      }
      this.checkTableName();
      if (!this.dialogForm.isOnly) {
        return false;
      }
      this.$refs["dialogFormRef"].validate((valid) => {
        if (valid) {
          this.loadText = "正在上传并解析文件";
          this.loading = true;
          this.$refs.uploadRef.submit();
        }
      });
    },

    compelete() {
      // 判断多标签合理性
      let labelCount = 0;
      for (const key in this.featuresMap) {
        if (Object.hasOwnProperty.call(this.featuresMap, key)) {
          if (this.featuresMap[key] == "label") {
            labelCount++;
          }
        }
      }
      if (labelCount < 1) {
        this.$message({
          showClose: true,
          type: "warning",
          message: "请至少设置一个标签特征",
        });
        return false;
      }
      if (this.dialogForm.dataDisease != "多疾病" && labelCount > 1) {
        this.$message({
          showClose: true,
          type: "warning",
          message: "只有多病种数据集能设置多个标签特征",
        });
        return false;
      }

      this.loadText2 = "正在添加字段类型";
      this.loading2 = true;
      let tableHeaders = [];
      for (const key in this.featuresMap) {
        if (Object.hasOwnProperty.call(this.featuresMap, key)) {
          switch (this.featuresMap[key]) {
            case "people":
              tableHeaders.push({
                fieldName: key,
                isDemography: "1",
              });
              break;
            case "medical":
              tableHeaders.push({
                fieldName: key,
                // isDemography: "0",
                isPhysiological: "1",
                // isSociology: "0",
              });
              break;
            case "social":
              tableHeaders.push({
                fieldName: key,
                // isDemography: "0",
                // isPhysiological: "0",
                isSociology: "1",
              });
              break;
            case "label":
              tableHeaders.push({
                fieldName: key,
                isLabel: "1",
              });
              break;
            default:
              break;
          }
        }
      }
      let userId = sessionStorage.getItem("userid")-0;
      // 上传特征分类结果
      postRequest("/tTableManager/insertTableManager", {
        tableName: this.dialogForm.tableName,
        tableHeaders,
        userId
      }).then((res) => {
        console.log(res);
        // this.$message({
        //   showClose: true,
        //   type: "success",
        //   message: "操作成功，已添加数据表",
        // });
      });

      // 重新上传数据表，使其保存到数据列表中
      // 此处上传时后台已有数据表，可和后台配合只发送保存通知已提高效率
      this.options.url = "/DataTable/uploadTable";
      this.$axios(this.options).then((res) => {
        
        this.loading2 = false;
        this.resetForm('dialogFormRef');
        if (res?.code == "200") {
          this.$message({
            showClose: true,
            type: "success",
            message: "上传成功",
          });
          this.featuresVision = false;
          this.dialogFormVisible = false;
          this.getDataList();
        } else {
          this.$message({
            showClose: true,
            type: "error",
            message: "上传失败",
          });
        }
      });
    },
  },
};
</script>

<style scoped>
#top_buttons > * {
  display: inline-block;
}
#DiseaseFilter {
  margin-right: 40px;
}
#creatorFilter {
  margin-right: 20px;
}
#top_right_buttons {
  float: right;
  margin-right: 100px;
}
#table {
  margin-top: 10px;
}

.featureLabel {
  height: 55vh;
  width: 100%;
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  overflow: auto;
}

.featureLabel .el-form-item__label {
  color: #252525;
}
</style>
