<template>
  <div>
    <button @click="click">打印</button>
  </div>
</template>
<script setup lang="ts">
const props = defineProps<{
  params: {}
}>()
const click = () => {
  console.log(props.params);
};
console.log(props.params);
</script>
