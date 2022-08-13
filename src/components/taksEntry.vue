<script setup>
const props = defineProps({ taskItem: Object })
</script>
<template>
    <div :class="`taskItem`" :timestamp="taskItem.createdAt" :isfocusing="``">
        <div :class="`taskMain`">
            <div :class="`taskBox`">
                <input :type="`checkbox`" :class="taskItem.isDone" :checked="taskItem.isDone">
                <input :class="[`taskBox-inputTag`, `taskContent`]" :type="`text`" :value="taskItem.text">
            </div>
            <div :class="`taskDetails`">
            </div>
        </div>
        <div :class="`taskOptions`">
            <input :type="`checkbox`" :class="`setDeadline`">截止时间
            <button :class="`deleteTask`">删除</button>
        </div>
    </div>
</template>
<style>
.taskItem[isFocusing="true"] .taskOptions {
    display: block;
}
</style>