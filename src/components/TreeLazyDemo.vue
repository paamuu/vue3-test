<script setup>
import { ref } from 'vue'

// 树节点数据（初始化时就有多级节点，并全部展开）
const treeData = ref([
  {
    id: 1,
    label: '节点 1',
    hasLazy: true,
    children: [
      { id: 11, label: '节点 1-1', hasLazy: true },
      { id: 12, label: '节点 1-2' },
    ],
  },
  {
    id: 2,
    label: '节点 2',
    hasLazy: true,
    children: [
      { id: 21, label: '节点 2-1' },
      {
        id: 22,
        label: '节点 2-2',
        hasLazy: true,
        children: [{ id: 221, label: '节点 2-2-1' }],
      },
    ],
  },
  {
    id: 3,
    label: '节点 3',
    children: [{ id: 31, label: '节点 3-1' }],
  },
])

// el-tree 实例
const treeRef = ref(null)

// el-tree 的 props 配置
const defaultProps = {
  children: 'children',
  label: 'label',
}

// 模拟异步接口：为某个节点懒加载子节点
const loadChildren = (parent) => {
  return new Promise((resolve) => {
    setTimeout(() => {
      const base = Date.now()
      resolve([
        {
          id: base,
          label: `${parent.label} 的懒加载子节点 1`,
          hasLazy: false,
        },
        {
          id: base + 1,
          label: `${parent.label} 的懒加载子节点 2`,
          hasLazy: false,
        },
      ])
    }, 500)
  })
}

// 点击已展开节点时，按需懒加载其子节点
const handleNodeClick = async (data) => {
  // 没有开启懒加载标记，或者已经加载过子节点，则不再处理
  if (!data.hasLazy || data.childrenLoaded) return

  // 标记为已加载，防止重复请求
  data.childrenLoaded = true

  const children = await loadChildren(data)

  if (!children || !children.length || !treeRef.value) return

  // 使用 el-tree 暴露的 append 方法，将新子节点挂到当前节点下
  children.forEach((child) => {
    treeRef.value.append(child, data)
  })
}
</script>

<template>
  <div class="tree-wrapper">
    <el-tree
      ref="treeRef"
      :data="treeData"
      node-key="id"
      default-expand-all
      :props="defaultProps"
      @node-click="handleNodeClick"
    />
  </div>
</template>

<style scoped>
.tree-wrapper {
  max-width: 600px;
  margin: 40px auto;
}
</style>

