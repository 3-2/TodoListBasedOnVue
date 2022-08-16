<script setup>
import {ref} from  'vue'
const setDeadlineElement = ref();
props.taskItem.isSetting = false
const props = defineProps({
    taskItem: Object,
    storageCenter: Object
})
function deadlineInput(event) {
    console.log('Date is: ',event.target.value);
    props.taskItem.deadline = new Date(event.target.value + ':00.000Z').getTime(); // 转换为Unix时间戳放入存储中心，使用 UTC+0 时间
}
function taskContentFocus() {
    setDeadlineElement.value.value=new Date().toISOString().slice(0, 16)
    document.styleSheets[props.storageCenter.userStatus.StyleSheetIndex].cssRules[0].selectorText = `[timestamp="${props.taskItem.createdAt}"] > .taskOptions`;
}
function taskContentBlur(event) {
    if (event.target.value == "") {
        event.target.value = "空事项"
    }
    props.taskItem.text = event.target.value;
}
function setDeadline() {
    if (props.taskItem.deadline === null || props.taskItem.deadline === '') {
        props.taskItem.deadline = ''
        props.taskItem.isSetting = true;
    }
    else {
        props.taskItem.deadline = null;
        props.taskItem.isSetting = false;
    }
}
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
            <div :class="`taskDetails`">
                <input v-if="Boolean(taskItem.deadline) || taskItem.isSetting" type="datetime-local" class="deadline"
                    @blur="deadlineInput"
                    :value="taskItem.deadline ? new Date(taskItem.deadline).toISOString().slice(0, 16) : ''">
            </div>
        </div>
        <div :class="`taskOptions`">
            <input :type="`checkbox`" :class="`setDeadline`" @click="setDeadline"
                :checked="taskItem.isSetting || Boolean(taskItem.deadline)" ref="setDeadlineElement">截止时间
            <button :class="`deleteTask`"
                @click="() => { props.storageCenter.userData.taskItemData.splice(props.storageCenter.userData.taskItemData.indexOf(props.taskItem), 1) }">删除</button>
        </div>
    </div>
</template>