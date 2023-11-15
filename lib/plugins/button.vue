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
    methods: {
        onClick() {
            console.log('button--click--');
            this.$emit('click')
        }
    }
}
</script>

<style>

</style>