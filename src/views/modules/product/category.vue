<template>
  <div>
    <el-tree :data="menus" :props="defaultProps"
             :expand-on-click-node="false" show-checkbox node-key="catId"
             :default-expanded-keys="expandedKey" draggable
             :allow-drop="allowDrop"
             @node-drop="handleDrop"
    >

     <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <=2"
            type="text"
            size="mini"
            @click="() => append(data)">
            Append
          </el-button>
          <el-button
            type="text"
            size="mini"
            @click="() => edit(data)">
            Edit
          </el-button>
          <el-button
            v-if="node.childNodes.length===0"
            type="text"
            size="mini"
            @click="() => remove(node, data)">
            Delete
          </el-button>
        </span>
      </span>

    </el-tree>

    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      width="30%"
      :close-on-click-modal=false>
      <el-form :model="category">
        <el-form-item label="分類名稱">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="圖標">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="計量單位">
          <el-input v-model="category.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="submitData">确 定</el-button>
  </span>
    </el-dialog>
  </div>
</template>

<script>
// 这里可以导入其他文件（比如：组件，工具 js，第三方插件 js，json 文件，图片文件等等）
// 例如：import 《组件名称》 from '《组件路径》 ';

export default {
  data () {
    return {
      updateNodes: [],
      maxLevel: 0,
      // dialog標題
      title: '',
      // 控制按下確定時是要呼叫append 或edit方法
      dialogType: '',
      category: {name: '', parentCid: 0, catLevel: 0, showStatus: 1, sort: 0, catId: null, productUnit: '', icon: ''},
      dialogVisible: false,
      menus: [],
      expandedKey: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  // 方法集合
  methods: {
    append (data) {
      // 標題
      this.title = '添加分類'
      // 對話框類型
      this.dialogType = 'add'
      // 打開對話框
      this.dialogVisible = true
      // 取得參數
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel * 1 + 1

      // 開啟新增dialog 要清空數值
      this.category.catId = null
      this.category.name = ''
      this.category.icon = ''
      this.category.productUnit = ''
      this.category.sort = 0
      this.category.showStatus = 1
    },
    edit (data) {
      // 標題
      this.title = '修改分類'
      // 對話框類型
      this.dialogType = 'edit'
      // 打開對話框
      this.dialogVisible = true
      // 取得catId 查數據用
      const catId = data.catId
      // 發送請求獲取最新數據
      this.$http({
        url: this.$http.adornUrl('/product/category/info/' + catId),
        method: 'get'
      }).then(({data}) => {
        console.log('要回顯得數據', data.data)
        // 取得參數
        this.category.name = data.data.name
        this.category.catId = data.data.catId
        this.category.icon = data.data.icon
        this.category.productUnit = data.data.productUnit
      })
    },

    submitData () {
      // 新增
      if (this.dialogType === 'add') {
        this.addCategory()
      }
      // 修改
      if (this.dialogType === 'edit') {
        this.editCategory()
      }
    },

    // 添加三級分類
    addCategory () {
      // 新增API 請求
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(({data}) => {
        this.$message({
          type: 'success',
          message: '保存成功!'
        })
        // 關閉對話框
        this.dialogVisible = false
        // 新增成功刷新介面(重拉菜單)
        this.getMenus()
        // 設置需要默認展開的菜單(被新增三級菜單的父菜單)
        this.expandedKey = [this.category.parentCid]
      })
    },

    // 修改三級分類
    editCategory () {
      const {
        catId,
        name,
        icon,
        productUnit
      } = this.category

      // 修改API 請求
      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'post',
        data: this.$http.adornData({catId, name, icon, productUnit},
          false)
      }).then(({data}) => {
        this.$message({
          type: 'success',
          message: '菜單修改成功!'
        })
        // 關閉對話框
        this.dialogVisible = false
        // 新增成功刷新介面(重拉菜單)
        this.getMenus()
        // 設置需要默認展開的菜單(被新增三級菜單的父菜單)
        this.expandedKey = [this.category.parentCid]
      })
    },

    // 刪除三級分類
    remove (node, data) {
      const ids = [data.catId]
      // 刪除前加入提示
      const name = data.name
      this.$confirm('是否刪除[' + name + ']菜單?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        // 刪除API 請求
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({data}) => {
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
          // 刪除成功刷新介面(重拉菜單)
          this.getMenus()
          // 設置需要默認展開的菜單(被刪除的菜單)
          this.expandedKey = [node.parent.data.catId]
        })
      })
    },
    getMenus () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({data}) => {
        console.log('成功獲取到菜單數據..', data.data)
        this.menus = data.data
      })
    },
    allowDrop (draggingNode, dropNode, type) {
      // 1、被拖动的当前节点以及所在的父节点总层数不能大于3
      // 1）、被拖动的当前节点总层数
      console.log('allowDrop:', draggingNode, dropNode, type)
      this.countNodeLevel(draggingNode)
      // 当前正在拖动的节点+父节点所在的深度不大于3即可
      let deep = Math.abs(this.maxLevel - draggingNode.level) + 1

      if (type === 'inner') {
        return deep + dropNode.level <= 3
      } else {
        return deep + dropNode.parent.level <= 3
      }
    },
    handleDrop (draggingNode, dropNode, dropType, ev) {
      console.log('tree drop: ', dropNode.label, dropType)
      // 1.當前節點最新父節點id
      let pCid = 0
      let siblings = null

      if (dropType === 'before' || dropType === 'after') {
        pCid = dropNode.parent.data.catId === undefined ? 0 : dropNode.parent.data.catId
        siblings = dropNode.parent.childNodes
      } else {
        pCid = dropNode.data.catId
        siblings = dropNode.childNodes
      }

      // 2.當前托拽節點的最新順序
      for (let i = 0; i < siblings.length; i++) {
        // 如果遍例是當前托拽的節點
        if (siblings[i].data.catId === draggingNode.data.catId) {
          let catLevel = draggingNode.level
          if (siblings[i].level !== draggingNode.level) {
            // 如果當前節點的層級發生變化
            catLevel = siblings[i].level
            // 修改他子節點層級
            this.updateChildNodeLevel(siblings[i])
          }
          this.updateNodes.push({catId: siblings[i].data.catId, sort: i, parentCid: pCid, catLevel: catLevel})
        } else {
          this.updateNodes.push({catId: siblings[i].data.catId, sort: i})
        }
      }
      // 3.當前托拽節點的最新層級
      console.log(this.updateNodes)
      // 修改API 請求
      this.$http({
        url: this.$http.adornUrl('/product/category/update/sort'),
        method: 'post',
        data: this.$http.adornData(this.updateNodes, false)
      }).then(({data}) => {
        this.$message({
          type: 'success',
          message: '菜單順序修改成功!'
        })
        // 刷新介面(重拉菜單)
        this.getMenus()
        // 設置需要默認展開的菜單(被新增三級菜單的父菜單)
        this.expandedKey = [pCid]
        // 清空
        this.updateNodes = []
        this.maxLevel = 0
      })
    },
    // 更新子節點
    updateChildNodeLevel (node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          const cNode = node.childNodes[i].data
          this.updateNodes.push({catId: cNode.catId, catLevel: node.childNodes[i].level})
          // 遞迴繼續更新子節點下的節點
          this.updateChildNodeLevel(node.childNodes[i])
        }
      }
    },
    countNodeLevel (node) {
      // 找到所有子節點，求出最大深度
      if (node.children != null && node.children.length > 0) {
        for (let i = 0; i < node.children.length; i++) {
          if (node.children[i].catLevel > this.maxLevel) {
            this.maxLevel = node.children[i].catLevel
          }
          this.countNodeLevel(node.children[i])
        }
      }
    }
  },
  // 生命周期 - 创建完成（可以访问当前this 实例）
  created () {
    this.getMenus()
  },
  // 生命周期 - 挂载完成（可以访问 DOM 元素）
  mounted () {
  },
  beforeCreate () {
  },
  beforeMount () {
  }, // 生命周期 - 挂载之前
  beforeUpdate () {
  }, // 生命周期 - 更新之前
  updated () {
  }, // 生命周期 - 更新之后
  beforeDestroy () {
  }, // 生命周期 - 销毁之前
  destroyed () {
  }, // 生命周期 - 销毁完成
  activated () {
  } // 如果页面有 keep-alive 缓存功能,这个函数会触发
}
</script>

<style scoped>
</style>
