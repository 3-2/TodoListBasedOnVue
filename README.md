# Root Component 结构

由 Visual Studio Code 折叠代码而来

```Vue
<script setup>
import { reactive, onBeforeMount, ref, computed } from 'vue';
import { Setting, Plus } from '@element-plus/icons-vue'
import TaskContainer from './components/TaskContainer.vue';
import GroupList from './components/groupList.vue'

const GroupName_defaultGroup = '默认分组';
const GroupName_allTasks = '全部事项';
const GroupName_TasksWithDeadline = '将要截止'

// 数据结构
let reactiveStorage = reactive({ myStorageCenter: {} });
function StorageCenter(obj) {
}
StorageCenter.prototype.addTask = function (taskItemObject) {
};
StorageCenter.prototype.addGroup = function (groupItem) {
};
StorageCenter.prototype.renameGroup = function (oldName, newName) {
};
StorageCenter.prototype.isExistedGroupName = function (userInput) {
}
function pauseToLocalStorage() {
}
function resumeFromLocalStorage() {
}
function TaskItem(data) {
}
function GroupItem(data) {
}
// Reactivity
let isSwitchingGroupsInPortrait = computed(() => {
})

// DOM 引用
const addTaskButton = ref();
const TextBoxToAddNewTask = ref()

// 业务：启动网页
onBeforeMount(() => {
})
// 业务：关闭网页
window.addEventListener('pagehide', function () { // 仅在关闭网页时备份
});
// 业务：横屏
window.addEventListener('orientationchange', function () {
})
// 业务：抹掉数据
function format() {
}
// 业务：导入
function exportData() { prompt('', JSON.stringify(reactiveStorage.myStorageCenter.userData)) }
// 业务：导出
function importData() {
}

// 事件 handler
document.body.addEventListener('click', function (event) {
})
// v-on handler
function clickAddGroup() {
}
function expandGroups() {
}
function clickBlankBelowSidebar() {
}
function clickAddTask() {
}
// Element Plus
const dropdown1 = ref()
function showClick() {
}

</script>

<template>
  <!-- 左侧分组 -->
  <div :id="`sidebar`" :isSwitchingGroupsInPortrait="isSwitchingGroupsInPortrait" @click="clickSidebar">
    <!-- Child component：分组列表 -->
    <GroupList :groupItemArray="reactiveStorage.myStorageCenter.userData.groupItemData"
      :storageCenter="reactiveStorage.myStorageCenter" />
    <!-- 分组列表底部 -->
    <div :id="`blankBelowGroupList`" @click="clickBlankBelowSidebar"></div>
    <div :id="`underGroupList`">
    </div>

  </div>
  <!-- 右侧任务清单 -->
  <div :id="`right`" :isSwitchingGroupsInPortrait="isSwitchingGroupsInPortrait">
    <!-- 清单顶部 -->
    <div :id="`topBar`">
    </div>
    <!-- 清单中部 -->
    <div :id="`container`">
    </div>
    <!-- 清单底部 -->
    <div :id="`blankBelowTaskList`"></div>
    <div :id="`underContainer`">
    </div>
  </div>
</template>

<style scoped>
</style>```
