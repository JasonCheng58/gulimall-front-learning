<template>
  <el-tree :data="data" :props="defaultProps" @node-click="handleNodeClick"></el-tree>
</template>

<script>
// 这里可以导入其他文件（比如：组件，工具 js，第三方插件 js，json 文件，图片文件等等）
// 例如：import 《组件名称》 from '《组件路径》 ';
export default {
  data () {
    return {
      data: []
    }
  },
  // 方法集合
  methods: {
    handleNodeClick (data) {
      console.log(data)
    },
    getMenus () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(data => {
        console.log('成功')
      })
    }
  },
  // 生命周期 - 创建完成（可以访问当前this 实例）
  created () {
  },
  // 生命周期 - 挂载完成（可以访问 DOM 元素）
  mounted () {
    this.getMenus()
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