<script setup>
import { ref } from 'vue'
import { Operation, Clock } from '@element-plus/icons-vue'

const props = defineProps({
    groupItem: Object,
    storageCenter: Object
})
function renameGroup() {
    let userInput = prompt('请输入新的分组名称');
    if (userInput !== null) { // 用户没点「取消」
        if (userInput !== '') {
            if (props.storageCenter.isExistedGroupName(userInput)) {
                alert('已有名为' + userInput + '的分组');
            }
            else { // 很C语言
                props.storageCenter.renameGroup(props.groupItem.name, userInput);
            }
        }
        else { alert('分组名称不能为空') }
    }
}
function deleteGroup() {
    let number = props.storageCenter.userData.taskItemData.filter(e => e.group === props.groupItem.name).length;
    if (confirm(number ? "确认删除" + props.groupItem.name + "分组？这将会删除该分组中共计" + number + "个事项。" : '确认删除分组' + props.groupItem.name + '？（该分组中无任务）')) {
        let index = props.storageCenter.userData.groupItemData.indexOf(props.groupItem);
        props.storageCenter.userData.groupItemData.splice(index, 1);
        if (props.groupItem.name === props.storageCenter.userStatus.currentGroup) {
            if (props.storageCenter.userData.groupItemData[index] !== undefined) {
                props.storageCenter.userStatus.currentGroup = props.storageCenter.userData.groupItemData[index].name;
            }
            else {
                props.storageCenter.userStatus.currentGroup = props.storageCenter.userData.groupItemData[index - 1].name;
            }
        }
    }
}
function clickGroupItem() {
    // event.stopPropagation();
    props.storageCenter.userStatus.currentGroup = props.groupItem.name;
    if (window.matchMedia('(orientation: portrait)')) {
        props.storageCenter.userStatus.isSwitchingGroupsInPortrait = false;
    }
}
const dropdown1 = ref()
function showClick(event) {
    event.stopPropagation();
    dropdown1.value.handleOpen()
}
</script>
<template>
    <div :class="`groupItem`" :groupname="groupItem.name" @click="clickGroupItem" :activeGroup="storageCenter.userStatus.currentGroup === groupItem.name">
        <el-icon v-if="groupItem.name === '将要截止'" style="margin: auto"><Clock /></el-icon>
        <el-icon v-else-if="groupItem.name === '全部事项'" style="margin: auto"><Memo /></el-icon>
        <el-button v-else @click="showClick" style="padding:0; border:none;background-color: transparent;color:black" size="large" :icon="Operation">
        </el-button>
        <el-dropdown ref="dropdown1" trigger="contextmenu">
            <span class="el-dropdown-link"></span>
            <template #dropdown>
                <el-dropdown-menu>
                    <el-dropdown-item @click="renameGroup">重命名</el-dropdown-item>
                    <el-dropdown-item @click="deleteGroup" v-if="storageCenter.userData.groupItemData.length > 1">删除
                    </el-dropdown-item>
                </el-dropdown-menu>
            </template>
        </el-dropdown>
        <div :class="`groupName`">{{ groupItem.name }}</div>
        <el-button style="visibility: hidden" :icon="Operation">
        </el-button>
        <div :class="`taskNumber`">{{ props.storageCenter.userData.taskItemData.filter(e => e.group ===
                groupItem.name && !e.isDone).length
        }}</div>
    </div>
</template>