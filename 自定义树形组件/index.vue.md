<template>
  <div class="group-container" :class="{'is-transfer': isTransfer}">
    <div class="group-header">全部分组</div>
    <div v-if="isGroup" class="add-group" @click="handleShowPopUp('新建分组', 0)"><span class="icon">+</span>新建分组</div>
    <div class="group-main">
      <Group :list="groupList" :is-group="isGroup" :is-transfer="isTransfer" @showPopUp="handleShowPopUp" @getList="getTableList" @checkGroup="checkGroup" />
    </div>
    <pop-up :title="popupTitle" :dialog-visible="dialogVisible" @confirmPopUp="confirmPopUp" @closePopUp="closePopUp">
      <el-input v-if="popupTitle !== '删除分组'" v-model="groupName" placeholder="请输入分组名称" />
      <div v-if="popupTitle === '删除分组'" class="delete-content">确定删除分组？</div>
    </pop-up>
  </div>
</template>
<script>
import Group from './Group.vue'
import PopUp from '@/components/PopUp'
import { getGroupList, addGroup, updateGroup, delGroup } from '@/api/group'
export default {
  components: { Group, PopUp },
  props: {
    isGroup: { // 设备分组页面才可进行操作
      type: Boolean,
      default: false
    },
    isTransfer: { // 转移页面样式有差异
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      dialogVisible: false,
      groupName: '',
      currentGroupId: 0,
      popupTitle: '新建分组',
      groupList: []
    }
  },
  created() {
    this.getList()
  },
  methods: {
    async getList() {
      const { data } = await getGroupList()
      this.groupList = data.rows
      this.$store.dispatch('app/setDefaultGroupId', data.rows[0].id)
      if (this.isTransfer) {
        this.$emit('treeList', data.rows)
      } else {
        this.$emit('defaultGroupId', data.rows[0].id)
      }
    },
    getTableList(id) {
      this.$emit('getTableList', id)
    },
    checkGroup(id, name) {
      this.$emit('checkGroup', id, name)
    },
    confirmPopUp() {
      switch (this.popupTitle) {
        case '新建分组':
        case '创建下级':
          this.addGroup()
          break
        case '修改组名':
          this.updateGroupName()
          break
        case '删除分组':
          this.delGroup()
          break
      }
      this.groupName = ''
      this.dialogVisible = false
    },
    closePopUp() {
      this.groupName = ''
      this.dialogVisible = false
    },
    handleShowPopUp(name, id) { // 新建、删除、修改组名
      this.popupTitle = name
      this.currentGroupId = id
      this.dialogVisible = true
    },
    async addGroup() {
      try {
        await addGroup({ groupName: this.groupName, parentId: this.currentGroupId })
        this.$toast({
          title: '创建成功',
          content: `新建分组成功！`
        })
        this.getList()
      } catch (error) {
        this.$toast({
          title: '创建失败',
          content: error,
          icon: 'error'
        })
      }
    },
    async updateGroupName() {
      try {
        await updateGroup({ groupName: this.groupName, groupId: this.currentGroupId })
        this.$toast({
          title: '更新成功',
          content: `更新分组名成功！`
        })
        this.getList()
      } catch (error) {
        this.$toast({
          title: '更新失败',
          content: error,
          icon: 'error'
        })
      }
    },
    async delGroup() {
      try {
        await delGroup({ groupId: this.currentGroupId })
        this.$toast({
          title: '删除成功',
          content: `删除分组成功！`
        })
        this.getList()
      } catch (error) {
        this.$toast({
          title: '删除失败',
          content: error,
          icon: 'error'
        })
      }
    }
  }
}
</script>
<style lang="scss" scoped>
.group-container {
  width: 100%;
  font-size: 14px;
  &.is-transfer {
    padding-right: 10px;
    .group-header {
        height: 40px;
        line-height: 40px;
        border-color: #F2F6FA;
    }
  }
  .group-header {
    color: #8094AC;
    height: 25px;
    border-bottom: 1px solid #8094AC;
  }
  .add-group {
    height: 30px;
    line-height: 30px;
    cursor: pointer;
    .icon {
      margin-right: 12px;
      padding-left: 4px;
    }
  }
  .group-main {
    margin-left: -23px;
  }
}
</style>
