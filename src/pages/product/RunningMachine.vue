<template>
  <v-card>
    <v-card-title>
      <v-btn color="primary" @click="addrunningMachine">新增跑步机</v-btn>
      <!--搜索框，与search属性关联-->
      <v-spacer/>
      <v-flex xs3>
        <v-text-field label="输入id关键字搜索" v-model.lazy="search" append-icon="search" hide-details/>
      </v-flex>
    </v-card-title>
    <v-divider/>
    <v-data-table
      :headers="headers"
      :items="categories"
      :pagination.sync="pagination"
      :total-items="totalCategories"
      :loading="loading"
      class="elevation-1"
    >
      <template slot="items" slot-scope="props">
        <td class="text-xs-center">{{ props.item.id }}</td>
        <td class="text-xs-center">{{ props.item.cname }}</td>
        <td class="text-xs-center">{{ props.item.currentVersionCode }}</td>
        <td class="text-xs-center">{{ props.item.customerLevel }}</td>
        <td class="text-xs-center">{{ props.item.runtimeMileage }}</td>
        <td class="text-xs-center">{{ props.item.country }}</td>
        <td class="text-xs-center">{{ props.item.province }}</td>
        <td class="text-xs-center">{{ props.item.address }}</td>



        <td class="justify-center layout px-0">
          <v-btn icon @click="editBrand(props.item)">
            <i class="el-icon-edit"/>
          </v-btn>
          <v-btn icon @click="deleteBrand(props.item)">
            <i class="el-icon-delete"/>
          </v-btn>
        </td>
      </template>
    </v-data-table>
    <!--弹出新增对话框-->
    <v-dialog max-width="500" v-model="addShow" persistent scrollable>
      <v-card>
        <!--对话框的标题-->
        <v-toolbar dense dark color="primary">
          <v-toolbar-title>新增跑步机</v-toolbar-title>
          <v-spacer/>
          <!--关闭窗口的按钮-->
          <v-btn icon @click="closeWindow"><v-icon>close</v-icon></v-btn>
        </v-toolbar>
        <!--对话框的内容，表单-->
        <v-card-text class="px-5" style="height:400px">
          <RunningMachine-AddForm @close="closeWindow" :oldrunningMachine="oldrunningMachine" />
        </v-card-text>
      </v-card>
    </v-dialog>

    <!--弹出修改对话框-->
    <v-dialog max-width="500" v-model="editShow" persistent scrollable>
      <v-card>
        <!--对话框的标题-->
        <v-toolbar dense dark color="primary">
          <v-toolbar-title>修改跑步机</v-toolbar-title><!--{{isEdit ? '修改' : '新增'}}品牌-->
          <v-spacer/>
          <!--关闭窗口的按钮-->
          <v-btn icon @click="closeWindow"><v-icon>close</v-icon></v-btn>
        </v-toolbar>
        <!--对话框的内容，表单-->
        <v-card-text class="px-5" style="height:400px">
          <RunningMachine-EditForm @close="closeWindow" :oldrunningMachine="oldrunningMachine" />  <!--:isEdit="isEdit"-->
        </v-card-text>
      </v-card>
    </v-dialog>
  </v-card>
</template>

<script>
  // 导入自定义的表单组件
  import RunningMachineAddForm from './RunningMachineAddForm'
  import RunningMachineEditForm from './RunningMachineEditForm'
  export default {
    name: "runningMachine",
    data() {
      return {
        search: '', // 搜索过滤字段
        totalCategories: 0, // 总条数
        categories: [], // 当前页品牌数据
        loading: true, // 是否在加载中
        pagination: {}, // 分页信息
        headers: [
          {text: '跑步机id（可排序,单位为时间毫秒秘钥）', align: 'center', value: 'id'},
          {text: '所属分类', align: 'center',  value: 'cid'},
          {text: '当前版本号', align: 'center',  value: 'currentVersionCode',sortable: false},
          {text: '客户级别', align: 'center', value: 'customer_level'},
          {text: '运行里程（公里）', align: 'center', value: 'runtimeMileage'},
          {text: '国家', align: 'center', value: 'country'},
          {text: '省份', align: 'center',  value: 'province'},
          {text: '地址', align: 'center',  value: 'address'},
          {text: '操作', align: 'center', value: 'id', sortable: false}
        ],
        addShow: false,// 控制对话框的显示
        editShow:false,
        oldrunningMachine: {}, // 即将被编辑的品牌数据
        isEdit: false, // 是否是编辑
        flag:1
      }
    },
    mounted() { // 渲染后执行
      // 查询数据
      this.getDataFromServer();
    },
    watch: {
      pagination: { // 监视pagination属性的变化
        deep: true, // deep为true，会监视pagination的属性及属性中的对象属性变化
        handler() {
          // 变化后的回调函数，这里我们再次调用getDataFromServer即可
          this.getDataFromServer();
        }
      },
      search: { // 监视搜索字段
        handler() {
          this.getDataFromServer();
        }
      }
    },
    methods: {
      getDataFromServer() { // 从服务的加载数的方法。
        // 发起请求
        this.$http.get("/product/runningmachine/page", {
          params: {
            key: this.search, // 搜索条件
            page: this.pagination.page,// 当前页
            rows: this.pagination.rowsPerPage,// 每页大小
            sortBy: this.pagination.sortBy,// 排序字段
            desc: this.pagination.descending// 是否降序
          }
        }).then(resp => { // 这里使用箭头函数
          this.categories = resp.data.items;
          this.totalCategories = resp.data.total;
          // 完成赋值后，把加载状态赋值为false
          this.loading = false;
        })
      },

      addrunningMachine() {
        // 修改标记
        //this.isEdit = false;
        // 控制弹窗可见：
        this.addShow = true;
        // 把oldBrand变为null
        this.oldrunningMachine = null;
      },

      editBrand(oldrunningMachine){
        // 根据品牌信息查询商品分类
        this.$http.get("/product/runningmachine/?id=" + oldrunningMachine.id)
          .then(({data}) => {
            // 修改标记
            //this.isEdit = true;
            // 控制弹窗可见：
            this.editShow = true;
            // 获取要编辑的runningMachine
            this.oldrunningMachine = oldrunningMachine;
          })
      },

      deleteBrand(oldrunningMachine){
        // 根据品牌信息查询商品分类
        this.$http.delete("/product/runningmachine/?id=" + oldrunningMachine.id)
          .then(() => {
            this.$message.success("删除成功！");
            this.getDataFromServer();
          })
      },
      closeWindow(){
        // 重新加载数据
        this.getDataFromServer();
        // 关闭窗口
        this.addShow = false;
        this.editShow = false;
      }
    },
    components:{
      RunningMachineAddForm,
      RunningMachineEditForm
    }
  }
</script>

<style scoped>

</style>
