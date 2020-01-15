<template>
  <div class="basic">
    <el-row>
      <el-col :span="24">
        <el-card>
          <div slot="header" class="header">
              <div>
                <span>商品信息</span>
                <el-radio-group class="radio-group" v-model="statusType">
                  <el-radio :label="'4'">全部</el-radio>
                  <el-radio :label="'1'">已通过</el-radio>
                  <el-radio :label="'2'">待审核</el-radio>
                  <el-radio :label="'3'">未通过</el-radio>
                </el-radio-group>
                <span class="sort">
                  商品分类<el-cascader class="casa" v-model="value" :options="options" @change="handleChange"></el-cascader>
                </span>
              </div>
          </div>
          <div class="table-wrapper">
            <el-table v-loading="loading" element-loading-text="加载数据中" :data="tableData" border>
              <el-table-column label="商品名称" prop="name" align="center"></el-table-column>
              <el-table-column label="所属农场" prop="farm" align="center"></el-table-column>
              <el-table-column label="产品规格" prop="specs" align="center"></el-table-column>
              <el-table-column label="应季期(月)" prop="seasinal" align="center"></el-table-column>
              <el-table-column
                label="商品上市日期"
                min-width="120px"
                prop="addedDate"
                align="center"
                :formatter="getAddedDate"
              ></el-table-column>
              <el-table-column
                label="商品下市日期"
                min-width="120px"
                prop="offDate"
                align="center"
                :formatter="getOffDate"
              ></el-table-column>
              <el-table-column label="商户售价(元)" prop="merchantPrice" align="center"></el-table-column>
              <el-table-column label="平台售价(元)" prop="platformPrice" align="center"></el-table-column>
              <el-table-column label="配送范围" prop="areas" align="center">
                <template slot-scope="scope">
                  <div class="nowrap" v-for="(item, index) in scope.row.areas" :key="index">{{item.province + ',' + item.city + ',' + item.district}}</div>
                </template>
              </el-table-column>
              <el-table-column label="操作" align="center">
                <template slot-scope="scope">
                  <el-button v-if="scope.row.status == '2'" size="mini" type="primary" @click="showDetail(scope.row)">审核</el-button>
                </template>
              </el-table-column>
            </el-table>
            <el-pagination
              style="margin-top: 16px; text-align:right;"
              layout="total, sizes, prev, pager, next, jumper"
              :page-sizes="[5, 10, 15, 20]"
              :total="total"
              @size-change="handleSizeChange"
              @current-change="handleCurrentChange"
            ></el-pagination>
          </div>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>
<script>
import all from "../../data/city.json";
import sorts from "../../data/productSort.json";
export default {
  components: {},
  data() {
    return {
      loading: false,
      tableData: [],
      total: 0,
      value: ["1"],
      options: sorts,
      pagesize: 10,
      currentpage: 1,
      goodsType: "1",
      statusType: "4",
      status: "1,2,3"
    };
  },
  mounted() {
    this.getTableData("1");
  },
  watch: {
    statusType(val) {
      this.status = this.statusType == "4" ? "1,2,3" : this.statusType;
      this.getTableData();
    }
  },
  computed: {},
  methods: {
    getTableData() {
      this.axios
        .get(
          "/goodsExamList?goodsType=" +
            this.goodsType +
            "&&pageNum=" +
            this.currentpage +
            "&&pageSize=" +
            this.pagesize +
            "&&status=" +
            this.status
        )
        .then(res => {
          if (res.code == "1") {
            this.tableData = res.list.rows;
            this.total = res.list.total;
          }
        })
        .catch();
    },
    //上架日期格式化
    getAddedDate(row, column) {
      return row.addedDate.split(" ")[0];
    },
    //下架日期格式化
    getOffDate(row, column) {
      return row.offDate.split(" ")[0];
    },
    //查看详情
    showDetail(detail) {
      localStorage.setItem('detail',JSON.stringify(detail));
      this.$router.push({path: 'goods-detail', query: {id: detail.id}});
    },
    //商品种类
    handleChange(value) {
      this.goodsType = value.join(",");
      this.getTableData();
    },
    handleSizeChange(value) {
      this.pagesize = value;
      this.getTableData();
    },
    handleCurrentChange(value) {
      this.currentpage = value;
      this.getTableData();
    }
  }
};
</script>
<style lang='scss' scoped>
@import "../../assets/styles/mixin";
.basic {
  .header {
    min-width: 900px;
    @include fx(flex-start);
    .add-btn {
      margin-left: 10px;
    }
    .sort {
      margin-left: 40px;
    }
    .radio-group,
    .casa {
      margin-left: 20px;
    }
  }
  .el-table {
    .warning-row {
      background-color: oldlace;
    }
    .table-expand {
      font-size: 0;
      margin-left: 200px;
    }
    .table-expand label {
      width: 70px;
      color: $label;
    }
    .table-expand .el-form-item {
      margin-right: 0;
      margin-bottom: 0;
      display: flex;
    }
    .nowrap {
      white-space: nowrap;
    }
  }
  .recommend-tag {
    display: inline-block;
    margin: 4px 0;
    margin-right: 4px;
    &:last-child {
      margin-right: 0;
    }
  }
  .checkbox-group {
    margin-left: 20px;
  }
}
</style>
