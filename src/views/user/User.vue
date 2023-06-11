<template>
  <el-card class="main-card">
    <div class="title">{{ this.$route.name }}</div>
    <!-- 表格操作 -->
    <div class="operation-container">
      <!-- 条件筛选 -->
      <div style="margin-left:auto">
        <!-- 登录方式 -->
        <el-select
          clearable
          v-model="loginType"
          placeholder="请选择登录方式"
          size="small"
          style="margin-right:1rem"
        >
          <el-option
            v-for="item in typeList"
            :key="item.type"
            :label="item.desc"
            :value="item.type"
          />
        </el-select>
        <el-input
          v-model="nickName"
          prefix-icon="el-icon-search"
          size="small"
          placeholder="请输入昵称"
          style="width:200px"
          @keyup.enter.native="searchUsers"
        />
        <el-button
          type="primary"
          size="small"
          icon="el-icon-search"
          style="margin-left:1rem"
          @click="searchUsers"
        >
          搜索
        </el-button>
      </div>
    </div>
    <!-- 表格展示 -->
    <el-table border :data="userList" v-loading="loading">
      <!-- 表格列 -->
      <el-table-column prop="avatar" label="头像" align="center" width="100">
        <template slot-scope="scope">
          <img :src="scope.row.avatar" width="40" height="40" alt="用户头像" />
        </template>
      </el-table-column>
      <el-table-column
        prop="nickname"
        label="昵称"
        align="center"
        width="140"
      />
      <el-table-column
        prop="loginType"
        label="登录方式"
        align="center"
        width="80"
      >
        <template slot-scope="scope">
          <el-tag type="success" v-if="scope.row.loginType === 1">邮箱</el-tag>
          <el-tag v-if="scope.row.loginType === 2">QQ</el-tag>
          <el-tag type="danger" v-if="scope.row.loginType === 3">微博</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="roleList" label="用户角色" align="center">
        <template slot-scope="scope">
          <el-tag
            v-for="(item, index) of scope.row.roleList"
            :key="index"
            style="margin-right:4px;margin-top:4px"
          >
            {{ item.roleName }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="isDisable" label="禁用" align="center" width="100">
        <template slot-scope="scope">
          <el-switch v-model="scope.row.disable" :disabled="true" />
        </template>
      </el-table-column>
      <el-table-column
        prop="ipAddress"
        label="登录ip"
        align="center"
        width="140"
      />
      <el-table-column
        prop="ipSource"
        label="登录地址"
        align="center"
        width="140"
      />
      <el-table-column
        prop="createTime"
        label="创建时间"
        width="130"
        align="center"
      >
        <template slot-scope="scope">
          <i class="el-icon-time" style="margin-right:5px" />
          {{ scope.row.createTime | date }}
        </template>
      </el-table-column>
      <el-table-column
        prop="lastLoginTime"
        label="上次登录时间"
        width="130"
        align="center"
      >
        <template slot-scope="scope">
          <i class="el-icon-time" style="margin-right:5px" />
          {{ scope.row.lastLoginTime | date }}
        </template>
      </el-table-column>
      <!-- 列操作 -->
      <el-table-column label="操作" align="center" width="100">
        <template slot-scope="scope">
          <el-button
            type="primary"
            size="mini"
            @click="openEditModel(scope.row)"
          >
            编辑
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <el-pagination
      class="pagination-container"
      background
      @size-change="sizeChange"
      @current-change="currentChange"
      :current-page="pageNumber"
      :page-size="pageSize"
      :total="total"
      :page-sizes="[10, 20]"
      layout="total, sizes, prev, pager, next, jumper"
    />
    <!-- 修改对话框 -->
    <el-dialog :visible.sync="isEdit" width="30%">
      <div class="dialog-title-container" slot="title">
        修改用户
      </div>
      <el-form
        label-width="auto"
        size="medium"
        :model="userForm"
        :rules="userRules"
      >
        <el-form-item label="昵称" prop="nickname">
          <el-input v-model="userForm.nickname" style="width:220px" />
        </el-form-item>
        <el-form-item label="是否禁用">
          <el-radio-group v-model="userForm.disable">
            <el-radio :label="true">是</el-radio>
            <el-radio :label="false">否</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="角色" prop="roleIdList">
          <el-checkbox-group v-model="userForm.roleIdList">
            <el-checkbox
              v-for="item of userRoleList"
              :key="item.id"
              :label="item.id"
            >
              {{ item.roleName }}
            </el-checkbox>
          </el-checkbox-group>
        </el-form-item>
      </el-form>
      <div slot="footer">
        <el-button @click="isEdit = false">取 消</el-button>
        <el-button type="primary" @click="editUserRole">
          确 定
        </el-button>
      </div>
    </el-dialog>
  </el-card>
</template>

<script>
export default {
  created() {
    this.listUsers();
    this.listRoles();
  },
  data() {
    return {
      //修改表单相关参数
      isEdit: false,
      userForm: {
        id: null,
        nickname: "",
        disable: "",
        roleIdList: []
      },
      userRules: {
        nickname: [{ required: true, message: "请输入昵称", trigger: "blur" }],
        roleIdList: [
          { required: true, message: "请选择对应角色", trigger: "blur" }
        ]
      },
      //表格相关参数
      loading: true,
      userRoleList: [],
      userList: [
        {
          id: "",
          avatar: "",
          nickname: "",
          roleList: [],
          loginType: "",
          ipAddress: "",
          ipSource: "",
          createTime: "",
          lastLoginTime: "",
          disable: false
        }
      ],
      //查询条件参数
      typeList: [
        {
          type: 1,
          desc: "邮箱"
        },
        {
          type: 2,
          desc: "QQ"
        },
        {
          type: 3,
          desc: "微博"
        }
      ],
      loginType: null,
      nickName: null,
      pageNumber: 1,
      pageSize: 10,
      total: 0
    };
  },
  methods: {
    searchUsers() {
      this.pageNumber = 1;
      this.listUsers();
    },
    sizeChange(pageSize) {
      this.pageSize = pageSize;
      this.listUsers();
    },
    currentChange(pageNumber) {
      this.pageNumber = pageNumber;
      this.listUsers();
    },
    openEditModel(user) {
      this.userForm.id = user.id;
      this.userForm.disable = user.disable;
      this.userForm.nickname = user.nickname;
      this.userForm.roleIdList = [];
      user.roleList.forEach(item => {
        this.userForm.roleIdList.push(item.id);
      });
      this.isEdit = true;
    },
    editUserRole() {
      this.$refs.userForm.validate(valid => {
        if (valid) {
          this.axios
            .put("/api/admin/updateUserRole", this.userForm)
            .then(({ data }) => {
              if (data.flag) {
                this.$notify.success({
                  title: "成功",
                  message: data.message
                });
                this.listUsers();
              } else {
                this.$notify.error({
                  title: "失败",
                  message: data.message
                });
              }
              this.isEdit = false;
            });
        }
      });
    },
    listUsers() {
      this.axios
        .get("/api/admin/users", {
          params: {
            pageNumber: this.pageNumber,
            pageSize: this.pageSize,
            nickName: this.nickName,
            loginType: this.loginType
          }
        })
        .then(({ data }) => {
          this.userList = data.data.records;
          this.total = data.data.total;
          this.loading = false;
        });
    },
    listRoles() {
      this.axios.get("/api/admin/roleIdList").then(({ data }) => {
        this.userRoleList = data.data;
      });
    }
  },
  watch: {
    loginType() {
      this.pageNumber = 1;
      this.listUsers();
    }
  }
};
</script>
