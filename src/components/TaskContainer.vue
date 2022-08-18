<script setup>
import TaskEntry from './taksEntry.vue'
const props = defineProps({
    currentList: Object,
    storageCenter: Object
})
function ifDisplayDivider() {
    let flag = false
    props.currentList.forEach((taskItem) => {
        if (taskItem.isDone) { flag = true; }
    })
    return flag;
}
</script>
<template>
    <div :id="`incomplete`">
        <template v-for="taskItem in currentList" :key="taskItem.timestamp">
            <TaskEntry v-if="!taskItem.isDone" :taskItem="taskItem" :storageCenter="storageCenter" />
        </template>
    </div>
    <div v-if="ifDisplayDivider()" style="border-bottom: 2px solid #5768a9; block-size: 1rem;"></div>
    <div v-if="ifDisplayDivider()" style="block-size: 1rem;"></div>
    <div :id="`completed`">
        <template v-for="taskItem in currentList" :key="taskItem.timestamp">
            <TaskEntry v-if="taskItem.isDone" :taskItem="taskItem" :storageCenter="storageCenter" />
        </template>
    </div>
</template>