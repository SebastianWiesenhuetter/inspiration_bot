<template>
    <div class="absolute top-0 left-0 h-screen w-screen">
        <div class="w-full h-full flex justify-center items-center">
            <button class="h-10 w-fit bg-slate-200 rounded-lg p-2" @click="leave">Start</button>
        </div>

    </div>
</template>

<script setup lang="ts">
const emit = defineEmits(['leave'])
const leave = () => {
    emit('leave')
}

</script>
<style scoped>
</style>