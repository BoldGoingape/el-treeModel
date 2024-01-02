<template>
  <el-row>
    <el-col :span="12"
      ><div class="grid-content ep-bg-purple" />
      <el-tree
        :props="props"
        :load="loadNode1"
        lazy
        node-key="id"
        show-checkbox
        @node-expand="tree1NodeExpand"
        @check="tree1Check"
        @node-collapse="nodeCollapse"
        ref="tree1"
      />
    </el-col>
    <el-col :span="12"
      ><div class="grid-content ep-bg-purple-light" />
      <el-tree
        :props="props"
        :load="loadNode2"
        lazy
        show-checkbox
        ref="tree2"
        node-key="id"
        :default-expanded-keys="defaultexpandedkeys2"
        :default-checked-keys="defaultcheckedkeys2"
        :filter-node-method="filterNode"
      />
    </el-col>
  </el-row>
</template>

<script setup>
import { ref, onMounted } from "vue";

const props = {
  label: "name",
  children: "zones",
  isLeaf: "leaf"
};

const tree1 = ref();
const tree2 = ref();

// 树2默认展开
const defaultexpandedkeys2 = ref();
// 树2默认选择
const defaultcheckedkeys2 = ref();

// 树节点过滤时保留的数组
const filterarr = [];

// 收起树1节点时，同步收起树2相同节点
const nodeCollapse = data => {
  let nodeKey = tree2.value.getNode(data.id);
  //收起节点方法
  nodeKey.collapse();
};

// 树1的展开方法
const tree1NodeExpand = data => {
  // 把绑定的node-key赋值给树2默认展开，达到1·2同时展开效果
  defaultexpandedkeys2.value = [data.id];
};

// 树1展开时，同步维护树2的过滤的方法
const tree1NodeExpandArrangeFilterarr = nowNode => {
  //若当前节点拥有子节点，把当前节点的子节点数据，添加到fillerarr中，维护树2筛选
  if (nowNode.childNodes && nowNode.childNodes.length > 0) {
    const arr = [];
    getchildData(nowNode, arr);

    arr.forEach(ele => {
      if (!filterarr.includes(ele)) {
        filterarr.push(ele);
      }
    });
  }
};

// 树1的复选框选中方法
const tree1Check = data => {
  // 获取当前节点
  let nowNode = tree1.value.getNode(data.id);
  const arr = [];

  if (nowNode.checked) {
    // 当前节点被选中（勾选时的方法），当前节点下的所有子节点应同样被选中，此方法把被选中的节点id添加到filterarr中维护树2筛选
    getchildData(nowNode, arr);
    arr.forEach(ele => {
      if (!filterarr.includes(ele)) {
        filterarr.push(ele);
      }
    });
  } else {
    /* 当前节点被取消选中（取消勾选时的方法），当前节点下的所有子节点应同样被取消选中，
     * 此方法把被选中的节点的所有子节点的id从filterarr中删除维护树2筛选，
     * 并且把被选中的节点的所有父节点的id添加到filterarr中维护树2筛选
     */
    getchildData(nowNode, arr);

    const pararr = [];

    getParentData(nowNode, pararr);
    arr.forEach(ele => {
      filterarr.splice(
        filterarr.findIndex(item => item === ele),
        1
      );
    });

    pararr.forEach(ele => {
      filterarr.splice(
        filterarr.findIndex(item => item === ele),
        1
      );
    });
  }
  // 调用树2过滤
  tree2.value.filter(data.id);

  // 当过滤完成时，把树1的选中（包含未选中）状态，同步到树2中
  arr.forEach(ele => {
    let tree1Node = tree1.value.getNode(ele);
    // 改变树2的选中状态
    tree2.value.setChecked(ele, tree1Node.checked);
  });
};

// 获取当前节点的 id，与当前节点下子节点的所有id 返回到arr中
const getchildData = (node, arr) => {
  // push前检查数组，避免重复push
  if (!arr.includes(node.data.id)) {
    arr.push(node.data.id);
  }

  if (node.childNodes && node.childNodes.length > 0) {
    node.childNodes.forEach(element => {
      getchildData(element, arr);
    });
  }
};

// 获取当前节点的 id，与当前节点的所有父节点的id 返回到arr中
const getParentData = (node, arr) => {
  if (
    node.parent &&
    !node.parent.checked &&
    !node.parent.indeterminate &&
    node.parent.data &&
    node.parent.data.id
  ) {
    if (!arr.includes(node.parent.data.id)) {
      arr.push(node.parent.data.id);
    }
    getParentData(node.parent, arr);
  }
};

// 过滤节点的方法，当前节点（data）的id 在filterarr中存在，才在树2中展示
const filterNode = (value, data) => {
  // 传空时默认全不显示
  if (!value) return false;

  return filterarr.includes(data.id);
};

// 懒加载树1
const loadNode1 = (node, resolve) => {
  if (node.level === 0) {
    return resolve([{ name: "region", id: "1" }]);
  }
  if (node.level > 1) return resolve([]);

  setTimeout(() => {
    const data = [
      {
        name: "leaf",
        leaf: true,
        id: "1-1"
      },
      {
        name: "zone",
        id: "1-2"
      }
    ];

    resolve(data);

    // 一定要在resolve(data)后执行，否则树2筛选维护会取消选中当前节点
    // 若当前节点被选中后调用懒加载，则将把加载完成的当前节点的子节点数据，添加到fillerarr中，维护树2筛选
    // 获取树1的当前节点
    let nowNode = tree1.value.getNode(node.data.id);
    if (nowNode.checked) {
      tree1NodeExpandArrangeFilterarr(nowNode);
    }
  }, 500);
};

// 懒加载树2
const loadNode2 = (node, resolve) => {
  if (node.level === 0) {
    return resolve([{ name: "region", id: "1" }]);
  }
  if (node.level > 1) return resolve([]);

  setTimeout(() => {
    const data = [
      {
        name: "leaf",
        leaf: true,
        id: "1-1"
      },
      {
        name: "zone",
        id: "1-2"
      }
    ];

    resolve(data);
  }, 500);
};

onMounted(() => {
  // 页面加载时调用树过滤让右侧树置空
  tree2.value.filter();
});
</script>
