<script setup>
import { ref } from 'vue'

/**
 * 说明：
 * - 使用 Element Plus 原生懒加载能力：`lazy` + `load` 属性。
 * - 初始化时根节点数据固定，但子节点统一通过懒加载获取。
 * - 通过 `default-expanded-keys` 指定若干节点在初始化后默认展开，
 *   Element Plus 会自动触发这些节点的懒加载。
 */

// 根节点（第一层）静态数据，其余子节点通过懒加载获取
const rootNodes = [
  { id: 1, label: '根节点 1' },
  { id: 2, label: '根节点 2' },
  { id: 3, label: '根节点 3' },
]

// el-tree 绑定的数据（懒加载模式下通常只需要提供根节点）
const treeData = ref(rootNodes)

// 需要在初始化时默认展开的节点 key 列表
// 这里示例：展开 id 为 1 和 2 的根节点
const defaultExpandedKeys = ref([1, 2])

// el-tree 实例引用（如果后续需要手动操作树，可用）
const treeRef = ref(null)

// el-tree 的 props 配置
const defaultProps = {
  label: 'label',
  children: 'children',
  isLeaf: 'isLeaf',
}

// 模拟一个异步接口，根据节点生成子节点数据
const mockFetchChildren = (nodeData, level) => {
  return new Promise((resolve) => {
    setTimeout(() => {
      const base = Date.now()

      // 简单根据层级控制是否还有子节点
      const hasMoreChildren = level < 3

      const children = [
        {
          id: base,
          label: `${nodeData.label} - 子节点 A`,
          // 如果还能继续懒加载，则不标记 isLeaf
          ...(hasMoreChildren ? {} : { isLeaf: true }),
        },
        {
          id: base + 1,
          label: `${nodeData.label} - 子节点 B`,
          ...(hasMoreChildren ? {} : { isLeaf: true }),
        },
      ]

      resolve(children)
    }, 500)
  })
}

/**
 * 懒加载回调
 * @param node 当前节点（内部节点对象，包含 level、data 等）
 * @param resolve 用于将懒加载结果返回给 el-tree 的回调
 */
const loadNode = async (node, resolve) => {
  // level === 0 表示“根”的虚拟节点，此时需要返回第一层根列表
  if (node.level === 0) {
    return resolve(treeData.value)
  }

  const nodeData = node.data
  const level = node.level

  const children = await mockFetchChildren(nodeData, level)
  resolve(children)
}

// 节点点击（不影响懒加载，仅示例）
const handleNodeClick = (data, node) => {
  console.log('node click:', data, node)
}
</script>

<template>
  <div class="tree-wrapper">
    <el-tree
      ref="treeRef"
      :data="treeData"
      node-key="id"
      :props="defaultProps"
      lazy
      :load="loadNode"
      :default-expanded-keys="defaultExpandedKeys"
      :expand-on-click-node="false"
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

