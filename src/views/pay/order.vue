<template>
  <div class="app-container">
    <div
      class="filter-container"
      style="display: flex; justify-content: space-between"
    >
      <el-button
        v-waves
        class="filter-item"
        type="primary"
        icon="skill"
        @click="handleDownload"
      >
        导出选中
      </el-button>
      <div>
        <el-input
          v-model="listQuery.titel"
          placeholder="请输入商品名称"
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
      <el-table-column label="订单号" width="100" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.id }}</span>
        </template>
      </el-table-column>
      <el-table-column label="商品名称" min-width="150px">
        <template slot-scope="{ row }">
          <span>{{ row.goods[0].title }}</span>
        </template>
      </el-table-column>
      <el-table-column label="订单类型" width="100" align="center">
        <template slot-scope="{ row }">
          {{ row.type | typeFilter }}
        </template>
      </el-table-column>
      <el-table-column label="订单状态" width="100" align="center">
        <template slot-scope="{ row }">
          <el-tag :type="row.status == 'success' ? 'success' : 'danger'">
            {{ row.status | statusFilter }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column label="原价/实付(元)" width="150" align="center">
        <template slot-scope="{ row }">
          <span style="color: #bbbbbb">{{ row.total_price }}</span>
          <span>/</span>
          <span style="color: red">{{ row.price }}</span>
        </template>
      </el-table-column>
      <el-table-column label="支付方式" width="80" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.pay_method | methodFilter }}</span>
        </template>
      </el-table-column>
      <el-table-column label="创建/支付时间" width="180px" align="center">
        <template slot-scope="{ row }">
          <p>{{ row.created_time }}</p>
          <p>{{ row.pay_time }}</p>
        </template>
      </el-table-column>
      <el-table-column
        label="操作"
        width="280px"
        class-name="small-padding  fixed-width"
        align="center"
      >
        <template slot-scope="{ row, $index }">
          <el-popconfirm
            title="是否要删除该记录？"
            @onConfirm="handleDelete(row, $index)"
            style="margin-left: 10px"
          >
            <el-button
              slot="reference"
              size="mini"
              type="danger"
              style="margin-right: 10px"
              >删除</el-button
            >
          </el-popconfirm>
        </template>
      </el-table-column>
    </el-table>
    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="listQuery.page"
      :limit.sync="listQuery.limit"
      @pagination="getlist"
    />
  </div>
</template>

<script>
import { fetchOrder, deleteOrder } from "@/api/pay";

import waves from "@/directive/waves"; // waves directive
import Tinymce from "@/components/Tinymce";
import { parseTime } from "@/utils";
import Pagination from "@/components/Pagination"; // secondary package based on el-pagination
let statusOptions = {
  pendding: "待支付",
  success: "成功",
  fail: "失败",
};
let typeOptions = {
  group: "拼团",
  default: "普通",
};
let methodOptions = {
  wechat: "微信支付",
};
export default {
  name: "ComplexTable",
  components: {
    Pagination,
    Tinymce,
  },
  filters: {
    statusFilter(val) {
      return statusOptions[val];
    },
    typeFilter(val) {
      return typeOptions[val];
    },
    methodFilter(val) {
      return methodOptions[val];
    },
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
        titel: "",
      },
      multipleSelection: [],
      downloadLoading: false,
      filename: "",
    };
  },
  directives: {
    waves,
  },
  created() {
    this.getlist();
  },
  methods: {
    getlist(v) {
      this.listLoading = true;
    //   console.log(v);
      fetchOrder(v).then((res) => {
        // console.log(res.data.items);
        this.list = res.data.items;
        this.total = res.data.total;
        this.listLoading = false;
      });
    },
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    handleDownload() {
      this.downloadLoading = true;
      import("@/vendor/Export2Excel").then((excel) => {
        const tHeader = [
          "订单号",
          "商品名称",
          "订单类型",
          "订单状态",
          "原价（元）",
          "实付（元）",
          "支付方式",
          "创建时间",
          "支付时间",
        ];
        const filterVal = [
          "no",
          "goods",
          "type",
          "status",
          "total_price",
          "price",
          "pay_method",
          "created_time",
          "pay_time",
        ];
        const list = this.multipleSelection.map((item) => {
          let o = { ...item };
          o.goods = item.goods.map((v) => v.title).join(",");
          o.type = typeOptions[item.type];
          o.status = statusOptions[item.status];
          o.pay_method = methodOptions[item.pay_method];
          return o;
        });
        const data = this.formatJson(filterVal, list);
        excel.export_json_to_excel({
          header: tHeader,
          data,
        });
        this.downloadLoading = false;
      });
    },
    formatJson(filterVal, jsonData) {
      return jsonData.map((v) =>
        filterVal.map((j) => {
          if (j === "timestamp") {
            return parseTime(v[j]);
          } else {
            return v[j];
          }
        })
      );
    },
    handleFilter() {
      this.listQuery.page = 1;
      this.getlist(this.listQuery);
    },
    handleDelete(row, index) {
      console.log(row, index);
      deleteOrder(index).then((res) => {
        console.log(res);
        if (res.code === 20000) {
          this.$notify({
            title: "提示",
            message: "删除成功",
            type: "success",
            duration: 2000,
          });
          this.list.splice(index, 1);
        }
      });
    },
  },
};
</script>

<style scoped>
</style>