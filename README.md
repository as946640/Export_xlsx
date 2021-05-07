
# 前端Vue Excel 导出 基于 Element

**安装**

    1 npm i xlsx  xlsx-style --save 

    2 src目录下面新建 vendor文件夹

    3 下载当前js文件 把js文件放入 vendor文件夹



**配置项**

配置 | 值 | 类型 | 备注
---|------|------|---
open | false | Boolean | 导出弹窗显示
Params | {pageNum:1, pageSize: 10} | Object | 列表数据条件查询，具体参数看自己后端
fieldTranslate | [{key:'merID',contText:{1:'翻译1',2: '翻译2'}},] | Array | 字段翻译配置
exportData | 具体参数看下面 | Object | 表格配置
extraKey | [{name: "表头", key: key}] | Array |exportData配置: 列表额外字段导出 key => 字段
filename | "Xlsx导出" | String | exportData配置:表格名称
total | "0" | String | exportData配置:总条数
tabUrl | "xxx" | String |exportData配置:列表Url 
columns | "xxx" | Array | 表格列 父级传递过来 具体方法看下面 

## dom案列

**Html**


```html
 <!-- 导出组件 -->
    <Export
      :open="ExportOpen"
      :exportData="exportData"
      :fieldTranslate="fieldTranslate"
      :columns="columns"
      :Params="queryParams"
      @clase="ExportOpen = false"
    />
```

**Js**


```js
// data 定义

//导出表格列数据
    columns: [],
    //导出字段翻译
    fieldTranslate: [{ key: "status", contText: { 0: "成功", 1: "失败" } }],
    //导出数据
    exportData: {
        filename: "短信发送记录",
        tabUrl: "/system/smsRecord/list",
        total: 0
     },
    //导出弹窗显示
    ExportOpen: false,
    
    
<!--列表数据获取-->
    getList() {
      this.loading = true;
      getSmsRecordList(this.queryParams).then(response => {
        this.smsTemplateList = response.rows;
        this.total = response.total;
        this.loading = false;
        // 表格导出 总条数
        this.exportData.total = response.total;
        // 表格列数据 map设置 默认 选中
        this.columns = this.$refs.table.columns.map(item => {
          item.checked = true;
          return item;
        });
        
      });
    },
```

## 效果展示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210507165237493.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTE5MjQyMQ==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210507165323740.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTE5MjQyMQ==,size_16,color_FFFFFF,t_70)



