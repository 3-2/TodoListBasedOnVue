<script setup>
import TaskEntry from './taksEntry.vue'
const props = defineProps({
    currentList: Object,
    storageCenter: Object
})

</script>
<template>
    <div :id="`incomplete`">
        <template v-for="taskItem in currentList" :key="taskItem.timestamp">
            <TaskEntry v-if="!taskItem.isDone" :taskItem="taskItem" 
            :storageCenter="storageCenter"/>
        </template>
    </div>
    <div :id="`completed`">
        <template v-for="taskItem in currentList" :key="taskItem.timestamp">
            <TaskEntry v-if="taskItem.isDone" :taskItem="taskItem" 
            :storageCenter="storageCenter"/>
        </template>
    </div>
</template>