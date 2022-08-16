<script setup>
import{ref}from'vue'
import { Operation } from '@element-plus/icons-vue'

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
    console.log(props.groupItem.name)
    let number = props.storageCenter.userData.taskItemData.filter(e => e.group === props.groupItem.name).length;
    if (confirm(number ? "确认删除" + props.groupItem.name + "分组？这将会删除该分组中共计" + number + "个事项。" : '确认删除分组'+props.groupItem.name+'？（该分组中无任务）')) {
        let index = props.storageCenter.userData.groupItemData.indexOf(props.groupItem);
        props.storageCenter.userData.groupItemData.splice(index, 1);
        if(props.groupItem.name === props.storageCenter.userStatus.currentGroup){
            if(props.storageCenter.userData.groupItemData[index] !== undefined){
                props.storageCenter.userStatus.currentGroup=props.storageCenter.userData.groupItemData[index].name;
            }
            else{
                props.storageCenter.userStatus.currentGroup=props.storageCenter.userData.groupItemData[index-1].name;
            }
        }
    }
}
const dropdown1 = ref()
function showClick() {
  dropdown1.value.handleOpen()
}
</script>
<template>
    <div :class="`groupItem`" :groupname="groupItem.name">
        <div :class="`groupName`">{{ groupItem.name }}</div>
        <div :class="`groupTrailing`">
            <div :class="`taskNumber`">{{ props.storageCenter.userData.taskItemData.filter(e => e.group ===
                    groupItem.name && !e.isDone).length
            }}</div>
            <div style="margin: 15px">
                <el-button @click="showClick">
                    <el-icon>
                        <Operation />
                    </el-icon>
                </el-button>
            </div>
            <el-dropdown ref="dropdown1" trigger="contextmenu" style="margin-right: 30px">
                <span class="el-dropdown-link"></span>
                <template #dropdown>
                    <el-dropdown-menu>
                        <el-dropdown-item @click="renameGroup">重命名</el-dropdown-item>
                        <el-dropdown-item @click="deleteGroup" v-if="storageCenter.userData.groupItemData.length > 1">删除</el-dropdown-item>
                    </el-dropdown-menu>
                </template>
            </el-dropdown>
        </div>
    </div>
</template>