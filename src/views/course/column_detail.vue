<template>
  <div class="app-container">
    <el-card shadow="never">
      <div style="display: flex; align-items: flex-start">
        <img
          :src="detail.cover"
          alt=""
          style="width: 200px; height: 100px; margin-right: 20px"
        />
        <div style="flex: 1">
          <div style="display: flex; justify-content: space-between">
            <h4 style="margin-top: 5px; margin-bottom: 0">
              {{ detail.title }}
            </h4>
            <small style="color: #bbbbbb">{{
              detail.isend ? "已完结" : "连载中"
            }}</small>
          </div>
          <p style="margin: 8px 0">
            <small style="color: #bbbbbb">{{ detail.try }}</small>
          </p>
          <p>
            <small style="color: red">￥{{ detail.price }}</small>
          </p>
          <el-button-group>
            <el-button size="small" type="warning" @click="changeStatus">
              {{ detail.status ? "下架" : "上架" }}
            </el-button>
            <el-button size="small" type="default" @click="changeIsend">
              设为{{ detail.isend ? "连载中" : "已完结" }}
            </el-button>
          </el-button-group>
        </div>
      </div>
    </el-card>
    <div
      class="filter-container"
      style="display: flex; justify-content: space-between; margin-top: 20px"
    >
      <el-button
        class="filter-item"
        style="margin-left: 10px"
        type="primary"
        icon="el-icon-edit"
        @click="addCourse"
      >
        新增目录
      </el-button>
      <div>
        <el-select
          v-model="listQuery.status"
          placeholder="商品状态"
          clearable
          style="width: 120px; margin-right: 10px"
          class="filter-item"
        >
          <el-option
            v-for="(item, k) in statusOptions"
            :key="k"
            :label="item"
            :value="k"
          />
        </el-select>
        <el-input
          v-model="listQuery.title"
          placeholder="标题"
          style="width: 200px"
          class="filter-item"
          @keyup.enter.native="handleFilter"
        />
        <el-button
          v-waves
          class="filter-item"
          type="primary"
          icon="el-icon-search"
          @click="handleFilter"
        >
          搜索
        </el-button>
      </div>
    </div>
    <el-table
      ref="dragTable"
      v-loading="listLoading"
      :data="list"
      row-key="id"
      border
      fit
      highlight-current-row
      style="width: 100%"
    >
      <el-table-column type="index" width="50"> </el-table-column>

      <el-table-column label="单品内容" min-width="180px">
        <template slot-scope="{ row }">
          <div style="display: flex">
            <img
              :src="row.cover"
              style="width: 100px; height: 50px; margin-right: 10px"
            />
            <div
              style="
                display: flex;
                flex-direction: column;
                justify-content: space-between;
              "
            >
              <span>{{ row.title }}</span>
              <span style="color: red">￥{{ row.price }}</span>
            </div>
          </div>
        </template>
      </el-table-column>
      <el-table-column label="订阅量" width="110px" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.sub_count }}</span>
        </template>
      </el-table-column>

      <el-table-column label="状态" class-name="status-col" width="100">
        <template slot-scope="{ row }">
          <el-tag :type="row.status ? 'success' : 'danger'">
            {{ row.status | statusFilter }}
          </el-tag>
        </template>
      </el-table-column>

      <el-table-column
        label="操作"
        align="center"
        width="230"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="{ row, $index }">
          <el-button type="primary" size="mini" @click="handleUpdate(row)">
            编辑
          </el-button>
          <el-popconfirm
            title="是否要删除该记录？"
            @onConfirm="handleDelete(row, $index)"
            style="margin-left: 10px"
          >
            <el-button
              v-if="row.status != 'deleted'"
              size="mini"
              type="danger"
              slot="reference"
              >删除</el-button
            >
          </el-popconfirm>
        </template>
      </el-table-column>

      <el-table-column align="center" label="Drag" width="80">
        <template slot-scope="{}">
          <svg-icon class="drag-handler" icon-class="drag" />
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import { fetchDetail, fetchDetailCourse } from "@/api/column.js";
const statusOptions = {
  0: "已下架",
  1: "已上架",
};
export default {
  data() {
    return {
      list: [],
      listLoading: true,
      detail: {
        content: "",
        cover: "",
        created_time: "",
        id: 0,
        isend: 0,
        price: 0,
        status: 1,
        sub_count: 0,
        title: "",
        try: "",
        updated_time: "",
      },
      listQuery: {
        status: undefined,
        title: undefined,
      },
      statusOptions,
    };
  },
  created() {
    // console.log(this.$router.history.current.query.id);
    this.getData();
    this.getList();
  },
  methods: {
    // 获取当前信息
    getData() {
      fetchDetail({ id: this.$router.history.current.query.id }).then((res) => {
        // console.log(res.data);
        this.detail = res.data;
        // console.log(this.detail);
      });
    },
    // 获取目录
    async getList() {
      this.listLoading = true;
      let res = await fetchDetailCourse(this.listQuery);
      // console.log(res);
      this.list = res.data.items;
      console.log(this.list);
      this.listLoading = false;
    },
    // 改变上架状态
    changeStatus() {
      this.detail.status = this.detail.status ? 0 : 1;
    },
    // 改变是否已完结
    changeIsend() {
      this.detail.isend = this.detail.isend ? 0 : 1;
    },
    // 新增目录
    addCourse() {
        
    },
    // 搜索
    handleFilter() {
      this.getList();
      setTimeout(() => {
        if (this.listQuery.status == 0) {
          this.list = this.list.filter((item) => item.status === 0);
        } else if (this.listQuery.status == 1) {
          this.list = this.list.filter((item) => item.status === 1);
        }
        if(this.listQuery.title){
            console.log(this.listQuery.title);
            this.list = this.list.filter(item => item.title.indexOf(this.listQuery.title) !== -1)
            console.log(this.list);
        }else{
            this.getList()
        }
      }, 1000);

    },
    // 编辑
    handleUpdate() {},
    // 删除
    handleDelete(row, index) {
      this.$notify({
        title: "提示",
        message: "删除成功",
        type: "success",
        duration: 2000,
      });
      this.list.splice(index, 1);
    },
  },
  filters: {
    statusFilter(status) {
      return statusOptions[status];
    },
  },
};
</script>

<style scoped>
</style>