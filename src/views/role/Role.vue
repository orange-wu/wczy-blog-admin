<template>
  <el-card class="main-card">
    <div class="title">{{ this.$route.name }}</div>
    <!-- 表格操作 -->
    <div class="operation-container">
      <el-button
        type="primary"
        size="small"
        icon="el-icon-plus"
        @click="openRoleModel(null)"
      >
        新增
      </el-button>
      <!-- 条件筛选 -->
      <div style="margin-left:auto">
        <el-input
          v-model="roleName"
          prefix-icon="el-icon-search"
          size="small"
          placeholder="请输入角色名"
          style="width:200px"
          @keyup.enter.native="searchRoles"
        />
        <el-button
          type="primary"
          size="small"
          icon="el-icon-search"
          style="margin-left:1rem"
          @click="searchRoles"
        >
          搜索
        </el-button>
      </div>
    </div>
    <!-- 表格展示 -->
    <el-table border :data="roleList" v-loading="loading">
      <el-table-column prop="roleName" label="角色名" align="center" />
      <el-table-column prop="roleLabel" label="权限名" align="center">
        <template slot-scope="scope">
          <el-tag>
            {{ scope.row.roleLabel }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="isDisable" label="禁用" align="center" width="100">
        <template slot-scope="scope">
          <el-switch v-model="scope.row.disable" :disabled="true" />
        </template>
      </el-table-column>
      <el-table-column
        prop="createTime"
        label="创建时间"
        width="150"
        align="center"
      >
        <template slot-scope="scope">
          <i class="el-icon-time" style="margin-right:5px" />
          {{ scope.row.createTime | date }}
        </template>
      </el-table-column>
      <!-- 列操作 -->
      <el-table-column label="操作" align="center" width="220">
        <template slot-scope="scope">
          <el-button type="text" size="mini" @click="openRoleModel(scope.row)">
            <i class="el-icon-edit" /> 角色权限
          </el-button>
          <el-popconfirm
            title="确定删除吗？"
            style="margin-left:10px"
            @confirm="deleteRoles(scope.row.id)"
          >
            <el-button size="mini" type="text" slot="reference">
              <i class="el-icon-delete" /> 删除
            </el-button>
          </el-popconfirm>
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
    <!-- 菜单对话框 -->
    <el-dialog :visible.sync="roleShow" width="30%">
      <div class="dialog-title-container" slot="title" ref="roleTitle" />
      <el-form
        label-width="auto"
        size="medium"
        :model="roleForm"
        :rules="formRules"
        ref="formItem"
      >
        <el-row>
          <el-col :span="8">
            <el-form-item label="角色名" prop="roleName">
              <el-input v-model="roleForm.roleName" style="width:100%" />
            </el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item label="权限名" prop="roleLabel">
              <el-input
                v-model="roleForm.roleLabel"
                style="width:100%"
                :disabled="roleLabelUpdate"
              />
            </el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item label="是否禁用">
              <el-radio-group v-model="roleForm.disable">
                <el-radio :label="true">是</el-radio>
                <el-radio :label="false">否</el-radio>
              </el-radio-group>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="12">
            <el-form-item label="菜单权限">
              <el-tree
                :data="menuList"
                :default-checked-keys="roleForm.menuIds"
                accordion
                show-checkbox
                node-key="id"
                ref="menuTree"
              />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="资源权限" label-width="auto">
              <el-tree
                :data="resourceList"
                :default-checked-keys="roleForm.resourceIds"
                accordion
                show-checkbox
                node-key="id"
                ref="resourceTree"
              />
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
      <div slot="footer">
        <el-button @click="roleShow = false">取 消</el-button>
        <el-button type="primary" @click="saveOrUpdateRole">
          确 定
        </el-button>
      </div>
    </el-dialog>
  </el-card>
</template>

<script>
export default {
  created() {
    this.listRoles();
  },
  data: function() {
    return {
      loading: true,
      isDelete: false,
      roleList: [],
      roleName: null,
      pageNumber: 1,
      pageSize: 10,
      total: 0,
      roleShow: false,
      roleLabelUpdate: false,
      resourceList: [],
      menuList: [],
      roleForm: {
        id: null,
        roleName: "",
        roleLabel: "",
        disable: false,
        resourceIds: [],
        menuIds: []
      },
      formRules: {
        roleName: [
          { required: true, message: "请输入角色名", trigger: "blur" }
        ],
        roleLabel: [
          { required: true, message: "请输入权限名", trigger: "blur" }
        ]
      }
    };
  },
  methods: {
    searchRoles() {
      this.pageNumber = 1;
      this.listRoles();
    },
    sizeChange(pageSize) {
      this.pageSize = pageSize;
      this.listRoles();
    },
    currentChange(pageNumber) {
      this.pageNumber = pageNumber;
      this.listRoles();
    },
    listRoles() {
      this.axios
        .get("/api/admin/roles", {
          params: {
            pageNumber: this.pageNumber,
            pageSize: this.pageSize,
            roleName: this.roleName
          }
        })
        .then(({ data }) => {
          this.roleList = data.data.records;
          this.total = data.data.total;
          this.loading = false;
        });
      this.roleResourceList();
      this.roleMenuList();
    },
    roleResourceList() {
      this.axios.get("/api/admin/role/resources").then(({ data }) => {
        this.resourceList = data.data;
      });
    },
    roleMenuList() {
      this.axios.get("/api/admin/role/menus").then(({ data }) => {
        this.menuList = data.data;
      });
    },
    deleteRoles(id) {
      this.axios.delete("/api/admin/roles/" + id).then(({ data }) => {
        if (data.flag) {
          this.$notify.success({
            title: "成功",
            message: data.message
          });
          this.listRoles();
        } else {
          this.$notify.error({
            title: "失败",
            message: data.message
          });
        }
        this.isDelete = false;
      });
    },
    openRoleModel(role) {
      this.$refs.roleTitle.innerHTML = role ? "修改角色" : "新增角色";
      if (role != null) {
        this.roleLabelUpdate = true;
        this.roleForm = JSON.parse(JSON.stringify(role));
      } else {
        this.roleLabelUpdate = false;
        this.roleForm = {
          roleName: "",
          roleLabel: "",
          disable: false,
          resourceIds: [],
          menuIds: []
        };
      }
      this.roleShow = true;
      this.$nextTick(() => {
        this.$refs.menuTree.setCheckedKeys(
          this.roleForm.menuIds == null ? [] : this.roleForm.menuIds
        );
        this.$refs.resourceTree.setCheckedKeys(
          this.roleForm.resourceIds == null ? [] : this.roleForm.resourceIds
        );
      });
    },
    saveOrUpdateRole() {
      this.$refs.formItem.validate(valid => {
        if (valid) {
          this.roleForm.resourceIds = this.$refs.resourceTree
            .getCheckedKeys()
            .concat(this.$refs.menuTree.getHalfCheckedKeys());
          this.roleForm.menuIds = this.$refs.menuTree
            .getCheckedKeys()
            .concat(this.$refs.menuTree.getHalfCheckedKeys());
          let api;
          if (this.roleForm.id == null) {
            api = "/api/admin/saveRole";
          } else {
            api = "/api/admin/updateRole";
          }
          this.axios.post(api, this.roleForm).then(({ data }) => {
            if (data.flag) {
              this.$notify.success({
                title: "成功",
                message: data.message
              });
              this.listRoles();
            } else {
              this.$notify.error({
                title: "失败",
                message: data.message
              });
            }
            this.roleShow = false;
          });
        }
      });
    }
  }
};
</script>
