<template>
  <div class="app-container">
    <div
      class="filter-container"
      style="display: flex; justify-content: space-between"
    >
      <el-button
        class="filter-item"
        style="margin-left: 10px"
        type="primary"
        icon="el-icon-edit"
        @click="handleCreate"
      >
        创建优惠券
      </el-button>
    </div>
    <el-table
      :key="tableKey"
      v-loading="listLoading"
      :data="list"
      border
      fit
      highlight-current-row
      style="width: 100%"
    >
      <el-table-column label="优惠券ID" width="80px" align="center">
        <template slot-scope="{ row }">
          {{ row.id }}
        </template>
      </el-table-column>
      <el-table-column label="面值" width="100" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.price }}</span>
        </template>
      </el-table-column>
      <el-table-column label="适用课程" min-width="120">
        <template slot-scope="{ row }">
          <span>{{ row.value.title }}</span>
        </template>
      </el-table-column>
      <el-table-column label="使用期限" width="170" align="center">
        <template slot-scope="{ row }">
          {{ row.created_time + "至" + row.end_time }}
        </template>
      </el-table-column>
      <el-table-column label="发行量" width="100" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.c_num }}</span>
        </template>
      </el-table-column>
      <el-table-column label="已领取" width="100" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.received_num }}</span>
        </template>
      </el-table-column>
      <el-table-column label="已使用" width="100" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.used_num }}</span>
        </template>
      </el-table-column>
      <el-table-column label="状态" width="100" align="center">
        <template slot-scope="{ row }">
          <span
            :style="
              row.time_status == '领取中' ? 'color:red;' : 'color:#bbbbbb;'
            "
            >{{ row.time_status }}</span
          >
        </template>
      </el-table-column>
      <el-table-column
        label="操作"
        align="center"
        width="230"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="{ row, $index }">
          <el-button
            type="primary"
            size="mini"
            @click="handleUpdate(row, $index)"
          >
            编辑
          </el-button>
          <el-button
            size="mini"
            type="danger"
            :disabled="row.status == 0"
            @click="changeStatus(row, $index)"
            >下架</el-button
          >
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import { fetchCoupon, createCoupon, updateCoupon } from "@/api/marketing";
import chooseCourse from "@/components/ChooseCourse/index.vue";
import waves from "@/directive/waves"; // waves directive
import util from "@/utils/util.js";
import Pagination from "@/components/Pagination"; // secondary package based on el-pagination
let timeStatusOptions = {
  0: "未开始",
  1: "领取中",
  2: "已结束",
  3: "已下架",
};
export default {
  data() {
    return {
      tableKey: 0,
      list: null,
      total: 0,
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 20,
      },
      dialogFormVisible: false,
    };
  },
  created() {
    this.getlist();
  },
  methods: {
    getlist() {
      this.listLoading = true;
      fetchCoupon().then((res) => {
        // console.log(res.data.items);
        if (res.code === 20000) {
          this.list = res.data.items.map((item) => {
            let k = util.formatGroupStatus(item);
            item.time_status = timeStatusOptions[k];
            return item;
          });
          this.total = res.data.total;
          this.listLoading = false;
          console.log(this.list);
        }
      });
    },
    handleCreate() {},
    handleUpdate() {},
    changeStatus(row) {
      this.$confirm("是否要下架该活动？", "提示", {
        confirmButtonText: "下架",
        cancelButtonText: "取消",
        type: "warning",
      }).then((action) => {
        row.status = 0;
        row.time_status = "已下架";
        this.$message({
          message: "下架成功",
          type: "success",
        });
      });
    },
  },
};
</script>

<style scoped>
</style>