<script setup>
import GroupEntry from './groupEntry.vue'
const props = defineProps({
    groupItemArray: Object,
    storageCenter: Object
})

</script>
<template>
    <GroupEntry :groupItem="{ name: '全部事项' }" :storageCenter="storageCenter" />
    <GroupEntry :groupItem="{ name: '将要截止' }" :storageCenter="storageCenter" />
    <div style="border-bottom: 2px solid #c2c2c2"></div>
    <div :id="`groupContainer`">
        <template v-for="groupItem in groupItemArray">
            <GroupEntry :groupItem="groupItem" :storageCenter="storageCenter" />
        </template>
    </div>
</template>