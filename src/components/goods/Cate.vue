<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <!-- 表格-->
      <tree-table
        :data="cateList"
        :columns="columns"
        show-index
        index-text="#"
        :show-row-hover="false"
        :expand-type="false"
        :selection-type="false"
        border
      >
        <!-- 是否有效-->
        <template slot="isok" slot-scope="scope">
          <i class="el-icon-success" v-if="scope.row.cat_deleted===false" style="color:lightgreen;"></i>
          <i class="el-icon-error" v-else style="color:lightred;"></i>
        </template>
        <!-- 排序-->
        <template slot="order" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level===0">一级</el-tag>
          <el-tag type="warning" v-else-if="scope.row.cat_level===1">二级</el-tag>
          <el-tag type="danger" v-else>三级</el-tag>
        </template>
        <!-- 操作-->
        <template slot="opt" slot-scope="scope">
          <el-button
            type="primary"
            icon="el-icon-edit"
            size="mini"
            @click="showEditDialog(scope.row)"
          >编辑</el-button>
          <el-button
            type="danger"
            icon="el-icon-delete"
            size="mini"
            @click="deleteCate(scope.row)"
          >删除</el-button>
        </template>
      </tree-table>
      <!-- 分页-->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>
    <!-- 添加分类弹出框 -->
    <el-dialog title="添加分类" :visible.sync="addDialogVisible" width="50%" @close="addDialogClose">
      <el-form :model="addCateForm" :rules="addCateFormRule" ref="addCateForm" label-width="100px">
        <el-form-item label="分类名称：" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类：">
          <el-cascader
            v-model="selectKeys"
            :props="selectProps"
            :options="selectCateList"
            @change="handleChange"
            clearable
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑分类对话框 -->
    <el-dialog title="编辑分类" :visible.sync="editDialogVisible" width="50%" @close="editDialogClose">
      <el-form :model="editRuleForm" :rules="editRules" ref="editRefs" label-width="100px">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="editRuleForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      cateList: [],
      // 总数据条数
      total: 0,
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          type: 'template',
          template: 'isok'
        },
        {
          label: '排序',
          type: 'template',
          template: 'order'
        },
        {
          label: '操作',
          type: 'template',
          template: 'opt'
        }
      ],
      addDialogVisible: false,
      addCateForm: {
        cat_name: '',
        // 分类父 ID
        cat_pid: 0,
        // 分类当前层级
        cat_level: 0
      },
      addCateFormRule: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' },
          { min: 2, max: 10, message: '长度在 2 到 10 个字符', trigger: 'blur' }
        ]
      },
      // 选中父级分类的id数组
      selectKeys: [],
      selectCateList: [],
      selectProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        checkStrictly: true
      },
      editDialogVisible: false,
      editRuleForm: {},
      editRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ]
      },
      // 当前编辑的id
      nowCateId: ''
    }
  },
  created() {
    this.getCatelist()
  },
  methods: {
    async getCatelist() {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类列表失败')
      }
      this.cateList = res.data.result
      this.total = res.data.total
    },
    handleSizeChange(val) {
      this.queryInfo.pagesize = val
      this.getCatelist()
    },
    handleCurrentChange(val) {
      this.queryInfo.pagenum = val
      this.getCatelist()
    },
    // 弹出添加分类对话框
    async showAddDialog() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })

      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类数据失败！')
      }
      this.selectCateList = res.data
      this.addDialogVisible = true
      console.log(this.selectCateList)
    },
    // 选择项发生变化促发
    handleChange(val) {
      console.log(this.selectKeys)
      if (this.selectKeys.length > 0) {
        this.addCateForm.cat_pid = this.selectKeys[this.selectKeys.length - 1]
        this.addCateForm.cat_level = this.selectKeys.length
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
      console.log(this.addCateForm)
    },
    addDialogClose() {
      this.$refs.addCateForm.resetFields()
      this.selectKeys = []
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    },
    addCate() {
      this.$refs.addCateForm.validate(async valid => {
        if (!valid) {
          return
        }
        const { data: res } = await this.$http.post(
          'categories',
          this.addCateForm
        )
        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败')
        }
        this.$message.success('添加分类成功')
        this.getCatelist()
        this.addDialogVisible = false
      })
    },
    // 删除分类
    async deleteCate(row) {
      const confirmValue = await this.$confirm('是否确认删除分类', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      console.log(confirmValue)
      if (confirmValue !== 'cancel') {
        const { data: res } = await this.$http.delete(
          'categories/' + row.cat_id
        )
        if (res.meta.status !== 200) {
          return this.$message.error('删除分类失败')
        }
        this.$message.success('删除分类成功')
        this.getCatelist()
      }
    },
    // 弹出编辑分类
    async showEditDialog(row) {
      this.nowCateId = row.cat_id
      const { data: res } = await this.$http.get('categories/' + row.cat_id)
      if (res.meta.status !== 200) {
        return this.$message.error('获取分类信息失败')
      }
      this.editRuleForm = res.data
      this.editDialogVisible = true
    },
    editDialogClose() {
      this.$refs.editRefs.resetFields()
    },
    // 编辑分类
    editCate() {
      this.$refs.editRefs.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(
          'categories/' + this.nowCateId,
          { cat_name: this.editRuleForm.cat_name }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('编辑分类失败')
        }
        this.$message.success('编辑分类成功')
        this.nowCateId = ''
        this.getCatelist()
        this.editDialogVisible = false
      })
    }
  }
}
</script>
<style lang="less" scoped>
.zk-table {
  margin-top: 15px;
}
.el-cascader {
  display: block;
}
</style>