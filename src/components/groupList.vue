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
    <div :id="`dividerGroupList`"></div>
    <div :id="`groupContainer`">
        <template v-for="groupItem in groupItemArray">
            <GroupEntry :groupItem="groupItem" :storageCenter="storageCenter" />
        </template>
    </div>
    
    <p :id="`messageToHR`">把手机横过来，landscape 的样式不同哟~</p>
</template>