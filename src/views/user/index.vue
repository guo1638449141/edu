<template>
  <div class="app-container">
    <div
      class="filter-container"
      style="display: flex; justify-content: space-between"
    >
      <el-dropdown>
        <el-button type="danger" size="small">
          黑名单设置<i class="el-icon-arrow-down el-icon--right"></i>
        </el-button>
        <el-dropdown-menu slot="dropdown">
          <el-dropdown-item @click.native="bancomment(true)"
            >禁止评论</el-dropdown-item
          >
          <el-dropdown-item @click.native="banvisit(true)"
            >禁止访问</el-dropdown-item
          >
        </el-dropdown-menu>
      </el-dropdown>
      <div>
        <el-input
          v-model="listQuery.titel"
          placeholder="请输入用户名"
          style="width: 200px"
          class="filter-item"
          @keyup.enter.native="handleFilter"
        ></el-input>
        <el-button
          v-waves
          class="filter-item"
          type="primary"
          icon="el-icon-search"
          @click="handleFilter"
          >搜索</el-button
        >
      </div>
    </div>
    <el-table
      :key="tableKey"
      v-loading="listLoading"
      :data="list"
      border
      fit
      highlight-current-row
      style="width: 100%"
      ref="multipleTable"
      @selection-change="handleSelectionChange"
    >
      <el-table-column
        type="selection"
        width="55"
        align="center"
      ></el-table-column>
      <el-table-column label="用户" min-width="180px">
        <template slot-scope="{ row }">
          <div style="display: flex; align-items: center">
            <img
              :src="row.user.avatar"
              style="
                width: 50px;
                heigth: 50px;
                margin-right: 10px;
                border-radius: 100%;
              "
              alt=""
            />
            <span>{{ row.user.username }}</span>
          </div>
        </template>
      </el-table-column>
      <el-table-column label="消费金额" width="110px" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.total_consume }}</span>
        </template>
      </el-table-column>
      <el-table-column label="创建时间" width="180px" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.created_time }}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="操作"
        width="280px"
        class-name="small-padding fixed-width"
        align="center"
      >
        <template slot-scope="{ row }">
          <el-button type="primary" size="mini" @click="openInfo(row.user.id)"
            >详情</el-button
          >
          <el-button type="success" size="mini" style="margin-right: 10px"
            >联系用户</el-button
          >
          <el-dropdown>
            <el-button type="danger" size="mini">
              黑名单设置<i class="el-icon-arrow-down el-icon--right"></i>
            </el-button>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item @click.native="bancomment(row.user.id)"
                >禁止评论</el-dropdown-item
              >
              <el-dropdown-item @click.native="banvisit(row.user.id)"
                >禁止访问</el-dropdown-item
              >
            </el-dropdown-menu>
          </el-dropdown>
        </template>
      </el-table-column>
    </el-table>
    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="listQuery.page"
      :limit.sync="listQuery.limit"
      @pagination="getList"
    />
    <info ref="info"></info>
  </div>
</template>
<script>
import { fetchList, changeCommentStatus, changeAccessStatus } from "@/api/user";
import waves from "@/directive/waves"; // waves directive
import { parseTime } from "@/utils";
import Pagination from "@/components/Pagination"; // 分页组件
import Tinymce from "@/components/Tinymce"; // 富文本编辑器
import info from "./info";
export default {
  name: "ComplexTable",
  components: {
    Pagination,
    Tinymce,
    info,
  },
  directives: {
    waves,
  },
  data() {
    return {
      tableKey: 0,
      list: [],
      total: 0,
      listLoading: false,
      listQuery: {
        page: 1,
        limit: 20,
        title: "",
      },
      multipleSelection: [],
    };
  },
  created() {
    this.getList();
  },
  methods: {
    async getList(listQuery) {
      this.listLoading = true;
      let res = await fetchList(this.listQuery);
      //   console.log(res);
      if (res.code === 20000) {
        this.list = res.data.items;
        this.total = res.data.total;
        console.log(this.list);
        this.listLoading = false;
      }
    },
    // 打开详情页
    openInfo(id) {
      console.log(id);
      this.$refs.info.open(id);
    },
    // 禁止评论
    bancomment(ids) {
      if (typeof ids == "boolean" && !this.multipleSelection.length) {
        return this.$message({
          message: "请先选中需要操作的用户",
          type: "error",
        });
      }
      let id =
        typeof ids == "boolean"
          ? this.multipleSelection.map((item) => item.id)
          : ids;
      // console.log(id);
      let allText = typeof ids == "boolean" ? "选中" : "当前";
      // console.log(allText);
      this.$confirm("是否要禁止" + allText + "用户评论？", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }).then((action) => {
        this.listLoading = true;
        changeCommentStatus({
          id,
          status: 0,
        }).then((res) => {
          console.log(res);
          if (res.code === 20000) {
            this.$message.success("操作成功");
            this.$refs.multipleTable.clearSelection();
            this.listLoading = false;
          }
        });
      });
    },
    // 禁止访问
    banvisit(ids) {
      if (typeof ids == "boolean" && !this.multipleSelection.length) {
        return this.$message({
          message: "请先选中需要操作的用户",
          type: "error",
        });
      }
      let id =
        typeof ids == "boolean"
          ? this.multipleSelection.map((item) => item.id)
          : ids;
      // console.log(id);
      let allText = typeof ids == "boolean" ? "选中" : "当前";
      // console.log(allText);
      this.$confirm("是否要禁止" + allText + "用户评论？", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }).then((action) => {
        this.listLoading = true;
        changeAccessStatus({
          id,
          status: 0,
        }).then((res) => {
          console.log(res);
          if (res.code === 20000) {
            this.$message.success("操作成功");
            this.$refs.multipleTable.clearSelection();
            this.listLoading = false;
          }
        });
      });
    },
    // 搜索
    handleFilter() {
      this.listQuery.page = 1;
      this.getList(this.listQuery);
    },
    // 表格改变
    handleSelectionChange(val) {
      this.multipleSelection = val;
      // console.log(this.multipleSelection);
    },
  },
};
</script>

<style scoped>
/* .filter-container{
    display: flex; 
    justify-content: space-between;
    align-items: center;
} */
</style>
