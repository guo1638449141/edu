<template>
  <div class="app-container">
    <div>
      <el-button type="primary" @click="addPay"
        ><i class="el-icon-plus" />新增收款账号</el-button
      >
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
      <el-table-column label="账号" min-width="180">
        <template slot-scope="{ row }">
          <span>{{ row.account }}</span>
        </template>
      </el-table-column>
      <el-table-column label="开户人" width="150" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.name }}</span>
        </template>
      </el-table-column>
      <el-table-column label="开户行" width="200" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.path }}</span>
        </template>
      </el-table-column>
      <el-table-column label="操作" width="200" align="center">
        <template slot-scope="{ row, $index }">
          <el-button type="primary" @click="editPay(row, $index)"
            >编辑</el-button
          >
          <el-button type="danger" @click="delPay(row, $index)">删除</el-button>
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
    <el-dialog
      :title="isEdit ? '修改' : '添加'"
      :visible.sync="dialogVisible"
      width="50%"
    >
      <el-form
        ref="dataForm"
        :rules="rules"
        :model="temp"
        label-position="left"
        label-width="70px"
        style="width: 400px; margin-left: 25px"
      >
        <el-form-item label="账号" prop="account" label-width="100px">
          <el-input v-model="temp.account" />
        </el-form-item>
        <el-form-item label="省市区" label-width="100px">
          <v-distpicker
            @province="onChangeProvince"
            @city="onChangeCity"
            @area="onChangeArea"
          ></v-distpicker>
        </el-form-item>
        <el-form-item label="详细地址" prop="path" label-width="100px">
          <el-input v-model="temp.path" />
        </el-form-item>
        <el-form-item label="所属银行" prop="bank" label-width="100px">
          <el-select v-model="temp.bank" placeholder="请选择">
            <el-option
              v-for="item in options"
              :key="item.value"
              :label="item.label"
              :value="item.label"
            >
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="姓名" prop="name" label-width="100px">
          <el-input v-model="temp.name" />
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="closeAdd">取 消</el-button>
        <el-button type="primary" @click="isEdit ? submitEdit() : submitAdd()"
          >确 定</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
import VDistpicker from "v-distpicker";
import {
  fetchPayments,
  createPayment,
  updatePayment,
  deletePayment,
} from "@/api/pay";
import Pagination from "@/components/Pagination"; // secondary package based on el-pagination
import { dateFtt } from "../../utils/data";
export default {
  components: { VDistpicker, Pagination },
  data() {
    return {
      dialogVisible: false,
      tableKey: 0,
      list: [],
      total: 0,
      isEdit: false,
      listLoading: false,
      listQuery: {
        page: 1,
        limit: 20,
        titel: "",
      },
      temp: {
        account: "",
        area: "",
        bank: "",
        city: "",
        created_time: "",
        id: "",
        name: "",
        path: "",
        province: "",
        status: 1,
      },
      options: [
        {
          value: "选项一",
          label: "国家开发银行",
        },
        {
          value: "选项二",
          label: "中国银行",
        },
        {
          value: "选项三",
          label: "中国建设银行",
        },
        {
          value: "选项四",
          label: "中国工商银行",
        },
      ],
      rules: {
        account: [{ required: true, message: "标题为必填项", trigger: "blur" }],
        path: [{ required: true, message: "标题为必填项", trigger: "blur" }],
        bank: [{ required: true, message: "标题为必填项", trigger: "change" }],
        name: [{ required: true, message: "标题为必填项", trigger: "blur" }],
      },
    };
  },
  created() {
    this.getlist();
  },
  methods: {
    getlist() {
      fetchPayments().then((res) => {
        console.log(res.data.items);
        this.list = res.data.items;
        this.total = res.data.total;
      });
    },
    onChangeProvince(e) {
      //   console.log(e.value);
      this.temp.province = e.value;
    },
    onChangeCity(e) {
      //   console.log(e);
      this.temp.city = e.value;
    },
    onChangeArea(e) {
      //   console.log(e);
      this.temp.area = e.value;
    },
    addPay() {
      this.dialogVisible = true;
    },
    closeAdd() {
      this.dialogVisible = false;
    },
    submitAdd() {
      this.temp.id = parseInt(Math.random() * 100) + 1024; // mock a id
      let time = new Date();
      time = dateFtt("yyyy-MM-dd hh:mm:ss", time);
      this.temp.created_time = time;
      console.log(this.temp);
      //   this.$refs.dataForm.valid
      createPayment(this.temp).then((res) => {
        console.log(res);
        if (res.code === 20000) {
          this.$notify({
            title: "提示",
            message: "添加成功",
            type: "success",
            duration: 2000,
          });
          this.list.unshift(this.temp);
          this.dialogVisible = false;
        }
      });
    },
    // 编辑
    submitEdit() {
      // console.log(this.temp);
      updatePayment(this.temp).then((res) => {
        // console.log(res);
        if (res.code === 20000) {
          this.$notify({
            title: "提示",
            message: "修改成功",
            type: "success",
            duration: 2000,
          });
          this.dialogVisible = false;
        }
      });
    },
    editPay(row, index) {
      this.dialogVisible = true;
      this.isEdit = true;
      console.log(row);
      this.temp = row;
      // this.temp.province = row.province
    },
    delPay(row, index) {
      deletePayment({ id: row.id }).then((res) => {
        // console.log(res);
        if (res.code === 20000) {
          this.$notify({
            title: "提示",
            message: "删除成功",
            type: "success",
            duration: 2000,
          });
          this.dialogVisible = false;
          this.list.splice(index,1)
        }
      });
    },
  },
};
</script>

<style scoped>
</style>