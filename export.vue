<!--
 * @Descripttion : 
 * @Autor        : 阿森
 * @Date         : 2021-02-20 11:48:48
 * @LastEditTime: 2021-05-07 15:21:56
 * @open         : 导出弹窗 开启
 * @exportData   : 导出配置项   filename 文件名 total： 总条数 tabUrl： 表格数据接口
 * @fieldTranslate: 字段翻译配置 数组形式 [{key:'merID',contText:{1:'翻译1',2: '翻译2'}},] key值对应 =》 contText 键
 * @extraKey     :额外得 字段导出 数组 name：对应表头 key：对应字段
 * @columns      : 表格列数据 在父组件获取传递过来 获取方法： this.columns = this.$refs.table.columns.map((item) => {item.checked = true;return item;});
 * @FilePath     : \vue-admin-template\src\components\export\index.vue
-->
<template>
  <div>
    <el-dialog
      title="Xlsx导出"
      :visible.sync="open"
      width="30%"
      @close="close"
      center
    >
      <div class="excel-export-page">
        <div class="export-dialog">
          <div
            style="
              display: flex;
              justify-content: space-between;
              align-items: center;
            "
          >
            <span>页码</span>
            <div style="display: flex; align-items: center">
              <div>
                <el-input
                  v-model.number="queryParams.pageNum"
                  style="width: 80px"
                  size="mini"
                />
                <span>页</span>
              </div>
              <div>
                <el-input
                  v-model.number="queryParams.pageSize"
                  style="width: 80px"
                  size="mini"
                />
                <span>条</span>
              </div>
              <span style="padding: 0 20px">所有页</span>
              <el-checkbox v-model="getMore" @change="exprotAll" />
            </div>
          </div>
          <div style="color: blue">
            <span style="color: red">(导出最大量数据请在空闲时操作)</span>
          </div>
          <div class="export-clomon-choice">
            <el-checkbox-group v-model="checkedCities">
              <div
                class="export-row-style"
                v-for="(item, index) in columns"
                :key="index"
              >
                <el-checkbox
                  v-if="item.label && item.label != '操作'"
                  :label="item.label"
                  @change="
                    (val) => {
                      handleCheckedCitiesChange(val, item.property);
                    }
                  "
                >
                  <span style="color: #3e3f41"> {{ item.label }}</span>
                </el-checkbox>
              </div>
            </el-checkbox-group>
          </div>
        </div>
      </div>
      <span slot="footer">
        <div class="export-btn-style">
          <el-button
            type="primary"
            size="mini"
            icon="el-icon-download"
            @click="getExportDt"
            >导出</el-button
          >
          <el-button type="wraty" size="mini" @click="close">取消</el-button>
        </div>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import request from "@/utils/request";
import Blob from "@/vendor/Blob";
import Export2Excel from "@/vendor/Export2Excel";
export default {
  name: "excel",
  props: {
    open: {
      type: Boolean,
      default: () => {
        return false;
      },
    },
    Params: {
      type: Object,
      default: () => {
        return {
          pageNum: 1,
          pageSize: 10,
        };
      },
    },
    // 字段翻译配置
    fieldTranslate: {
      type: Array,
      default: () => {
        return [];
      },
    },
    //   表格全部列数据 需要导出的数据
    columns: {
      type: Array,
    },
    // 导出配置项
    exportData: {
      type: Object,
      default: () => {
        return {
          filename: "Xlsx导出",
          total: 0,
          tabUrl: "",
          phoneKey: "",
          // 额外得 字段导出配置
          extraKey: [],
        };
      },
    },
  },
  data() {
    return {
      // 加载
      //   总条数
      total: 0,
      getMore: true, //是否是全部
      maxPage: 0,
      // 导出分页数据 以及导出数据
      exportFeild: {
        // 导出 头目
        tableCol: [],
      },
      // 导出标题
      checkedCities: [],
      // 导出的 key值
      filterVal: [],
      // 表格数据
      tableData: [],
      // 搜索对象
      queryParams: {
        pageNum: 1,
        pageSize: 10,
      },
      // 导出样式配置
      defaultFontStyle: {
        font: { name: "宋体", sz: 11, color: { auto: 1 } },
        border: { color: { auto: 1 } },
        aligenment: {
          // 自动换行
          wrapTex: 1,
          // 居中
          horizontar: "center",
          vertical: "center",
          indent: 0,
        },
      },
    };
  },
  watch: {
    columns: {
      handler(newVal) {
        this.$set(this.exportFeild, "tableCol", newVal);
        this.filterVal = [];
        this.checkedCities = [];
        newVal.map((item) => {
          if (item.property && item.label != "操作") {
            this.checkedCities.push(item.label);
            this.filterVal.push(item.property);
          }
        });
      },
      deep: true,
      immediate: true,
    },
    Params: {
      handler(newVal) {
        this.queryParams = newVal;
      },
      deep: true,
      immediate: true,
    },
    // 监听导出配置项得 额外字段
    "exportData": {
      handler(newVal, oldVal) {
        if (newVal.extraKey && newVal.extraKey.length > 0) {
          for (const item of newVal.extraKey) {
            let obj = {
              label: null,
              property: null,
            };
            console.log(item);
            obj.label = item.name;
            obj.property = item.key;
            this.columns.unshift(obj);
            this.checkedCities.unshift(item.name);
            this.filterVal.unshift(item.key);
          }
        }
      },
      deep: true,
      // immediate: true,
    },
  },
  computed: {
    expressStatus() {
      return (status) => {
        let text = "";
        switch (parseInt(status)) {
          case 1:
            return (text = "等待获取");
            break;
          case 2:
            return (text = "已经获取");
            break;
          case 3:
            return (text = "以及退款");
            break;
          default:
            return (text = "未知状态");
            break;
        }
      };
    },
  },
  created() {

  },
  methods: {
    // 是否导出最大量
    exprotAll(val) {
      // console.log(val);
      if (val) {
        this.queryParams.pageNum = 1;
        this.queryParams.pageSize = this.exportData.total;
      }
    },
    // 导出数据到Excel表
    exportExcel() {
      require.ensure([], () => {
        const { export_json_to_excel } = require("@/vendor/Export2Excel");
        const tHeader = this.checkedCities;
        const filterVal = this.filterVal;
        const list = this.tableData;
        const data = this.formatJson(filterVal, list);
        export_json_to_excel(tHeader, data, this.exportData.filename);
      });
    },
    // 获取导出的数据，执行导出 与展示的表格数据不一定是一致的
    getExportDt() {
      if (!this.getMore) {
        //    导出当前页数
      } else {
        // 导出全部数据，可输入页码，让用户可选
        this.queryParams.pageNum = 1;
        this.queryParams.pageSize = this.exportData.total;
      }
      // 发起网络请求
      request({
        url: this.exportData.tabUrl,
        method: "get",
        params: this.queryParams,
      }).then((res) => {
        if (res.code === 200) {
          this.tableData = res.rows;
          this.exportExcel();
        }
      });
    },
    formatJson(filterVal, jsonData) {
      let arrs = [];
      for (const _inx in jsonData) {
        arrs.push([]);
        filterVal.forEach((key) => {
          if (this.exportData.phoneKey && key == this.exportData.phoneKey) {
            const phone = this.phoneNumFilter(jsonData[_inx][key]);
            arrs[_inx].push(phone);
            return true;
          }
          //   判断是否需要状态翻译
          if (this.fieldTranslate.length > 0) {
            // 拿取到当前 key 对应的翻译对象
            let fieldKye = this.fieldTranslate.find((item) => {
              return item.key === key;
            });
            if (fieldKye != undefined) {
              arrs[_inx].push(fieldKye.contText[jsonData[_inx][key]]);
              // 跳过当前循环
              return true;
            }
          }
          //   判断是否是多级 key
          if (key && key.split(",").length > 1) {
            //   再次循环 key
            let teplate = "";
            for (const keys of key.split(",")) {
              let item = jsonData[_inx];

              // 是否多级对象key
              if (keys.split(".").length > 1) {

                teplate += eval("item." + keys);
              } else {
                teplate += item[keys];
              }
            }
            arrs[_inx].push(teplate);
            // 跳过当前循环
            return true;
          }
          // 是否是深度导出
          if (key.split(".").length > 1) {
            console.log("进入深度导出");
            // 二级对象 键
            let secondKey = key.split(".");
            // 判断深度对象 是否是数组
            if (jsonData[_inx][secondKey[0]].length > 0) {
              let items = jsonData[_inx][secondKey[0]][0][secondKey[1]];
              arrs[_inx].push(items);
              return true;
            }
            // 二级对象 值
            let items = jsonData[_inx][secondKey[0]][secondKey[1]];
            arrs[_inx].push(items);
            // 跳过当前循环
            return true;
          } else {
            arrs[_inx].push(jsonData[_inx][key]);
            // 跳过当前循环
            return true;
          }
        });
      }
      // console.log(arrs);
      return arrs;
    },
    // 勾选change
    handleCheckedCitiesChange(val, key) {
      console.log(this.filterVal.includes(key));
      if (val && !this.filterVal.includes(key)) {
        this.filterVal.push(key);
      } else {
        const _keyIndex = this.filterVal.findIndex((item) => {
          return item == key;
        });
        this.filterVal.splice(_keyIndex, 1);
      }
    },
    // 取消
    close() {
      this.queryParams.pageSize = 10;
      this.$emit("clase", false);
    },
    phoneNumFilter(phone) {
      // 1字符串转化成数组

      let phoneArr = [...phone];

      // 2.将数组中的4-7位变成*
      let str = "";
      phoneArr.map((res, index) => {
        if (index > 2 && index < 7) {
          str += "*";
          // return '*';
        } else {
          str += res;
          // return res;
        }
      });
      return str;
    },
  },
};
</script>

<style lang="scss" scoped>
.excel-export-page {
  width: 100%;
  padding: 20px;
}
.export-dialog {
  width: 100%;
}
.export-clomon-choice {
  height: 150px;
  border: 1px solid #aecaf0;
  overflow-y: scroll;
  margin-top: 2px;
  .export-row-style {
    border-top: 1px solid #ededed;
    padding-left: 10px;
  }
}
.export-btn-style {
  padding: 10px;
  display: flex;
  justify-content: center;
}
</style>
