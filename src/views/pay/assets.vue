<template>
  <div class="app-container">
    <div
      class="app-header"
      style="display: flex; justify-content: space-between"
    >
      <div>
        <p  style="display: flex; justify-content: space-between;align-items: center;">
          <span>可用余额(元)</span><el-button type="primary">申请提现</el-button>
        </p>
        <hr style="width: 100%" />
        <h3>20.00</h3>
      </div>
      <div>
        <p><span>累计收入(元)</span></p>
        <hr style="width: 100%；color:gainsboro" />
        <h3>20.00</h3>
      </div>
      <div>
        <p><span>待结算金额(元)</span></p>
        <hr style="width: 100%" />
        <h3>21</h3>
      </div>
      <div>
        <p><span>冻结金额(元)</span></p>
        <hr style="width: 100%" />
        <h3>20.00</h3>
      </div>
    </div>
    <el-table
      :key="tableKey"
      v-loading="listLoading"
      :data="list"
      border
      fit
      highlight-current-row
      style="width: 100%; margin-top: 20px"
      ref="multipleTable"
    >
      <el-table-column label="交易时间" width="180" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.created_time }}</span>
        </template>
      </el-table-column>
      <el-table-column label="提款账号" min-width="150" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.account }}</span>
        </template>
      </el-table-column>
      <el-table-column label="开户人" width="100" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.name }}</span>
        </template>
      </el-table-column>
      <el-table-column label="提现金额" width="100" align="center">
        <template slot-scope="{ row }">
          <span>￥{{ row.price }}</span>
        </template>
      </el-table-column>
      <el-table-column label="状态" width="100" align="center">
        <template slot-scope="{ row }">
          <el-tag :type="row.status == 1 ? 'success' : 'danger'">
            <span>{{ row.status ? "提现成功" : "提现失败" }}</span>
          </el-tag>
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
import { fetchAssets } from "@/api/pay";
import Pagination from "@/components/Pagination"; // secondary package based on el-pagination
export default {
  components: {
    Pagination,
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
    };
  },
  created() {
    this.getlist();
  },
  methods: {
    getlist() {
      fetchAssets().then((res) => {
        console.log(res.data.items);
        this.list = res.data.items;
        this.total = res.data.total;
      });
    },
  },
};
</script>

<style scoped>
.app-header{
    display: flex;
    justify-content: space-between;
    align-items: center;
}
.app-header div{
    width:20%;
    border: 1px solid gainsboro;
    border-radius: 5px;
}   
.app-header div p span{
margin-left: 20px;
}
.app-header div p button{
    margin-right: 20px;
}
.app-header div h3{
    margin-left: 20px;
}
</style>