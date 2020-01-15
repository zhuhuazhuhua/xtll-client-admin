<template>
  <div class="basic">
    <el-row>
      <el-col :span="24">
        <el-card>
          <div slot="header" class="header">
            <div>
              <span>账户列表</span>
              <el-button type="text" @click="addNew" class="add-btn">新增账号</el-button>
            </div>
            <el-radio-group v-model="type" class="radio-group">
              <el-radio :label="1">管理员</el-radio>
              <el-radio :label="2">代理商</el-radio>
              <el-radio :label="3">商户</el-radio>
            </el-radio-group>
            <div v-if="type != 1" class="check-group">
              <el-checkbox
                :indeterminate="isIndeterminate"
                v-model="checkAll"
                @change="handleCheckAllChange"
              >全选</el-checkbox>
              <el-checkbox-group
                class="all-group"
                v-model="checkedTypes"
                @change="handleCheckedTypesChange"
              >
                <el-checkbox
                  v-for="(item, index) in typeOptions"
                  :disabled="item == '物料审核' && type == '2'"
                  :label="item"
                  :key="index"
                >{{item}}</el-checkbox>
              </el-checkbox-group>
            </div>
            <el-checkbox
              class="unPass"
              v-model="passFirst"
              :disabled="type == '1'"
              @change="getNoPassFirstList"
            >创建审核未通过</el-checkbox>
            <el-dialog title="新增账号" :visible.sync="dialogFormVisible" :modal-append-to-body="false">
              <el-form
                :model="ruleForm"
                status-icon
                ref="ruleForm"
                :rules="rules"
                :inline="true"
                :label-position="labelPosition"
              >
                <el-form-item label="账户名称" prop="username" :label-width="formLabelWidth">
                  <el-input v-model="ruleForm.username" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="账户密码" prop="password" :label-width="formLabelWidth">
                  <el-input type="password" v-model="ruleForm.password" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="确认密码" prop="prePassword" :label-width="formLabelWidth">
                  <el-input type="password" v-model="ruleForm.prePassword" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item
                  :label="type == '1' ? '管理员姓名' : type == '2' ? '法人姓名' : '农场名'"
                  prop="name"
                  :label-width="formLabelWidth"
                >
                  <el-input v-model="ruleForm.name" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="身份证号" prop="idCard" :label-width="formLabelWidth">
                  <el-input v-model="ruleForm.idCard" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="联系方式" prop="phoneNumber" :label-width="formLabelWidth">
                  <el-input v-model="ruleForm.phoneNumber" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="账户类型" prop="userType" :label-width="formLabelWidth">
                  <el-select v-model="ruleForm.userType" placeholder="请选择">
                    <el-option
                      v-for="(item, index) in userTypeOptions"
                      :key="index"
                      :label="item.label"
                      :value="item.value"
                    ></el-option>
                  </el-select>
                </el-form-item>
                <el-form-item
                  v-if="type == '2'"
                  label="代理级别"
                  prop="agtClass"
                  :label-width="formLabelWidth"
                >
                  <el-select v-model="ruleForm.agtClass" placeholder="请选择">
                    <el-option
                      v-for="(item, index) in agtClassses"
                      :key="index"
                      :label="item.label"
                      :value="item.value"
                    ></el-option>
                  </el-select>
                </el-form-item>
                <el-form-item
                  v-if="ruleForm.userType == '1'"
                  label="管理员角色"
                  prop="role"
                  :label-width="formLabelWidth"
                >
                  <el-select v-model="ruleForm.role" placeholder="请选择" @change="changeRole">
                    <el-option
                      v-for="(item, index) in roleList"
                      :key="index"
                      :label="item.role"
                      :value="item.role"
                    ></el-option>
                  </el-select>
                </el-form-item>
                <el-form-item
                  class="area"
                  label="选择区域"
                  prop="areaChoose"
                  :label-width="formLabelWidth"
                >
                  <!-- <el-select v-model="province" placeholder="请选择">
                    <el-option
                      v-for="(item, index) in provinceOptions"
                      :key="index"
                      :label="item.label"
                      :value="item.value"
                    ></el-option>
                  </el-select>
                  <el-select v-model="city" placeholder="请选择">
                    <el-option
                      v-for="(item, index) in cityOptions"
                      :key="index"
                      :label="item.label"
                      :value="item.value"
                    ></el-option>
                  </el-select>
                  <el-select v-model="district" placeholder="请选择">
                    <el-option
                      v-for="(item, index) in districtOptions"
                      :key="index"
                      :label="item.label"
                      :value="item.value"
                    ></el-option>
                  </el-select> -->
                  <el-cascader
                    :options="areaOptions"
                    :props="props"
                    clearable
                    v-model="ruleForm.areaChoose"
                  ></el-cascader>
                </el-form-item>
                <el-form-item
                  v-if="ruleForm.userType == '1'"
                  label="权限"
                  prop="powerChoose"
                  :label-width="formLabelWidth"
                >
                  <el-checkbox-group v-model="ruleForm.powerChoose" class="checkbox-group">
                    <el-checkbox
                      v-for="(item, index) in powerList"
                      :key="index"
                      :label="item.label"
                      :value="item.value"
                    ></el-checkbox>
                  </el-checkbox-group>
                </el-form-item>
                <div v-if="ruleForm.userType != '1'">
                  <el-form-item>
                    <div class="upload-title">请上传身份证正面照</div>
                    <el-upload
                      class="uploader"
                      action
                      :show-file-list="false"
                      :auto-upload="false"
                      accept="image/jpeg, image/gif, image/png"
                      :on-change="onIdentityUploadChange"
                    >
                      <img v-if="identityImageUrl" :src="identityImageUrl" class="identity" />
                      <i v-else class="el-icon-plus uploader-icon"></i>
                    </el-upload>
                  </el-form-item>
                  <el-form-item>
                    <div class="upload-title">请上传营业执照</div>
                    <el-upload
                      class="uploader"
                      action
                      :auto-upload="false"
                      :show-file-list="false"
                      :on-change="onBusinessUploadChange"
                    >
                      <img v-if="businessImageUrl" :src="businessImageUrl" class="identity" />
                      <i v-else class="el-icon-plus uploader-icon"></i>
                    </el-upload>
                  </el-form-item>
                </div>
              </el-form>
              <div slot="footer" class="dialog-footer">
                <el-button type="primary" @click="submitForm('ruleForm')">提 交</el-button>
                <el-button type="primary" plain @click="resetForm('ruleForm')">重 置</el-button>
                <el-button type="info" @click="dialogFormVisible = false">取 消</el-button>
              </div>
            </el-dialog>
            <el-dialog title="修改账号" :visible.sync="dialogEditVisible" :modal-append-to-body="false">
              <el-form status-icon :inline="true" :label-position="labelPosition">
                <div></div>
                <el-form-item label="账户密码" :label-width="formLabelWidth">
                  <el-input type="text" v-model="editPassword" autocomplete="off"></el-input>
                </el-form-item>
                <el-button type="primary" @click="update('1')" :disabled="!editPassword">修改</el-button>
                <div></div>
                <el-form-item
                  :label="type == '1' ? '管理员姓名' : type == '2' ? '法人姓名' : '农场名'"
                  :label-width="formLabelWidth"
                >
                  <el-input v-model="editName" autocomplete="off"></el-input>
                </el-form-item>
                <el-button type="primary" @click="update('4')" :disabled="!editName">修改</el-button>
                <div></div>
                <el-form-item label="联系方式" :label-width="formLabelWidth">
                  <el-input v-model="editTelephone" autocomplete="off"></el-input>
                </el-form-item>
                <el-button type="primary" @click="update('2')" :disabled="!editTelephone">修改</el-button>
                <div></div>
                <el-form-item v-if="type == '1'" label="管理员角色" :label-width="formLabelWidth">
                  <el-select v-model="editRoleName" placeholder="请选择">
                    <el-option
                      v-for="(item, index) in roleList"
                      :key="index"
                      :label="item.role"
                      :value="item.role"
                    ></el-option>
                  </el-select>
                  <el-button type="primary" @click="update('6')" :disabled="!editRoleName">修改</el-button>
                </el-form-item>
                <el-form-item class="area" label="选择区域" :label-width="formLabelWidth">
                  <!-- <el-cascader
                    :options="areaOptions"
                    :props="props"
                    clearable
                    v-model="changeAreaChoose"
                  ></el-cascader>-->
                  <!-- <el-button type="primary" @click="update('3')" :disabled="!changeAreaChoose.length">修改</el-button> -->
                </el-form-item>
                <el-form-item v-if="type == '1'" label="权限" :label-width="formLabelWidth">
                  <el-checkbox-group v-model="editPower" class="checkbox-group">
                    <el-checkbox
                      v-for="(item, index) in powerList"
                      :key="index"
                      :label="item.label"
                      :value="item.value"
                    ></el-checkbox>
                  </el-checkbox-group>
                  <el-button type="primary" @click="update('5')" :disabled="!editPower">修改</el-button>
                </el-form-item>
              </el-form>
              <div slot="footer" class="dialog-footer">
                <el-button type="info" @click="dialogEditVisible = false">取 消</el-button>
              </div>
            </el-dialog>
          </div>
          <div class="table-wrapper">
            <el-table
              v-loading="loading"
              element-loading-text="加载数据中"
              element-loading-spinner="el-icon-loading"
              :data="tableData"
              border
            >
              <el-table-column type="expand">
                <template slot-scope="props">
                  <el-form label-position="left" class="table-expand">
                    <el-form-item label="账号">
                      <span>{{props.row.loginName}}</span>
                    </el-form-item>
                    <el-form-item v-if="type == '1'" label="角色">
                      <span>{{props.row.roleName}}</span>
                    </el-form-item>
                    <el-form-item :label="type == '1' ? '管理员名' : type == '2' ? '法人名' : '农场名'">
                      <span>{{props.row.name}}</span>
                    </el-form-item>
                    <el-form-item label="手机号">
                      <span>{{ props.row.telephone }}</span>
                    </el-form-item>
                    <el-form-item label="所属区域">
                      <div
                        class
                        v-for="(item, index) in props.row.areas"
                        :key="index"
                      >{{item.province && item.city && item.district && item.province + ',' + item.city + ',' + item.district}}</div>
                    </el-form-item>
                    <el-form-item v-if="type == 1" label="权限">
                      <span>{{ getPower(props.row.perm) }}</span>
                    </el-form-item>
                  </el-form>
                </template>
              </el-table-column>
              <el-table-column label="账号" prop="loginName" align="center"></el-table-column>
              <el-table-column v-if="type == 1" label="角色" prop="roleName" align="center"></el-table-column>
              <el-table-column
                :label="type == '1' ? '管理员名' : type == '2' ? '法人名' : '农场名'"
                prop="name"
                align="center"
              ></el-table-column>
              <el-table-column label="手机号" prop="telephone" align="center"></el-table-column>
              <!-- <el-table-column label="状态" prop="status" align="center"></el-table-column> -->
              <el-table-column label="操作" align="center">
                <template slot-scope="scope">
                  <el-button
                    size="mini"
                    type="primary"
                    v-if="scope.row.status != '1' && scope.row.status != '4' && type != '1'"
                    @click="showExam(scope.row)"
                  >{{scope.row.status == '2' ? '创建审核' : '物料审核'}}</el-button>
                  <el-button
                    size="mini"
                    type="primary"
                    plain
                    @click="handleEdit(scope.$index, scope.row)"
                  >
                    <i class="el-icon-edit operate-btn"></i>编辑
                  </el-button>
                  <el-button
                    size="mini"
                    type="danger"
                    plain
                    @click="handleDelete(scope.$index, scope.row)"
                  >
                    <i class="el-icon-delete operate-btn"></i>删除
                  </el-button>
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
        <!-- 审核弹框 -->
        <el-dialog :visible.sync="dialogExamVisible" :modal-append-to-body="false">
          <exam :objItem="examItem" @examUp="commit" :type="type"></exam>
        </el-dialog>
      </el-col>
    </el-row>
  </div>
</template>
<script>
// import { formatDate } from "src/utils/utils";
// import score from "src/components/Score/index";
import exam from "src/components/exam";
import areaJson from "src/data/city";
export default {
  components: {
    exam
  },
  mounted() {
    this.getTableData();
    this.getRoleList();
  },
  data() {
    var validatePass = (rule, value, callback) => {
      if (value === "") {
        callback(new Error("请输入密码"));
      } else {
        if (this.ruleForm.prePassword !== "") {
          this.$refs.ruleForm.validateField("prePassword");
        }
        callback();
      }
    };
    var validatePass2 = (rule, value, callback) => {
      if (value === "") {
        callback(new Error("请再次输入密码"));
      } else if (value !== this.ruleForm.password) {
        callback(new Error("两次输入密码不一致!"));
      } else {
        callback();
      }
    };
    var validateRole = (rule, value, callback) => {
      if (this.ruleForm.userType == "1" && value === "") {
        callback(new Error("请选择管理员角色"));
      } else {
        callback();
      }
    };
    var checkPhone = (rule, value, callback) => {
      if (!value) {
        return callback(new Error("请输入手机号"));
      } else {
        const reg = /^1[3|4|5|7|8][0-9]\d{8}$/;
        if (reg.test(value)) {
          callback();
        } else {
          return callback(new Error("请输入正确的手机号"));
        }
      }
    };
    var validateIdCard = (rule, value, callback) => {
      if (!value) {
        return callback(new Error("请输入身份证号"));
      } else {
        const reg = /^\d{17}[(0-9)|X|x ]{1}/;
        if (reg.test(value)) {
          callback();
        } else {
          return callback(new Error("请输入正确的身份证号"));
        }
      }
    };
    const allTypeOptions = ["已通过", "创建审核", "物料审核"];
    return {
      // provinceOptions: [],
      // cityOptions: [],
      // districtOptions: [],
      // province: '',
      // city: '',
      // district: '',
      agtClassses: [
        { value: "1", label: "省代理(15%分红)" },
        { value: "2", label: "市代理(8%分红)" },
        { value: "3", label: "区代理(5%分红)" }
      ],
      checkAll: false,
      isIndeterminate: true,
      checkedTypes: allTypeOptions,
      typeOptions: allTypeOptions,
      tableData: [],
      loading: true,
      pagesize: 10,
      currentpage: 1,
      total: 0,
      dialogFormVisible: false,
      dialogEditVisible: false,
      ruleForm: {
        name: "",
        username: "",
        password: "",
        prePassword: "",
        phoneNumber: "",
        idCard: "",
        userType: "1",
        role: "",
        powerChoose: [],
        // areaChoose: [],
        agtClass: ""
      },
      userFormData: {},
      rules: {
        name: [
          { required: true, message: "请输入账户人姓名", trigger: "blur" }
        ],
        username: [
          { required: true, message: "请输入您要创建的账户名", trigger: "blur" }
        ],
        phoneNumber: [{ validator: checkPhone, trigger: "blur" }],
        idCard: [{ validator: validateIdCard, trigger: "blur" }],
        password: [{ validator: validatePass, trigger: "blur" }],
        prePassword: [{ validator: validatePass2, trigger: "blur" }],
        role: [{ validator: validateRole, trigger: "change" }],
        userType: [
          { required: true, message: "请选择账户类型", trigger: "change" }
        ],
        powerChoose: [
          { required: true, message: "请选择权限", trigger: "change" }
        ],
        // areaChoose: [
        //   { required: true, message: "请选择区域", trigger: "change" }
        // ],
        agtClass: [
          { required: true, message: "请选择代理级别", trigger: "change" }
        ]
      },
      formLabelWidth: "100px",
      roleList: [],
      userTypeOptions: [
        { value: "1", label: "管理员" },
        { value: "2", label: "代理商" },
        { value: "3", label: "商户" }
      ],
      powerList: [
        { value: "1", label: "上架" },
        { value: "2", label: "下架" },
        { value: "3", label: "商品审核" },
        { value: "4", label: "商户审核" },
        { value: "5", label: "代理商添加" },
        { value: "6", label: "代理商修改" },
        { value: "7", label: "代理商删除" },
        { value: "8", label: "商户添加" },
        { value: "9", label: "商户修改" },
        { value: "10", label: "商户删除" }
      ],
      labelPosition: "left",
      identityImageUrl: "", //身份证
      businessImageUrl: "", //营业执照
      // props: { multiple: true }, //地区允许多选
      areas: [],
      // areaOptions: areaJson,
      type: 1, //账户列表类型,
      editName: "",
      editPassword: "",
      editTelephone: "",
      id: "",
      editRoleName: "", //修改类型
      editPower: [], //修改权限
      // changeAreas: [],
      // changeAreaChoose: [], //修改区域,
      dialogExamVisible: false,
      examItem: {},
      passFirst: false //创建审核是否通过
    };
  },
  computed: {},
  watch: {
    type(value) {
      this.passFirst = false;
      if (value == 1) {
        this.getTableData();
      } else {
        this.checkedTypes = this.typeOptions;
        this.checkAll = true;
        this.sort = "1,2,3";
        this.getTableData("1,2,3");
      }
      this.ruleForm.userType = value;
    }
  },
  methods: {
    agtClassChoose(e) {
      console.log(e);
    },
    //创建审核未通过列表
    getNoPassFirstList() {
      console.log(this.passFirst);
      if (this.passFirst) {
        this.checkAll = false;
        this.checkedTypes = [];
        this.getNoPassFirstTable();
      } else {
        this.tableData = [];
      }
    },
    getNoPassFirstTable() {
      this.axios
        .get(
          "/fail/exam1?type=" +
            this.type +
            "&&pageNum=" +
            this.currentpage +
            "&&pageSize=" +
            this.pagesize
        )
        .then(res => {
          if (res.code == "1") {
            this.tableData = res.list.rows;
          }
        })
        .catch();
    },
    //展示审核弹框
    showExam(examItem) {
      this.examItem = examItem;
      this.dialogExamVisible = true;
    },
    //提交审核
    commit(e) {
      console.log(e);
      if (e.commitType != "1") {
        let data = {
          id: this.examItem.id,
          status: this.examItem.status,
          type: this.type,
          reason: e.reason ? e.reason : ""
        };
        this.dialogExamVisible = false;
        this.axios
          .post("/exam", data)
          .then(res => {
            if (res.code == "1") {
              this.$message({
                message: res.msg,
                type: "success",
                duration: 2000
              });
              this.dialogExamVisible = false;
              this.getTableData();
              return false;
            }
          })
          .catch();
      }
      this.dialogExamVisible = false;
    },
    getPower(perm) {
      let power = [];
      $.each(this.powerList, (index, item) => {
        $.each(perm.split(","), (keyIndex, keyItem) => {
          if (item.value == keyItem) {
            power.push(item.label);
          }
        });
      });
      return power.join(",");
    },
    //不区分审核状态划分
    handleCheckAllChange(val) {
      this.passFirst = false;
      this.checkedTypes = val ? this.typeOptions : [];
      this.isIndeterminate = false;
      this.statusSort(this.checkedTypes.length);
    },
    //审核状态分类

    handleCheckedTypesChange(val) {
      this.passFirst = false;
      this.checkAll = val.length == this.typeOptions.length;
      this.isIndeterminate =
        val.length > 0 && val.length < this.typeOptions.length;
      this.statusSort(val.length);
    },
    statusSort(len) {
      if (!len) {
        this.tableData = [];
      } else {
        console.log(this.checkedTypes);
        let sort = [];
        this.checkedTypes.forEach(item => {
          sort.push(item == "已通过" ? "1" : item == "创建审核" ? "2" : "3");
        });
        this.sort = sort.join(",");
        this.getTableData(sort);
      }
    },
    //新增账号打开dialog，并设置当前类型
    addNew() {
      this.dialogFormVisible = true;
      this.ruleForm.userType = this.type + "";
    },
    changeRole() {
      var sub = this.roleList.find(item => {
        return item.role == this.ruleForm.role;
      });
      let keys = sub.perm.split(",");
      let powerChoose = [];
      $.each(this.powerList, (index, item) => {
        $.each(keys, (keyIndex, keyItem) => {
          if (item.value == keyItem) {
            powerChoose.push(item.label);
          }
        });
      });
      this.ruleForm.powerChoose = [];
      for (let i = 0; i < powerChoose.length; i++) {
        this.$set(this.ruleForm.powerChoose, i, powerChoose[i]);
      }
    },
    getRoleList() {
      let roleList = localStorage.getItem("roleList");
      if (roleList) {
        this.roleList = JSON.parse(roleList);
      } else {
        this.axios
          .get("/role/selectList")
          .then(res => {
            if (res.code == 1) {
              this.roleList = res.data;
            }
          })
          .catch(error => {});
      }
    },
    handleClose(done) {
      this.$confirm("确认关闭？")
        .then(_ => {
          done();
        })
        .catch(_ => {});
    },
    //本地浏览身份证图片
    onIdentityUploadChange(file) {
      this.localIdentityFile = file.raw; // 或者 this.localFile=file.raw
      let reader = new FileReader();
      reader.readAsDataURL(this.localIdentityFile); //这里也可以直接写参数event.raw
      reader.onload = () => {
        this.identityImageUrl = reader.result;
      };
    },
    //本地预览营业执照图片
    onBusinessUploadChange(file) {
      this.localBusinessFile = file.raw; // 或者 this.localFile=file.raw
      let reader = new FileReader();
      reader.readAsDataURL(this.localBusinessFile); //这里也可以直接写参数event.raw
      reader.onload = () => {
        this.businessImageUrl = reader.result;
      };
    },
    //新增账号表单提交,
    submitForm(formName) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          this.areas = [];
          for (var i = 0; i < this.ruleForm.areaChoose.length; i++) {
            this.areas.push(
              this.ruleForm.areaChoose[i][
                this.ruleForm.areaChoose[i].length - 1
              ]
            );
          }
          let formData = new FormData();
          if (this.ruleForm.userType == "1") {
            let url = "/user/add";
            let power = [];
            $.each(this.powerList, (index, item) => {
              $.each(this.ruleForm.powerChoose, (keyIndex, keyItem) => {
                if (item.label == keyItem) {
                  power.push(item.value);
                }
              });
            });
            let userFormData = {
              areasNum: this.areas,
              loginName: this.ruleForm.username,
              name: this.ruleForm.name,
              password: this.ruleForm.password,
              power: power,
              roleName: this.ruleForm.role,
              telephone: this.ruleForm.phoneNumber,
              userCardid: this.ruleForm.idCard
            };
            console.log(this.areas);
            this.axios
              .post("/user/add", userFormData, {
                headers: { "Content-Type": "application/json" }
              })
              .then(res => {
                this.success(res);
              })
              .catch(() => {});
            // this.add(url, userFormData);
          } else {
            formData.append("cardIdfile", this.localIdentityFile);
            formData.append("certenfile", this.localBusinessFile);
            formData.append("userCardid", this.ruleForm.idCard);
            formData.append("loginName", this.ruleForm.username);
            formData.append("name", this.ruleForm.name);
            formData.append("password", this.ruleForm.password);
            formData.append("areasNum", this.areas);
            formData.append("telephone", this.ruleForm.phoneNumber);
            if (type == "2") {
              formData.append("telephone", this.ruleForm.agtClass);
            }
            let url = this.ruleForm.userType == "2" ? "/agt/add" : "/buss/add";
            this.axios
              .post(url, formData, {
                headers: { "Content-Type": "multipart/form-data" }
              })
              .then(res => {
                this.success(res);
              })
              .catch(() => {});
          }
        } else {
          console.log("error submit!!");
          return false;
        }
      });
    },
    //修改信息
    update(number) {
      // console.log(this.changeAreaChoose);
      if (number == "5") {
        this.power = [];
        $.each(this.powerList, (index, item) => {
          $.each(this.editPower, (keyIndex, keyItem) => {
            if (item.label == keyItem) {
              this.power.push(item.value);
            }
          });
        });
      }
      if (number == "3") {
        this.changeAreas = [];
        // for (var i = 0; i < this.changeAreaChoose.length; i++) {
        //   this.changeAreas.push(
        //     this.changeAreaChoose[i][this.changeAreaChoose[i].length - 1]
        //   );
        // }
      }
      var data =
        number == "1"
          ? { password: this.editPassword, number: number, id: this.id }
          : number == "2"
          ? { telephone: this.editTelephone, number: number, id: this.id }
          : number == "3"
          ? { areasNum: this.changeAreas, number: number, id: this.id }
          : number == "4"
          ? { name: this.editName, number: number, id: this.id }
          : number == "5"
          ? { power: this.power, number: number, id: this.id }
          : { roleName: this.editRoleName, number: number, id: this.id };
      let url =
        this.type == 1
          ? "/sys/update/user"
          : this.type == 2
          ? "/sys/update/agt"
          : "/sys/update/buss";
      this.axios
        .post(url, data)
        .then(res => {
          this.delUserSuccess(res);
        })
        .catch(error => {});
    },
    //添加账号
    add(url, formData, config) {
      this.axios.post(url, formData, config).then(res => {
        this.success(res);
      });
    },
    //添加账号成功回调
    success(res) {
      if (res.code == 1) {
        this.$message({
          message: res.msg,
          type: "success",
          duration: 2000
        });
        this.resetForm("ruleForm");
        this.getTableData();
      } else {
        this.$message({
          message: res.msg,
          type: "error",
          duration: 2000
        });
      }
    },
    //删除账号成功回调
    delUserSuccess(res) {
      if (res.code == 1) {
        this.$message({
          message: res.msg,
          type: "success",
          duration: 2000
        });
        this.getTableData();
      } else {
        this.$message({
          message: res.msg,
          type: "error",
          duration: 2000
        });
      }
    },
    //重置表单
    resetForm(formName) {
      this.$refs[formName].resetFields();
      this.identityImageUrl = "";
      this.businessImageUrl = "";
      this.areas = [];
    },
    //页面初始化获取数据列表
    getTableData(allsort) {
      let status = allsort ? allsort : this.sort;
      let data =
        this.type == 1
          ? "?pageSize=" + this.pagesize + "&&pageNum=" + this.currentpage
          : "?pageSize=" +
            this.pagesize +
            "&&pageNum=" +
            this.currentpage +
            "&&sort=" +
            status;
      let url =
        this.type == 1
          ? "/user/selectList"
          : this.type == 2
          ? "/agt/selectList"
          : "/buss/selectList";
      this.axios
        .get(url + data)
        .then(data => {
          if (data.code == 1) {
            this.tableData = data.dataList.rows;
            this.total = data.dataList.total;
            this.loading = false;
          } else {
            this.tableData = [];
          }
        })
        .catch(error => {});
    },
    //账号编辑
    handleEdit(index, row) {
      this.dialogEditVisible = true;
      console.log(index, row);
      // this.editPassword = row.password
      this.editName = row.name;
      this.editTelephone = row.telephone;
      this.id = row.id;
      this.editRoleName = this.type == 1 ? row.roleName : "";
      // this.changeAreaChoose = row.areaNums;
      if (this.type == 1) {
        let keys = row.perm.split(",");
        let powerChoose = [];
        $.each(this.powerList, (index, item) => {
          $.each(keys, (keyIndex, keyItem) => {
            if (item.value == keyItem) {
              powerChoose.push(item.label);
            }
          });
        });
        this.editPower = [];
        for (let i = 0; i < powerChoose.length; i++) {
          this.$set(this.editPower, i, powerChoose[i]);
        }
      }
    },
    //账号删除
    handleDelete(index, row) {
      let name = row.loginName;
      let string =
        this.type == 1 ? "管理员" : this.type == 2 ? "代理商" : "商户";
      this.$confirm(`确定删除 ${name} ${string}？`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          let url =
            this.type == 1
              ? "/sys/del/user"
              : this.type == 2
              ? "/sys/del/agt"
              : "/sys/del/buss";
          let data = {
            loginName: row.loginName
          };
          this.axios
            .post(url, data)
            .then(res => {
              this.delUserSuccess(res, row);
            })
            .catch(() => {});
        })
        .catch(() => {});
    },
    handleSizeChange(value) {
      this.pagesize = value;
      this.getTableData();
    },
    handleCurrentChange(value) {
      this.currentpage = value;
      console.log(value);
      this.getTableData();
    }
  }
};
</script>
<style lang='scss' scoped>
@import "../../assets/styles/mixin";
.basic {
  .header {
    @include fx(flex-start);
    .add-btn {
      margin-left: 10px;
    }
    .radio-group {
      @include fx(space-around);
      margin-left: 100px;
    }
    .check-group {
      margin-left: 100px;
      display: flex;
      .all-group {
        margin-left: 20px;
      }
    }
    .unPass {
      margin-left: 100px;
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
    .operate-btn {
      margin-right: 3px;
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
  .upload-title {
    text-align: center;
  }
  .uploader .el-upload {
    border: 1px dashed $uploadBorder;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
    .uploader-icon {
      @include wh(178px, 178px);
      @include sc(28px, #8c939d);
      line-height: 178px;
      text-align: center;
    }
    .identity {
      @include wh(178px, 178px);
      display: block;
    }
  }
  .uploader .el-upload:hover {
    border-color: $uploadHoverBorder;
  }
  // .inline {
  //   display: flex;
  //   justify-content: start;
  // }
}
</style>
