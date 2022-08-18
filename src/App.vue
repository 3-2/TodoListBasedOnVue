<script setup>
import { reactive, onBeforeMount, ref, computed } from 'vue';
import { Setting, Memo, Plus } from '@element-plus/icons-vue'

import TaskContainer from './components/TaskContainer.vue';
import GroupList from './components/groupList.vue'
let reactiveStorage = reactive({ myStorageCenter: {} });
const GroupName_defaultGroup = '默认分组';
const GroupName_allTasks = '全部事项';
const GroupName_TasksWithDeadline = '将要截止'
let isSwitchingGroupsInPortrait = computed(() => { return reactiveStorage.myStorageCenter.userStatus.isSwitchingGroupsInPortrait })

function StorageCenter(obj) {
  this.userData = {
    taskItemData: [],
    groupItemData: [{ name: GroupName_defaultGroup }]
  };
  this.userStatus = {
    currentGroup: GroupName_defaultGroup,
    isSwitchingGroupsInPortrait: false,
  };
  Object.keys(obj).map(p => this[p] = obj[p]);
}
StorageCenter.prototype.addTask = function (taskItemObject) {
  this.userData.taskItemData.push(taskItemObject);
};
StorageCenter.prototype.addGroup = function (groupItem) {
  this.userData.groupItemData.push(groupItem);
};
StorageCenter.prototype.renameGroup = function (oldName, newName) {
  // 修改分组对象的 name
  reactiveStorage.myStorageCenter.userData.groupItemData.find(groupItem => oldName === groupItem.name).name = newName;
  reactiveStorage.myStorageCenter.userData.taskItemData.forEach(taskItem => {
    if (taskItem.group === oldName) {
      taskItem.group = newName;
    }
  })
  // 如果重命名的分组是活跃的分组
  if (oldName === reactiveStorage.myStorageCenter.userStatus.currentGroup) {
    reactiveStorage.myStorageCenter.userStatus.currentGroup = newName;
  }
};
StorageCenter.prototype.isExistedGroupName = function (userInput) { let returnValue = false; reactiveStorage.myStorageCenter.userData.groupItemData.forEach(groupItem => { if (groupItem.name === userInput) returnValue = true; }); return returnValue; }
function pauseToLocalStorage() {
  localStorage.setItem('pausedData', JSON.stringify(reactiveStorage.myStorageCenter));
}
function resumeFromLocalStorage() {
  reactiveStorage.myStorageCenter = new StorageCenter(JSON.parse(localStorage.getItem('pausedData')));
}
function TaskItem(data) {
  // 事项对象的默认 property
  this.text = '空事项';
  this.isDone = false;
  this.createdAt = new Date().getTime();
  this.group = reactiveStorage.myStorageCenter.userStatus.currentGroup;
  this.deadline = null;
  Object.keys(data).map(p => this[p] = data[p]);
}
function GroupItem(data) {
  // 分组对象的默认 property
  this.name = '未命名分组';
  Object.keys(data).map(p => this[p] = data[p]);
}
const addTaskButton = ref();
const TextBoxToAddNewTask = ref()

// 业务：启动网页
onBeforeMount(() => {
  if (localStorage.getItem('pausedData') === null || localStorage.getItem('pausedData') === '{}') { // 全新启动的判断由 == null 换成了 == {}
    reactiveStorage.myStorageCenter = new StorageCenter({});
  }
  else {
    resumeFromLocalStorage();
  }
})
// 业务：关闭网页
window.addEventListener('pagehide', function () { // 仅在关闭网页时备份
  reactiveStorage.myStorageCenter.userData.taskItemData.forEach(taskItem => { if (taskItem.deadline === '') taskItem.deadline = null })
  pauseToLocalStorage();
});
window.addEventListener('orientationchange', function () {
  if (isSwitchingGroupsInPortrait) reactiveStorage.myStorageCenter.userStatus.isSwitchingGroupsInPortrait = false;
})
function format() {
  localStorage.clear();
  reactiveStorage.myStorageCenter = {};
  location.reload();//刷新
}
function exportData() { prompt('', JSON.stringify(reactiveStorage.myStorageCenter.userData)) }
function importData() {
  if (confirm('确定导入数据？警告：这会覆盖当前已有的数据。')) {
    try {
      let imported = JSON.parse(prompt());
      if (imported !== null) { // 用户没点「取消」
        reactiveStorage.myStorageCenter = new StorageCenter({ userData: imported });
        reactiveStorage.myStorageCenter.userStatus.currentGroup = imported.groupItemData[0].name;
      }
    }
    catch (e) {
      alert('输入格式不正确，请检查确认后重试');
    }
  }
}

document.body.addEventListener('click', function (event) {
  if (event.target.closest('.taskItem') === null) { // 点击当前聚焦的任务之外的地方则关闭 taskOptions
    let isExsistedFocusingTask = document.querySelector('[displayTaskOptions]');
    if (isExsistedFocusingTask) {
      isExsistedFocusingTask.removeAttribute('displayTaskOptions');
      reactiveStorage.myStorageCenter.userData.taskItemData.forEach(taskItem => { if (taskItem.isSetting) taskItem.isSetting = false })
    }
  }
})
function clickAddGroup() {
  let userInput = prompt('请输入新分组的名称');
  if (userInput) {
    if (reactiveStorage.myStorageCenter.isExistedGroupName(userInput)) {
      alert('已存在名为' + userInput + '的分组。')
    } else {
      reactiveStorage.myStorageCenter.addGroup(new GroupItem({ name: userInput }));
      reactiveStorage.myStorageCenter.userStatus.currentGroup = userInput
    }
  }
}
function expandGroups() {
  reactiveStorage.myStorageCenter.userStatus.isSwitchingGroupsInPortrait = true;
}
function clickBlankBelowSidebar() {
  if (isSwitchingGroupsInPortrait) reactiveStorage.myStorageCenter.userStatus.isSwitchingGroupsInPortrait = false
}
function clickAddTask() {
  let userInput = TextBoxToAddNewTask.value.value;
  if (userInput !== '') {
    let newTaskItem = new TaskItem({ text: userInput }); // 构造新任务的对象
    if (reactiveStorage.myStorageCenter.userStatus.currentGroup === GroupName_allTasks) { // 如果活跃分组是虚拟分组「全部事项」
      newTaskItem.group = reactiveStorage.myStorageCenter.userData.groupItemData[0].name; // 则新任务对象的分组property变为第一个分组
    }
    reactiveStorage.myStorageCenter.addTask(newTaskItem);
  }
  TextBoxToAddNewTask.value.value = '';
}
const dropdown1 = ref()
function showClick() {
  dropdown1.value.handleOpen()
}

</script>

<template>
  <div :id="`sidebar`" :isSwitchingGroupsInPortrait="isSwitchingGroupsInPortrait" @click="clickSidebar">
    <!-- <div>分组</div> -->
    <GroupList :groupItemArray="reactiveStorage.myStorageCenter.userData.groupItemData"
      :storageCenter="reactiveStorage.myStorageCenter" />
    <div :id="`blankBelowGroupList`" @click="clickBlankBelowSidebar"></div>
    <div :id="`underGroupList`">
      <el-button :id="`addGroup`" type="primary" plain @click="clickAddGroup" :icon="Plus">
        添加分组
      </el-button>
      <el-button type="primary" plain @click="showClick" :icon="Setting">
      </el-button>
      <el-dropdown ref="dropdown1" trigger="contextmenu">
        <span class="el-dropdown-link"></span>
        <template #dropdown>
          <el-dropdown-menu>
            <el-dropdown-item @click="format">抹掉数据</el-dropdown-item>
            <el-dropdown-item @click="exportData">导出数据</el-dropdown-item>
            <el-dropdown-item @click="importData">导入数据</el-dropdown-item>
          </el-dropdown-menu>
        </template>
      </el-dropdown>
    </div>

  </div>
  <div :id="`right`" :isSwitchingGroupsInPortrait="isSwitchingGroupsInPortrait">
    <div :id="`topBar`">
      <div :id="`currentGorupName`">{{ reactiveStorage.myStorageCenter.userStatus.currentGroup }}</div>
      <el-button :id="`expandGroups`" ref="addTaskButton" plain @click="expandGroups" icon="Memo">
      </el-button>
      <!--当前分组的标题-->
    </div>
    <div :id="`container`">
      <TaskContainer v-if="reactiveStorage.myStorageCenter.userStatus.currentGroup === GroupName_allTasks"
        :currentList="reactiveStorage.myStorageCenter.userData.taskItemData"
        :storageCenter="reactiveStorage.myStorageCenter" />
      <TaskContainer v-else-if="reactiveStorage.myStorageCenter.userStatus.currentGroup === GroupName_TasksWithDeadline"
        :currentList="reactiveStorage.myStorageCenter.userData.taskItemData.filter(e => e.deadline)"
        :storageCenter="reactiveStorage.myStorageCenter" />
      <TaskContainer v-else-if="true"
        :currentList="reactiveStorage.myStorageCenter.userData.taskItemData.filter(e => e.group === reactiveStorage.myStorageCenter.userStatus.currentGroup)"
        :storageCenter="reactiveStorage.myStorageCenter" />
    </div>
    <div :id="`blankBelowTaskList`"></div>
    <div :id="`underContainer`">
      <!-- 紧贴在所有事项的底部，末尾总是有的输入框 -->
      <input :type="`text`" :class="`taskBox-inputTag`" :id="`addNewTask`" ref="TextBoxToAddNewTask"
        :placeholder="`添加任务`" @keyup.enter.native="clickAddTask">
      <div :class="`taskTrailing`">
        <el-button :id="`addTask`" ref="addTaskButton" type="primary" plain @click="clickAddTask" :icon="Plus">
        </el-button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.example-showcase .el-dropdown-link {
  cursor: pointer;
  color: var(--el-color-primary);
  display: flex;
  align-items: center;
}
</style>
