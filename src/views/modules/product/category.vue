<template>
  <el-tree :data="menus" :props="defaultProps"
           :expand-on-click-node="false" show-checkbox node-key="catId"
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
            v-if="node.childNodes.length===0"
            type="text"
            size="mini"
            @click="() => remove(node, data)">
            Delete
          </el-button>
        </span>
      </span>

  </el-tree>
</template>

<script>
// 这里可以导入其他文件（比如：组件，工具 js，第三方插件 js，json 文件，图片文件等等）
// 例如：import 《组件名称》 from '《组件路径》 ';
export default {
  data () {
    return {
      menus: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  // 方法集合
  methods: {
    handleNodeClick (data) {
      console.log(data)
    },
    append (data) {
      console.log('append', data)
    },
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
