<template>
  <ExampleComponent v-model:params="params" />
</template>

<script setup lang="ts">
import ExampleComponent from '@/components/ExampleComponent.vue'
import { onBeforeMount, reactive, watch, readonly } from 'vue'

const params = reactive<any>({
  createForm: {
    orderFreightProductDTOS: [
      {
        productId: "11",
        containerCode: 22,
        quantity: "33",
        gwKgs: "44",
        key: 1,
      },
      {
        productId: "11",
        containerCode: 22,
        quantity: "33",
        gwKgs: "44",
        key: 2,
      },
    ],
    validData: [],
    orderFreightProductDTOsSumPrice: 0,
  },
  contactPersonList: [], //联系人下拉
  contactEmailList: [], //联系人邮箱下拉
  selectedGroupData: {}, //选中的租户信息
  selectedPersonInCharge: {}, //选中的负责人信息
  freightData: {}, //选中的运价信息
  serviceProductTree: [], //选中的服务树形结构
  IncotermsData: [], //Incoterms下拉枚举
  TransportModeData: [], //物流服务方式下拉枚举
});

const init = () => {
  new Promise((resolve, reject) => {
    setTimeout(resolve, 3000);
  }).then(() => {
    params.freightData = {
      aaa: 222,
      bbb: 222,
      validStartDate: '2023-06-03',
      validEndDate: '2023-07-03',
    }
    params.serviceProductTree = {}
    if (
      params.freightData.validStartDate &&
      params.freightData.validEndDate
    ) {
      //处理有效期回显
      params.createForm.validData = [
        params.freightData.validStartDate,
        params.freightData.validEndDate,
      ];
    }
    params.createForm = {
      aaa: 222,
    };
    console.log('params', params);
    console.log('params.createForm', params.createForm);
  });
}

onBeforeMount(() => {
  init()
})
// watch(
//   () => params.createForm,
//   (val) => {
//     params.createForm = val
//   },
//   {
//     deep: true,
//     immediate: true
//   }
// )
</script>
