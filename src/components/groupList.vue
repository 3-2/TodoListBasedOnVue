<script setup>
import GroupEntry from './groupEntry.vue'
const props = defineProps({
    groupItemArray: Object,
    storageCenter: Object
})

</script>
<template>
    <div>
        <div :class="`groupItem`" :id="`allTasks`">
            <div :class="`groupName`">全部事项</div>
            <div :class="`groupTrailing`">
                <div :class="`taskNumber`">{{ storageCenter.userData.taskItemData.filter(e =>
                        !e.isDone).length
                }}</div>
            </div>
        </div>
        <div :class="`groupItem`" :id="`toDeadline`">
            <div :class="`groupName`">将要截止</div>
            <div :class="`groupTrailing`">
                <div :class="`taskNumber`">{{ storageCenter.userData.taskItemData.filter(e =>
                        e.deadline
                        &&
                        !e.isDone).length
                }}</div>
            </div>
        </div>
    </div>
    <template v-for="groupItem in groupItemArray">
        <GroupEntry :groupItem="groupItem" :storageCenter="storageCenter" />
    </template>
</template>