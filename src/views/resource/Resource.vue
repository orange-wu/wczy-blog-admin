<template>
  <el-card class="main-card">
    <!-- 标题 -->
    <div class="title">{{ this.$route.name }}</div>
    <div class="operation-container">
      <el-button
        type="primary"
        size="small"
        icon="el-icon-plus"
        @click="openModel(null)"
      >
        新增模块
      </el-button>
      <!-- 数据筛选 -->
      <div style="margin-left:auto">
        <el-select
          clearable
          v-model="modularSearch"
          placeholder="请选择来源"
          size="small"
          style="margin-right:1rem"
        >
          <el-option
            v-for="item in modularOptions"
            :key="item.modularId"
            :label="item.modularValue"
            :value="item.modularId"
          />
        </el-select>
        <el-button
          type="primary"
          size="small"
          icon="el-icon-search"
          style="margin-left:1rem"
          @click="listResources"
        >
          搜索
        </el-button>
      </div>
    </div>
    <!-- 权限列表 -->
    <el-table
      v-loading="loading"
      :data="resourceList"
      row-key="id"
      border
      :span-method="cellMerge"
    >
      <el-table-column prop="modularName" label="模块名" width="120" />
      <el-table-column prop="resourceName" label="接口名" width="150" />
      <el-table-column prop="url" label="接口路径" width="200" />
      <el-table-column prop="requestMethod" label="请求方式" width="100">
        <template slot-scope="scope" v-if="scope.row.requestMethod">
          <el-tag :type="tagType(scope.row.requestMethod)">
            {{ scope.row.requestMethod }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column
        prop="anonymous"
        label="匿名访问"
        align="center"
        width="100"
      >
        <template slot-scope="scope">
          <el-switch
            v-if="scope.row.url"
            v-model="scope.row.anonymous"
            @change="changeResource(scope.row)"
          />
        </template>
      </el-table-column>
      <el-table-column prop="createTime" label="创建时间" align="center">
        <template slot-scope="scope">
          <i class="el-icon-time" style="margin-right:5px" />
          {{ scope.row.createTime | dateTime }}
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" width="200">
        <template slot-scope="scope">
          <el-button
            type="text"
            size="mini"
            @click="openEditResourceModel(scope.row)"
          >
            <i class="el-icon-edit" /> 修改
          </el-button>
          <el-popconfirm
            title="确定删除吗？"
            style="margin-left:10px"
            @confirm="deleteResource(scope.row.id)"
          >
            <el-button size="mini" type="text" slot="reference">
              <i class="el-icon-delete" /> 删除
            </el-button>
          </el-popconfirm>
        </template>
      </el-table-column>
    </el-table>
    <!-- 新增模态框 -->
    <el-dialog :visible.sync="addModule" width="30%">
      <div class="dialog-title-container" slot="title" ref="moduleTitle" />
      <el-form label-width="80px" size="medium" :model="resourceForm">
        <el-form-item label="模块名">
          <el-input v-model="resourceForm.resourceName" style="width:220px" />
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addModule = false">取 消</el-button>
        <el-button type="primary" @click="addOrEditResource">
          确 定
        </el-button>
      </span>
    </el-dialog>
    <!-- 新增模态框 -->
    <el-dialog :visible.sync="addResource" width="30%">
      <div class="dialog-title-container" slot="title" ref="resourceTitle" />
      <el-form label-width="80px" size="medium" :model="resourceForm">
        <el-form-item label="资源名">
          <el-input v-model="resourceForm.resourceName" style="width:220px" />
        </el-form-item>
        <el-form-item label="资源路径">
          <el-input v-model="resourceForm.url" style="width:220px" />
        </el-form-item>
        <el-form-item label="请求方式">
          <el-radio-group v-model="resourceForm.requestMethod">
            <el-radio :label="'GET'">GET</el-radio>
            <el-radio :label="'POST'">POST</el-radio>
            <el-radio :label="'PUT'">PUT</el-radio>
            <el-radio :label="'DELETE'">DELETE</el-radio>
          </el-radio-group>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addResource = false">取 消</el-button>
        <el-button type="primary" @click="addOrEditResource">
          确 定
        </el-button>
      </span>
    </el-dialog>
  </el-card>
</template>

<script>
export default {
  created() {
    this.listResources();
  },
  data() {
    return {
      modularSearch: "",
      modularOptions: [
        {
          modularId: "",
          modularValue: ""
        }
      ],
      spanArr: [],
      loading: true,
      keywords: "",
      resourceList: [
        {
          id: "",
          parentId: "",
          modularName: "",
          resourceName: "",
          url: "",
          requestMethod: "",
          anonymous: false,
          createTime: ""
        }
      ],
      addModule: false,
      addResource: false,
      resourceForm: {}
    };
  },
  methods: {
    getSpanArr(data) {
      for (let i = 0; i < data.length; i++) {
        if (i === 0) {
          this.spanArr[0] = 1;
          this.index = 0;
        } else {
          // 判断当前元素与上一个元素是否相同
          if (data[i].parentId === data[i - 1].parentId) {
            this.spanArr[i] = 0;
            this.spanArr[this.index] += 1;
          } else {
            this.spanArr[i] = 1;
            this.index = i;
          }
        }
      }
    },
    cellMerge({ rowIndex, columnIndex }) {
      if (columnIndex === 0) {
        if (this.spanArr[rowIndex] > 0) {
          return {
            rowspan: this.spanArr[rowIndex],
            colspan: 1
          };
        } else {
          return {
            rowspan: 0,
            colspan: 0
          };
        }
      }
    },
    listResources() {
      this.axios
        .get("/api/admin/resources", {
          params: {
            keywords: this.keywords
          }
        })
        .then(({ data }) => {
          this.resourceList = data.data;
          this.loading = false;
          this.getSpanArr(this.resourceList);
        });
    },
    changeResource(resource) {
      this.axios.post("/api/admin/resources", resource).then(({ data }) => {
        if (data.flag) {
          this.$notify.success({
            title: "成功",
            message: data.message
          });
          this.listResources();
        } else {
          this.$notify.error({
            title: "失败",
            message: data.message
          });
        }
      });
    },
    openModel(resource) {
      if (resource != null) {
        this.resourceForm = JSON.parse(JSON.stringify(resource));
        this.$refs.moduleTitle.innerHTML = "修改模块";
      } else {
        this.resourceForm = {};
        this.$refs.moduleTitle.innerHTML = "添加模块";
      }
      this.addModule = true;
    },
    openEditResourceModel(resource) {
      if (resource.url == null) {
        this.openModel(resource);
        return false;
      }
      this.resourceForm = JSON.parse(JSON.stringify(resource));
      this.$refs.resourceTitle.innerHTML = "修改资源";
      this.addResource = true;
    },
    openAddResourceModel(resource) {
      console.log(resource);
      this.resourceForm = {};
      this.resourceForm.parentId = resource.id;
      this.$refs.resourceTitle.innerHTML = "添加资源";
      this.addResource = true;
    },
    deleteResource(id) {
      this.axios.delete("/api/admin/resources/" + id).then(({ data }) => {
        if (data.flag) {
          this.$notify.success({
            title: "成功",
            message: data.message
          });
          this.listResources();
        } else {
          this.$notify.error({
            title: "失败",
            message: data.message
          });
        }
      });
    },
    addOrEditResource() {
      if (this.resourceForm.resourceName.trim() === "") {
        this.$message.error("资源名不能为空");
        return false;
      }
      this.axios
        .post("/api/admin/resources", this.resourceForm)
        .then(({ data }) => {
          if (data.flag) {
            this.$notify.success({
              title: "成功",
              message: data.message
            });
            this.listResources();
          } else {
            this.$notify.error({
              title: "失败",
              message: data.message
            });
          }
          this.addModule = false;
          this.addResource = false;
        });
    }
  },
  computed: {
    tagType() {
      return function(type) {
        switch (type) {
          case "GET":
            return "";
          case "POST":
            return "success";
          case "PUT":
            return "warning";
          case "DELETE":
            return "danger";
        }
      };
    }
  }
};
</script>
