<template>
  <div class="groups">
    <div
      v-for="(item, index) in list"
      :key="index"
      class="group-item-container"
    >
      <div
        class="group-item"
        :class="{ 'active-item': activeIndex == item.id }"
        @click.stop="checkGroup(item.id, item.label)"
        @mouseenter="showEllipsisIndex = item.id"
        @mouseleave="showEllipsisIndex = -1"
      >
        <div
          class="group-item-left"
          @mouseenter="showActionBoxIndex === item.id ? showEllipsisIndex = item.id : ''"
        >
          <div class="item-left-icon" @click.stop="showChildren(item.id)">
            <i
              v-if="openList.indexOf(item.id) > -1"
              class="el-icon-arrow-down mr12"
            />
            <i v-else class="el-icon-arrow-right mr12" />
          </div>
          <span class="ellipsis">{{ item.label }}</span>
          <div v-show="showEllipsisIndex == item.id" class="full-name">
            {{ item.label }}
          </div>
        </div>
        <div class="group-item-right">
          <div class="group-item-count">{{ item.groupDevNum }}</div>
          <div
            v-if="isGroup && showEllipsisIndex == item.id"
            class="ellipsis-box"
            @click.stop="showActionBoxIndex = item.id"
          >
            <img
              class="ellipsis"
              src="../../assets/images/ellipsis.png"
              alt=""
            >
          </div>
        </div>
        <div v-show="showActionBoxIndex == item.id" class="action-box">
          <div class="action-item" @click="showPopUp('创建下级', item.id)">
            创建下级
          </div>
          <div
            v-show="item.label !== '默认分组'"
            class="action-item"
            @click="showPopUp('修改组名', item.id)"
          >
            修改组名
          </div>
          <div
            v-show="item.label !== '默认分组'"
            class="action-item"
            @click="showPopUp('删除分组', item.id)"
          >
            删除分组
          </div>
        </div>
      </div>
      <group
        v-if="openList.indexOf(item.id) > -1"
        :list="item.children"
        :is-group="isGroup"
        @showPopUp="showPopUp"
        @showChildren="showChildren"
        @checkGroup="checkGroup"
      />
    </div>
  </div>
</template>
<script>
export default {
  name: 'Group',
  props: {
    list: { // 所有树节点
      type: Array,
      default: () => {
        return []
      }
    },
    isGroup: {
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
      activeIndex: 1, // 当前选中组
      openList: [], // 展开组
      showEllipsisIndex: -1, // 省略号显示
      showActionBoxIndex: -1 // 操作弹框
    }
  },
  created() {
    this.isTransfer && (this.activeIndex = -1)
  },
  methods: {
    // 选中
    checkGroup(id, name) {
      this.activeIndex = id
      this.showActionBoxIndex = -1
      this.$emit('getList', id)
      this.$emit('checkGroup', id, name)
    },
    // 显示下级
    showChildren(id) {
      this.showActionBoxIndex = -1
      const list = this.openList
      list.includes(id) ? list.splice(list.indexOf(id), 1) : list.push(id)
      this.$emit('showChildren', id)
    },
    showPopUp(name, id) {
      this.$emit('showPopUp', name, id)
    }
  }
}
</script>
<style lang="scss" scoped>
.is-transfer {
  .group-item {
    height: 40px;
    &:hover {
      color: inherit;
      background: rgba(0, 82, 217, 0.1);
    }
    &.active-item {
      background: rgba(0, 82, 217, 0.7);
      color: rgba(255, 255, 255, 0.9);
      // .mr12 {
      //   opacity: 0;
      // }
    }
  }
}
.ellipsis {
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
}
.item-left-icon {
  width: 20px;
  height: 100%;
}
.groups {
  margin-left: 23px;
  font-size: 14px;
}
.group-item {
  height: 30px;
  display: flex;
  padding-right: 9px;
  justify-content: space-between;
  align-items: center;
  border-radius: 3px;
  cursor: pointer;
  position: relative;
  &:hover {
    color: #6948fa;
    background: #f3f1ff;
  }
  &.active-item {
    color: #6948fa;
  }
  .group-item-left {
    display: flex;
    align-items: center;
    width: 86%;
    position: relative;
    .full-name {
      position: absolute;
      z-index: 20;
      bottom: -48px;
      left: 15px;
      height: 38px;
      line-height: 42px;
      padding: 0 7px;
      border-radius: 5px;
      color: rgba(255, 255, 255, 0.9);
      background: rgba(0, 0, 0, 0.9);
      box-shadow: 0px 3px 14px 2px rgba(0, 0, 0, 0.05), 0px 8px 10px 1px rgba(0, 0, 0, 0.06), 0px 5px 5px -3px rgba(0, 0, 0, 0.1);
      &::before {
        content: '';
        position: absolute;
        top: 0;
        left: 20px;
        display: block;
        width: 8px;
        height: 8px;
        background: inherit;
        transform: translateY(-50%) rotate(45deg);
      }
    }
  }
  .group-item-right {
    display: flex;
    align-items: center;
    .ellipsis-box {
      height: 30px;
      display: flex;
      align-items: center;
    }
    .ellipsis {
      width: 10px;
      height: 2px;
      margin-left: 12px;
    }
  }
  .group-item-count {
    color: rgba(0, 0, 0, 0.2);
    font-size: 12px;
  }
  .action-box {
    position: absolute;
    top: 28px;
    right: -130px;
    width: 150px;
    padding: 8px 0 8px 16px;
    background: #fff;
    border-radius: 2px;
    border: 1px solid #dcdcdc;
    z-index: 100;
    .action-item {
      line-height: 33px;
      color: rgba(0, 0, 0, 0.9);
    }
  }
}
.mr12 {
  font-size: 14px;
  margin-right: 12px;
}
</style>
