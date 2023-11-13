<template>
  <div @click="onClick">button-test</div>
</template>

<script>
export default {
    name: 'JoyinButton',
    data() {
        return {
            test: '测试'
        }
    },
    method: {
        onClick() {
            console.log('button--click--');
        }
    }
}
</script>

<style>

</style>