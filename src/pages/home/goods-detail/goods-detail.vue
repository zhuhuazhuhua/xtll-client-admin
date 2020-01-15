<template>
  <div class="basic">
    <el-dialog title="已有商户详情" :visible.sync="dialogVisible" :modal-append-to-body="false">
      <div class="exists-detail">
        <div class="exists-info">
          <div class="exists-item">
            <div class="item-title">商品名称</div>
            <div class="item-cont">{{existsDetail.name}}</div>
          </div>
          <div class="exists-item">
            <div class="item-title">所属农场</div>
            <div class="item-cont">{{existsDetail.farm}}</div>
          </div>
          <div class="exists-item">
            <div class="item-title">产品规格</div>
            <div class="item-cont">{{existsDetail.specs}}</div>
          </div>
          <div class="exists-item">
            <div class="item-title">应季期</div>
            <div class="item-cont">{{existsDetail.seasinal}}</div>
          </div>
          <div class="exists-item">
            <div class="item-title">商品上市日期</div>
            <div class="item-cont">{{existsDetail.addedDate}}</div>
          </div>
          <div class="exists-item">
            <div class="item-title">商户售价</div>
            <div class="item-cont">{{'￥' + existsDetail.merchantPrice}}</div>
          </div>
          <div class="exists-item">
            <div class="item-title">商品下市日期</div>
            <div class="item-cont">{{existsDetail.offDate}}</div>
          </div>
          <div class="exists-item">
            <div class="item-title">平台售价</div>
            <div class="item-cont">{{'￥' + existsDetail.platformPrice}}</div>
          </div>
        </div>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button type="info" @click="dialogVisible = false">关闭</el-button>
      </span>
    </el-dialog>
    <el-row>
      <el-col :span="24">
        <el-card>
          <div slot="header" class="header">
            <el-page-header @back="goBack" content="商品审核"></el-page-header>
          </div>
          <div class="table-wrapper">
            <div class="goods-info">
              <div class="info-item">
                <div class="item-title">商品名称</div>
                <div class="item-cont">{{goodsDetail.name}}</div>
              </div>
              <div class="info-item">
                <div class="item-title">所属农场</div>
                <div class="item-cont">{{goodsDetail.farm}}</div>
              </div>
              <div class="info-item">
                <div class="item-title">产品规格</div>
                <div class="item-cont">{{goodsDetail.specs}}</div>
              </div>
              <div class="info-item">
                <div class="item-title">应季期</div>
                <div class="item-cont">{{goodsDetail.seasinal}}</div>
              </div>
              <div class="info-item">
                <div class="item-title">商品上市日期</div>
                <div class="item-cont">{{goodsDetail.addedDate}}</div>
              </div>
              <div class="info-item">
                <div class="item-title">商户售价</div>
                <div class="item-cont">{{'￥' + goodsDetail.merchantPrice}}</div>
              </div>
              <div class="info-item">
                <div class="item-title">商品下市日期</div>
                <div class="item-cont">{{goodsDetail.offDate}}</div>
              </div>
              <div class="info-item">
                <div class="item-title">平台售价</div>
                <div class="item-cont">{{'￥' + goodsDetail.platformPrice}}</div>
              </div>
            </div>
            <div class="compare">
              <div class="compare-item" style="width:40%">
                <div class="title">待审核配送范围</div>
                <div class="areas">
                  <div
                    class="area-item"
                    v-for="(item, index) in goodsDetail.areas"
                    :key="index"
                  >{{item.province + ', ' + item.city + ', ' + item.district}}</div>
                </div>
              </div>
              <div class="compare-item" style="width:60%">
                <div class="title">同商品配送范围已有商户</div>
                <div class="business">
                  <div class="business-item" v-for="(item, index) in businessList" :key="index">
                    <div
                      class="city"
                    >{{item.province + ', ' + item.city + ', ' + item.district + '(' + item.total + ')'}}</div>
                    <div class="table" v-if="item.total">
                      <el-table :data="item.rows" border>
                        <el-table-column label="商品名称" prop="name" align="center"></el-table-column>
                        <el-table-column label="所属农场" prop="farm" align="center"></el-table-column>
                        <el-table-column label="操作" align="center">
                          <template slot-scope="scope">
                            <el-button size="mini" type="primary" @click="showDetail(scope.row)">详情</el-button>
                          </template>
                        </el-table-column>
                      </el-table>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <el-row class="btn" justify="end">
            <el-button type="primary" @click="checkGoods('1')">通过</el-button>
            <el-button type="danger" @click="checkGoods('2')">不通过</el-button>
          </el-row>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>
<script>
export default {
  components: {},
  data() {
    return {
      goodsDetail: JSON.parse(localStorage.getItem("detail")),
      businessList: [],
      dialogVisible: false,
      existsDetail: {}
    };
  },
  created() {
    // this.getDetailInfo();
  },
  mounted() {
    this.getDetailInfo();
  },
  computed: {},
  methods: {
    goBack() {
      this.$router.go(-1);
    },
    //已有商户详情按钮
    showDetail(row) {
      console.log(row);
      this.dialogVisible = true;
      this.existsDetail = row;
    },
    //商品审核
    checkGoods(type) {
      if (type == "2") {
        this.$prompt("请输入审核不通过原因", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          inputType: "textarea",
          inputPattern: /\S/,
          inputErrorMessage: "审核不通过原因不可为空"
        })
          .then(({ value }) => {
            let data = {
              id: this.goodsDetail.id,
              reason: value
            };
            console.log(data);
            this.checkPost(data);
          })
          .catch(() => {});
      } else {
        this.$confirm("是否确定通过此商品？", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(() => {
            let data = {
              id: this.goodsDetail.id,
              reason: ""
            };
            this.checkPost(data);
          })
          .catch(() => {
            return false;
          });
      }
    },
    checkPost(data) {
      this.axios
        .post("/goodsExam", data)
        .then(res => {
          if (res.code == "1") {
            this.$message({
              message: res.msg,
              type: "success"
            });
            this.$router.go(-1);
          } else {
            this.$message({
              message: res.msg,
              type: "error"
            });
          }
        })
        .catch();
    },
    //页面初始化获取审核商品信息
    getDetailInfo() {
      this.axios
        .get("/goodsExam/view?id=" + this.$route.query.id)
        .then(res => {
          if (res.code == "1") {
            this.businessList = res.map;
          } else {
            this.$message({
              message: res.msg,
              type: "error",
              duration: 2000
            });
          }
        })
        .catch();
    }
  }
};
</script>
<style lang='scss' scoped>
@import "../../../assets/styles/mixin";
$titleColor: #909399;
$contColor: #444;
$border: #dcdfe6;
.basic {
  .exists-info {
    @include fx(space-around);
    min-width: 500px;
    max-width: 800px;
    flex-flow: row wrap;
    margin: 0 auto;
    .exists-item {
      margin-top: 20px;
      width: 50%;
      @include fx(flex-start);
      .item-title {
        width: 120px;
        text-align: right;
        color: $titleColor;
      }
      .item-cont {
        color: $contColor;
        margin-left: 20px;
      }
    }
  }
  .header {
    min-width: 900px;
    @include fx(flex-start);
    .add-btn {
      margin-left: 10px;
    }
    .radio-group,
    .casa {
      margin-left: 100px;
    }
  }
  .goods-info {
    @include fx(space-around);
    min-width: 800px;
    max-width: 1200px;
    flex-flow: row wrap;
    margin: 0 auto;
    .info-item {
      margin-top: 20px;
      width: 50%;
      @include fx(flex-start);
      .item-title {
        width: 120px;
        text-align: right;
        color: $titleColor;
      }
      .item-cont {
        color: $contColor;
        margin-left: 20px;
      }
    }
  }
  .compare {
    border-top: 1px solid $border;
    min-width: 800px;
    max-width: 1200px;
    margin: 50px auto 30px;
    display: flex;
    justify-content: flex-start;
    .compare-item {
      border-right: 1px solid $border;
      .title {
        margin: 20px 0 0 20px;
        @include sc(18px, #111);
      }
      .areas {
        margin-left: 50px;
        margin-top: 24px;
        .area-item {
          margin-bottom: 10px;
          .province {
            margin-left: 0;
          }
          .city {
            margin-left: 30px;
          }
          .district {
            margin-left: 60px;
          }
        }
      }
      .business {
        margin: 20px 20px;
        max-height: 400px;
        overflow-y: scroll;
        .business-item {
          margin-bottom: 20px;
          .city {
            margin-left: 15px;
          }
          .table {
            margin-top: 10px;
          }
        }
      }
    }
  }
  .btn {
    @include fx(flex-end);
  }
}
</style>
