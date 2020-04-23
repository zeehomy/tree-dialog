<template>
  <div id="app">
    <el-tree :data="data"
      ref="tree"
      node-key="id"
      default-expand-all
      :props="defaultProps"
      :expand-on-click-node="false"
      :render-content="renderContent"
    ></el-tree>
    <el-dialog
      title="编辑名称"
      :visible.sync="dialogVisible"
      center>
      <el-input v-model="labelInput" placeholder="请输入内容"></el-input>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="append">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
// node-key="id"
let id = 1000;
export default {
  name: 'app',
  data() {
    return {
      dialogVisible: false,
      labelInput: '',
      currentNode: null,
      currentDataItem: null,
      data: [
        {
          id: 1,
          label: '1级 1',
          children: []
        }
      ],
      defaultProps: {
        children: 'children',
        label: 'label'
      }
    };
  },
  methods: {
    // 
    showDialog(node, data) {
      this.dialogVisible = true;
      this.labelInput = `${node.level + 1}级 ${data.children.length + 1}`;

      this.currentNode = node;
      this.currentDataItem = data;
    },
    append() {
      if(this.labelInput) {

        const newChild = { 
          id: id++,
          label: this.labelInput,
          children: []
        };
        if (!this.currentDataItem.children) {
          this.$set(this.currentDataItem, 'children', []);
        }
        this.currentDataItem.children.push(newChild);
  
        this.dialogVisible = false;
      }

    },
    remove(node, data) {
      const parent = node.parent;
      const children = parent.data.children;
      const index = children.findIndex(item => item.id === data.id);
      children.splice(index, 1);
    },
    move(node, data, direction) {

      const parent = node.parent;
      const children = parent.data.children;
      const index = children.findIndex(item => item.id === data.id);

      this.$refs.tree.remove(data);
      if (direction < 0) {
        this.$refs.tree.insertBefore(data, children[index - 1]);
      } else {
        this.$refs.tree.insertAfter(data, children[index]);
      }
    },
    // eslint-disable-next-line no-unused-vars
    renderContent(h, { node, data, store }) {
      return (
        <span class="custom-tree-node">
          <span>{node.label}</span>
          <span class="tree-button-wrapper">
            { node.level > 1 && node.parent.data.children.findIndex(item => item.id === data.id) > 0 ?
              <el-button type="text"
                on-click={ () => this.move(node, data, -1) }
              >上移</el-button> : null
            }
            { node.level > 1 && node.parent.data.children.findIndex(item => item.id === data.id) < node.parent.data.children.length - 1 ?
              <el-button type="text"
                on-click={ () => this.move(node, data, 1) }
              >下移</el-button> : null
            }
            { node.level <= 2 ?
              <el-button type="text"
                on-click={ () => this.showDialog(node, data) }
              >添加下一级</el-button> : null
            }
            { node.level > 1 ?
              <el-button type="text"
                on-click={ () => this.remove(node, data) }
              >删除</el-button> : null
            }
          </span>
        </span>
      );
    }
  }
}
</script>

<style lang="less">
.custom-tree-node {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 14px;
  padding-right: 8px;
}
.tree-button-wrapper {
  display: none;
}
.el-tree-node__content:hover {

  .tree-button-wrapper {
    display: inline;
  }
}
</style>
