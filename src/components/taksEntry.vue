<script setup>
import { ref, watch } from 'vue'
// const setDeadlineElement = ref()
props.taskItem.isSetting = false
const props = defineProps({
    taskItem: Object,
    storageCenter: Object
})
function deadlineInput(event) {
    // props.taskItem.isSetting = false;
    props.taskItem.deadline = new Date(event.target.value + ':00.000Z').getTime(); // 转换为Unix时间戳放入存储中心，使用 UTC+0 时间
}
function taskContentFocus(event) {
    if (!event.target.parentElement.parentElement.nextElementSibling.hasAttribute('displayTaskOptions')) {
        if (document.querySelector('[displayTaskOptions]')) { document.querySelector('[displayTaskOptions]').removeAttribute('displayTaskOptions') }
        event.target.parentElement.parentElement.nextElementSibling.setAttribute('displayTaskOptions', '');
    }
    // setDeadlineElement.value.value = new Date().toISOString().slice(0, 16)
}
function taskContentBlur(event) {
    if (event.target.value == "") {
        event.target.value = "空事项"
    }
    props.taskItem.text = event.target.value;
}
function setDeadline() {
    if (props.taskItem.deadline === null) {
        props.taskItem.deadline = ''
        props.taskItem.deadline =new Date().toISOString().slice(0, 16);
        props.taskItem.isSetting = true;
    }
    else {
        props.taskItem.deadline = null;
        props.taskItem.isSetting = false;
    }
}
function clickDeleteTask() {
    { props.storageCenter.userData.taskItemData.splice(props.storageCenter.userData.taskItemData.indexOf(props.taskItem), 1); document.querySelector('[displayTaskOptions]').removeAttribute('displayTaskOptions') }
}
let ElSetDeadline = ref(Boolean(props.taskItem.deadline))
watch(ElSetDeadline,setDeadline)
</script>
<template>
    <div :class="`taskItem`" :timestamp="taskItem.createdAt">
        <div :class="`taskMain`">
            <div :class="`taskBox`">
                <input :type="`checkbox`" :class="`isDone`" :checked="taskItem.isDone"
                    @click="taskItem.isDone = !taskItem.isDone">
                <input :class="[`taskBox-inputTag`, `taskContent`]" :type="`text`" :value="taskItem.text"
                    @focus="taskContentFocus" @blur="taskContentBlur">
            </div>
            <div v-if="taskItem.isSetting && taskItem.deadline === ''|| taskItem.deadline !== null" :class="`taskDetails`">
                <input :type="`checkbox`" :class="`isDone`" style="visibility: hidden">

                <input v-if="taskItem.deadline || taskItem.isSetting && taskItem.deadline === ''"
                    :type="`datetime-local`" :class="`deadline`" @blur="deadlineInput"
                    :value="taskItem.deadline ? new Date(taskItem.deadline).toISOString().slice(0, 16) : ''">
            </div>
        </div>
        <div :class="`taskOptions`">
            <input :type="`checkbox`" :class="`isDone`" style="visibility: hidden">

            <!-- <input :type="`checkbox`" :class="`setDeadline`" @click="setDeadline"
                :checked="taskItem.isSetting || Boolean(taskItem.deadline)" ref="setDeadlineElement">截止时间 -->
            <el-checkbox v-model="ElSetDeadline" @click="setDeadline" label="截止时间" border style="background-color:white; color:#589ef8"/>
            <button :class="`deleteTask`" @click="clickDeleteTask">删除</button>
        </div>
    </div>
</template>