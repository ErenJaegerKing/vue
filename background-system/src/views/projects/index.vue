<template>
  <div class="app-container">
    <!-- 数据表格 -->
    <el-table
      v-loading="listLoading"
      :data="data"
      element-loading-next="加载数据中"
      border
      fit
      highlight-current-row
    >
      <!-- 序号 -->
      <el-table-column align="center" label="序号" width="60">
        <template slot-scope="scope">
          {{ scope.$index + 1 }}
        </template>
      </el-table-column>
      <!-- 项目名称 -->
      <el-table-column align="center" label="项目名称" width="150">
        <template slot-scope="scope">
          <a :href="scope.row.url" target="_blank" class="proName">{{
            scope.row.name
          }}</a>
        </template>
      </el-table-column>
      <!-- 项目描述 -->
      <el-table-column align="center" label="项目名称">
        <template slot-scope="scope">
          {{ scope.row.description.toString() }}
        </template>
      </el-table-column>
      <!-- 预览图 -->
      <el-table-column align="center" label="预览图" width="150">
        <template slot-scope="scope">
          <el-image
            :src="scope.row.thumb2"
            style="width: 120"
            :preview-src-list="srcList"
          ></el-image>
        </template>
      </el-table-column>
      <!-- 操作 -->
      <el-table-column label="操作" align="center">
        <template slot-scope="scope">
          <!-- github -->
          <el-tooltip
            class="item"
            effect="dark"
            content="GitHub"
            placement="top"
            :hide-after="2000"
          >
            <el-button
              type="primary"
              icon="iconfont icon-github"
              class="github"
              circle
              size="mini"
              @click="openGitHubHandle(scope.row)"
            ></el-button>
          </el-tooltip>
          <!-- 编辑 -->
          <el-tooltip
            class="item"
            effect="dark"
            content="编辑"
            placement="top"
            :hide-after="2000"
          >
            <el-button
              type="primary"
              icon="el-icon-edit"
              circle
              size="mini"
              @click="editProjectHandle(scope.row)"
            ></el-button>
          </el-tooltip>
          <!-- 删除 -->
          <el-tooltip
            class="item"
            effect="dark"
            content="删除"
            placement="top"
            :hide-after="2000"
          >
            <el-button
              type="danger"
              icon="el-icon-delete"
              circle
              size="mini"
              @click="deleteProjectHandle(scope.row)"
            ></el-button>
          </el-tooltip>
        </template>
      </el-table-column>
    </el-table>
    <!-- 编辑对话框 -->
    <el-dialog
      title="请编辑该项目信息"
      :visible.sync="dialogFormVisible"
      fullscreen
    >
      <el-form :model="form">
        <!-- 项目名称 -->
        <el-form-item label="项目名称">
          <el-input v-model="form.name"></el-input>
        </el-form-item>
        <!-- 项目描述 -->
        <el-form-item label="项目描述(每一项描述以英文逗号相隔)">
          <el-input v-model="form.description"></el-input>
        </el-form-item>
        <!-- 项目链接-->
        <el-form-item label="项目链接">
          <el-input v-model="form.url"></el-input>
        </el-form-item>
        <!-- github地址 -->
        <el-form-item label="GitHub地址">
          <el-input v-model="form.github"></el-input>
        </el-form-item>
        <!-- 预览图 -->
        <el-form-item label="预览图">
          <upload v-model="form.thumb" />
        </el-form-item>
        <!-- 项目等级 -->
        <el-form-item label="排序等级">
          <el-select v-model="form.order" placeholder="分类等级">
            <el-option label="1" value="1"></el-option>
            <el-option label="2" value="2"></el-option>
            <el-option label="3" value="3"></el-option>
            <el-option label="4" value="4"></el-option>
            <el-option label="5" value="5"></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="confirmEditProjectHandle"
          >确 定</el-button
        >
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { getProject, setProject, delProject } from "@/api/project";
import { server_URL } from "@/urlConfig";
import upload from "@/components/upload";
export default {
  components: {
    upload,
  },
  data() {
    return {
      data: [], // 存储数据
      listLoading: false,
      srcList: [], // 预览图数组
      dialogFormVisible: false, //一开始编辑对话框不可见
      form: {
        name: "",
        description: "",
        url: "",
        github: "",
        thumb: "",
        order: 1,
      },
    };
  },
  created() {
    this.fetchData();
  },
  methods: {
    fetchData() {
      this.listLoading = true;
      getProject().then(({ data }) => {
        this.listLoading = false;
        this.data = data;
        for (let i of this.data) {
          i.thumb2 = server_URL + i.thumb;
          this.srcList.push(i.thumb2);
        }
      });
    },
    openGitHubHandle(projectInfo) {
      window.open(projectInfo.github);
    },
    editProjectHandle(projectInfo) {
      this.dialogFormVisible = true;
      // 有个坑
      this.form = {
        ...projectInfo,
        description: projectInfo.description.toString(),
      };
    },
    deleteProjectHandle(projectInfo) {
      this.$confirm("确定要删除次项目吗?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          delProject(projectInfo.id).then(() => {
            this.fetchData();
            this.$message({
              type: "success",
              message: "删除成功!",
            });
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
    // 确认修改
    confirmEditProjectHandle() {
      let obj = { ...this.form };
      obj.description = this.form.description.split(",");
      obj.order = parseInt(this.form.order);

      setProject(obj.id, obj).then(() => {
        this.dialogFormVisible = false;
        this.fetchData();
        this.$message.success("修改成功");
      });
    },
  },
};
</script>

<style lang="scss" scoped>
.proName:hover {
  color: rgb(163, 163, 163);
}
.github {
  background-color: rgb(163, 163, 163);
  border: 1px solid rgb(163, 163, 163);
}
</style>
