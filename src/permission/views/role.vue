<template>
  <div class="scrollbar blockLayout" v-loading="loading">
    <div class="flex-row align-center toolBar">
      <div class="flex-1">
        <!-- title -->
      </div>
      <el-button
        v-has="role.add"
        type="primary"
        size="small"
        icon="el-icon-plus"
        @click="dialogVisible=true"
      >添加</el-button>
    </div>
    <el-table :data="list" style="width: 100%">
      <el-table-column prop="roleName" label="角色名称" align="center"></el-table-column>
      <el-table-column prop="description" label="备注"></el-table-column>

      <el-table-column label="操作" width="300" align="center">
        <template slot-scope="scope">
          <el-button v-has="role.edit" size="mini" type="info" @click="edit(scope.row)">编辑</el-button>
          <el-button v-has="role.remove" size="mini" type="danger" @click="remove(scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 弹窗 -->
    <el-dialog
      class="comDialog"
      title="权限信息"
      :visible="dialogVisible"
      width="800px"
      @close="handleCloseDialog"
    >
      <el-form size="small" ref="editForm" :rules="rules" :model="editForm" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="备注" prop="description">
          <el-input v-model="editForm.description" type="textarea"></el-input>
        </el-form-item>

        <el-form-item label="权限">
          <resourcePicker 
            picker 
            :checked="resourceIds" 
            @checkResource="checkResource" 
            @checkMenu="checkMenu" 
          />
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="save">确 定</el-button>
        <el-button @click="dialogVisible = false">取 消</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import * as util from "@/common/assets/util";
import * as role from "../api/role";

export default {
  components: {
    resourcePicker: resolve =>
      require(["../components/resourcePicker.vue"], resolve)
  },
  data() {
    return {
      role,
      dialogVisible: false,
      loading: false,
      list: [],
      editForm: {
        roleName: "",
        description: "",
        resource: [],
        menu: []
      },
      rules: {
        roleName: [{ required: true, message: "请输入角色名称", trigger: "blur" }]
      }
    };
  },
  computed: {
    resourceIds: function(){
      return this.editForm.resource.concat(this.editForm.menu)
    }
  },
  methods: {
    checkResource(ids) {
      this.editForm.resource = ids;
    },
    checkMenu(ids) {
      this.editForm.menu = ids;
    },
    edit(data) {
      this.editForm = data;
      this.$set(this.editForm, 'resource', data.resource)
      this.$set(this.editForm, 'menu', data.menu)
      this.dialogVisible = true;
    },
    save() {
      let vm = this;
      vm.$refs["editForm"].validate(valid => {
        if (valid) {
          let formData = util.deepcopy(vm.editForm);
          this.handleCloseDialog();

          if (!formData.id) {
            role.add(formData).then(() => {
              vm.fetchData();
              vm.$message({
                message: "添加成功",
                type: "success"
              });
            });
          } else {
            role.edit(formData).then(() => {
              vm.fetchData();
              vm.$message({
                message: "编辑成功",
                type: "success"
              });
            });
          }
        }
      });
    },
    handleCloseDialog: function() {
      this.dialogVisible = false;
      this.editForm = {
        roleName: "",
        description: "",
        resource: [],
        menu: []
      };
      this.$refs.editForm && this.$refs.editForm.resetFields()
    },
    remove(item) {
      if (!item || !item.id) {
        return null;
      }
      this.$confirm("是否删除?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      }).then(() => {
        role.remove({
          id: item.id
        }).then(() => {
          this.fetchData();
          this.$message({
            message: "删除成功",
            type: "success"
          });
        });
      });
    },
    fetchData: function() {
      this.loading = true;
      role.list().then(res => {
        this.loading = false;
        this.list = res.data.data;
      });
    }
  },
  created: function() {
    this.fetchData();
  }
};
</script>

<style scoped>
</style>
