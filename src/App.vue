<script setup>
import { reactive, onBeforeMount } from 'vue';
import TaskContainer from './components/TaskContainer.vue';

let reactiveStorage = reactive({ myStorageCenter: {} });
function StorageCenter(obj) {
  this.userData = {
    taskItemData: [],
    groupItemData: [{ name: defaultGroupName }]
  };
  this.userStatus = {
    currentGroup: defaultGroupName,
    // focusingTask: null
  };
  Object.keys(obj).map(p => this[p] = obj[p]);
}
StorageCenter.prototype.addTask = function (taskItemObject) {
  this.userData.taskItemData.push(taskItemObject);
};
StorageCenter.prototype.addGroup = function (groupItem) {
  this.userData.groupItemData.push(groupItem);
};
StorageCenter.prototype.deleteTask = function (timestamp) {
  let index = reactiveStorage.myStorageCenter.userData.taskItemData.findIndex(e => e.createdAt === timestamp);
  reactiveStorage.myStorageCenter.userData.taskItemData.splice(index, 1);
};
StorageCenter.prototype.deleteGroup = function (groupName) {
  let index = reactiveStorage.myStorageCenter.userData.groupItemData.findIndex(e => e.name === groupName);
  reactiveStorage.myStorageCenter.userData.groupItemData.splice(index, 1);
  // 删除该分组内的所有事项
  reactiveStorage.myStorageCenter.userData.taskItemData = reactiveStorage.myStorageCenter.userData.taskItemData.filter(e => e.group !== groupName);
  if (reactiveStorage.myStorageCenter.userStatus.currentGroup === groupName) { // 如果被删的分组是活跃分组
    if (reactiveStorage.myStorageCenter.userData.groupItemData[index] !== undefined) { // 此时 index 指向被删的下一个元素，如果没有越界
      reactiveStorage.myStorageCenter.userStatus.currentGroup = reactiveStorage.myStorageCenter.userData.groupItemData[index].name;
    }
    else { // 如果越界，即活跃分组变为上一分组
      reactiveStorage.myStorageCenter.userStatus.currentGroup = reactiveStorage.myStorageCenter.userData.groupItemData[index - 1].name;
    }
  }
};
StorageCenter.prototype.renameGroup = function (oldName, newName) {
  // 修改分组对象的 name
  reactiveStorage.myStorageCenter.findgroupItem(oldName).name = newName;
  for (taskItem of reactiveStorage.myStorageCenter.userData.taskItemData) {
    if (taskItem.group === oldName) {
      taskItem.group = newName;
    }
  }
  // 如果重命名的分组是活跃的分组
  if (oldName === reactiveStorage.myStorageCenter.userStatus.currentGroup) {
    reactiveStorage.myStorageCenter.userStatus.currentGroup = newName;
  }
};
StorageCenter.prototype.findTaskItemObject = function (timestamp) {
  for (taskItem of this.userData.taskItemData) {
    if (taskItem.createdAt === timestamp) {
      return taskItem;
    }
  }
};
StorageCenter.prototype.findgroupItem = function (groupName) {
  for (groupItem of this.userData.groupItemData) {
    if (groupName === groupItem.name) {
      return groupItem;
    }
  }
};
// StorageCenter.prototype.focusOnTask = function (taskItemNode) {
//   let timestamp = Number(taskItemNode.getAttribute('timestamp'));
//   reactiveStorage.myStorageCenter.userStatus.focusingTask = timestamp;
//   taskItemNode.setAttribute('isFocusing', 'true');
// }

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
function UserStatus(obj) {
  // 用户状态对象的默认 property
  this.currentGroup = null;
  Object.keys(obj).map(p => this[p] = obj[p]);
}
const defaultGroupName = '默认分组';
const virtualGroupName = '全部事项';
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
  pauseToLocalStorage();
});
window.addEventListener('orientationchange', function () {
  let ElementsSwitchingGroupsInPortrait = document.querySelectorAll('[isSwitchingGroupsInPortrait="true"]'); // 当前是否处于竖屏时的切换分组状态下
  if (ElementsSwitchingGroupsInPortrait.length) {
    for (element of ElementsSwitchingGroupsInPortrait) {
      element.setAttribute('isSwitchingGroupsInPortrait', 'false');
    }
  }
})
document.body.addEventListener('input', (event) => {
  if (event.target.tagName === 'SELECT') {
    if (event.target.name === 'dropDownGroupOptions') {
      switch (event.target.value) {
        case 'rename': {
          let userInput = prompt('请输入新的分组名称');
          let flag = true; // 很C语言
          if (userInput !== null) { // 用户没点「取消」
            if (userInput !== '') {
              for (groupItem of reactiveStorage.myStorageCenter.userData.groupItemData) {
                if (groupItem.name === userInput) {
                  flag = false;
                  alert('已有名为' + userInput + '的分组');
                  break; // 很C语言
                }
              }
              if (flag) { // 很C语言
                let groupItemNode = event.target.closest('.groupItem');
                let oldName = groupItemNode.getAttribute('groupName');
                reactiveStorage.myStorageCenter.renameGroup(oldName, userInput);
              }
            }
            else { alert('分组名称不能为空') }
          }
        }
          break;
        case 'delete': {
          let groupItemNode = event.target.closest('.groupItem');
          let groupName = groupItemNode.getAttribute('groupName');
          let number = reactiveStorage.myStorageCenter.userData.taskItemData.filter(e => e.group === groupName).length;
          if (confirm(number ? "确认删除" + groupName + "分组？这将会删除该分组中共计" + number + "个事项。" : '确认删除分组？（该分组中无任务）')) {
            reactiveStorage.myStorageCenter.deleteGroup(groupName);
          }
        }
          break;
      }
    }
    event.target.value = 'edit' //让下拉菜单回到第一个option
  }
});
// 听说这是公交车？
  document.body.addEventListener('click', function (event) {
    if (event.target.id === 'expandGroups') {
      ['sidebar', 'right'].forEach(e => {
        document.getElementById(e).setAttribute('isSwitchingGroupsInPortrait', 'true');
      })
    }
    if (event.target.id === 'reset') {
      localStorage.clear();
      reactiveStorage.myStorageCenter = {};
      location.reload();//刷新
    }
    if (event.target.id === 'addTask') {
      let userInput = document.getElementById('addNewTask').value;
      if (userInput !== '') {
        let newTaskItem = new TaskItem({ text: userInput }); // 构造新任务的对象
        if (reactiveStorage.myStorageCenter.userStatus.currentGroup === virtualGroupName) { // 如果活跃分组是虚拟分组「全部事项」
          newTaskItem.group = reactiveStorage.myStorageCenter.userData.groupItemData[0].name; // 则新任务对象的分组property变为第一个分组
        }
        reactiveStorage.myStorageCenter.addTask(newTaskItem);
      }
      document.getElementById('addNewTask').value = '';
    }
    if (event.target.id === 'addGroup') {
      let userInput = prompt('请输入新分组的名称');
      let flag = true; // 很C语言
      if (userInput !== null) { // 用户没点「取消」
        if (userInput !== '') {
          reactiveStorage.myStorageCenter.userData.groupItemData.forEach(groupItem => {
            if (groupItem.name === userInput) {
              flag = false;
              alert('已有名为' + userInput + '的分组');
              return // blank return
            }
          });
          if (flag) { // 很C语言
            reactiveStorage.myStorageCenter.addGroup(new GroupItem({ name: userInput }));
          }
        }
        else { alert('分组名称不能为空') }
      }
    }
    if (event.target.classList.contains('deleteTask')) {
      let taskItemNode = event.target.closest('.taskItem');
      let timestamp = Number(taskItemNode.getAttribute('timestamp'));
      reactiveStorage.myStorageCenter.deleteTask(timestamp);
      // reactiveStorage.myStorageCenter.userStatus.focusingTask = null;
    }
    if (event.target.classList.contains('isDone')) {
      let taskItemNode = event.target.closest('.taskItem');
      let timestamp = Number(taskItemNode.getAttribute('timestamp'));
      let taskItem = reactiveStorage.myStorageCenter.findTaskItemObject(timestamp);
      taskItem.isDone ? taskItem.isDone = false : taskItem.isDone = true;
    }
    if (event.target.classList.contains('groupName')) {
      reactiveStorage.myStorageCenter.userStatus.currentGroup = event.target.textContent;
      // reactiveStorage.myStorageCenter.userStatus.focusingTask = null;
      if (window.matchMedia('(orientation: portrait)')) {
        document.querySelectorAll('[isSwitchingGroupsInPortrait="true"]').forEach(e => { e.setAttribute('isSwitchingGroupsInPortrait', 'false') })
      }
    }
    if (event.target.id === 'export') {
      prompt('', JSON.stringify(reactiveStorage.myStorageCenter.userData));
    }
    if (event.target.id === 'import') {
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
    if (event.target.closest('.taskItem') === null && document.querySelector('.taskItem[isFocusing="true"]')) { // 点击当前聚焦的任务之外的地方则关闭 taskOptions
      // reactiveStorage.myStorageCenter.userStatus.focusingTask = null;
      document.querySelector('.taskItem[isFocusing="true"]').setAttribute('isFocusing', '');
    }
    if (event.target.classList.contains('setDeadline')) {
      let taskItemNode = event.target.closest('.taskItem');
      let timestamp = Number(taskItemNode.getAttribute('timestamp'));
      let taskItem = reactiveStorage.myStorageCenter.findTaskItemObject(timestamp);
      taskItem.deadline === null ? taskItem.deadline = '' : taskItem.deadline = null;
    }
  })
function cb (){console.log('nested emits.')}
</script>

<template>
  <div :id="`sidebar`">
    <div>分组</div>
    <div :id="`groupList`" :remainingGroups="reactiveStorage.myStorageCenter.userData.groupItemData.length">
      <div>
        <div :class="`groupItem`" :id="`allTasks`">
          <div :class="`groupName`">全部事项</div>
          <div :class="`groupTrailing`">
            <div :class="`taskNumber`">{{ reactiveStorage.myStorageCenter.userData.taskItemData.filter(e =>
                !e.isDone).length
            }}</div>
          </div>
        </div>
        <div :class="`groupItem`" :id="`toDeadline`">
          <div :class="`groupName`">将要截止</div>
          <div :class="`groupTrailing`">
            <div :class="`taskNumber`">{{ reactiveStorage.myStorageCenter.userData.taskItemData.filter(e => e.deadline
                &&
                !e.isDone).length
            }}</div>
          </div>
        </div>
      </div>
      <template v-for="groupItem in reactiveStorage.myStorageCenter.userData.groupItemData">
        <div :class="`groupItem`" :groupname="groupItem.name">
          <div :class="`groupName`">{{ groupItem.name }}</div>
          <div :class="`groupTrailing`">
            <div :class="`taskNumber`">{{ reactiveStorage.myStorageCenter.userData.taskItemData.filter(e => e.group ===
                groupItem.name && !e.isDone).length
            }}</div>
            <select :class="`groupOptions`" :name="`dropDownGroupOptions`">
              <option :value="`edit`">编辑</option>
              <option :class="`renameGroup`" :value="`rename`">重命名</option>
              <option :class="`deleteGroup`" :value="`delete`">删除</option>
            </select>
          </div>
        </div>
      </template>
    </div>
    <button :id="`addGroup`">添加分组</button>
    <div :id="`generalOptions`">
      <button :id="`reset`">抹掉数据</button>
      <button :id="`export`">导出数据</button>
      <button :id="`import`">导入数据</button>
    </div>
  </div>
  <div :id="`right`">
    <div :id="`topBar`">
      <button :id="`expandGroups`">展开分组</button>
      <div>{{ reactiveStorage.myStorageCenter.userStatus.currentGroup }}</div>
      <!--当前分组的标题-->
    </div>
    <TaskContainer v-if="reactiveStorage.myStorageCenter.userStatus.currentGroup === '全部事项'"
      :currentList="reactiveStorage.myStorageCenter.userData.taskItemData" @response="cb"/>
    <TaskContainer v-else-if="true"
      :currentList="reactiveStorage.myStorageCenter.userData.taskItemData.filter((e) => e.group === reactiveStorage.myStorageCenter.userStatus.currentGroup)" />
    <div :id="`underContainer`">
      <!-- 紧贴在所有事项的底部，末尾总是有的输入框 -->
      <input :type="`text`" :class="`taskBox-inputTag`" :id="`addNewTask`">
      <div :class="`taskTrailing`">
        <button :id="`addTask`">添加</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
</style>
