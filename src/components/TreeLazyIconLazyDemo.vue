<script setup>
import { ref } from 'vue'

// 初始化时就存在的多级树节点数据，并全部展开
// 其中部分节点支持后续懒加载（hasLazy: true）
const treeData = ref([
  {
    id: 1,
    label: '节点 1',
    hasLazy: true,
    children: [
      {
        id: 11,
        label: '节点 1-1（可继续懒加载）',
        hasLazy: true,
        // 初始没有 children，但需要显示展开图标，所以显式标记为非叶子
        isLeaf: false,
      },
      {
        id: 12,
        label: '节点 1-2',
        children: [
          { id: 121, label: '节点 1-2-1' },
          { id: 122, label: '节点 1-2-2' },
        ],
      },
    ],
  },
  {
    id: 2,
    label: '节点 2（本身有子节点，也可以再懒加载）',
    hasLazy: true,
    children: [
      { id: 21, label: '节点 2-1' },
      {
        id: 22,
        label: '节点 2-2（可继续懒加载）',
        hasLazy: true,
        children: [{ id: 221, label: '节点 2-2-1' }],
      },
    ],
  },
  {
    id: 3,
    label: '节点 3',
    children: [
      { id: 31, label: '节点 3-1' },
      { id: 32, label: '节点 3-2' },
    ],
  },
])

// el-tree 实例
const treeRef = ref(null)

// el-tree 的 props 配置
const defaultProps = {
  children: 'children',
  label: 'label',
  // 使用节点上的 isLeaf 字段来控制是否显示展开图标
  isLeaf: 'isLeaf',
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

// 点击节点（仅触发点击事件，不做懒加载）
const handleNodeClick = (data) => {
  // 这里可以处理节点点击，如右侧详情、高亮等
  console.log('node click:', data)
}

// 展开节点时，按需懒加载其子节点（通过点击展开图标触发）
const handleNodeExpand = async (data) => {
  // 没有开启懒加载标记，或者已经加载过子节点，则不再处理
  if (!data.hasLazy || data.childrenLoaded) return

  // 标记为已加载，防止重复请求
  data.childrenLoaded = true

  const children = await loadChildren(data)

  if (!children?.length || !treeRef.value) return

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
      :expand-on-click-node="false"
      @node-click="handleNodeClick"
      @node-expand="handleNodeExpand"
    />
  </div>
</template>

<style scoped>
.tree-wrapper {
  max-width: 600px;
  margin: 40px auto;
}
</style>

