<template>
  <div class="content">
    <div class="header">
      <div class="title">{{ $t("CreateBooking") }}</div>
      <div class="btn">
        <el-button type="primary" @click="handleSubmit('draft', ruleFormRef)">{{
            $t("draft")
          }}</el-button>
        <el-button type="primary" @click="handleSubmit('preview')">{{
            $t("Preview")
          }}</el-button>
        <el-button type="primary">{{ $t("Booking") }}</el-button>
      </div>
    </div>
    <div class="box">
      <div class="left">
        <a-anchor
          :affix="false"
          @click="anchorClick"
          :getContainer="() => centerRef"
        >
          <a-anchor-link
            href="#ContactInfo"
            class="bigLink"
            :title="t('booking.ContactInfo')"
            ref="ContactInfoRef"
          />
          <a-anchor-link
            href="#CargoInfo"
            class="bigLink"
            :title="t('booking.CargoInfo')"
          />
          <a-anchor-link
            href="#ServicesInfo"
            class="bigLink"
            :title="t('booking.ServicesInfo')"
            :class="isChildren ? 'service' : ''"
          >
            <!-- <a-anchor-link
              class="childrenLink"
              href="#Departure"
              title="Anchor Props"
            />
            <a-anchor-link
              class="childrenLink"
              href="#Carrier"
              title="Link Props"
            />
            <a-anchor-link
              class="childrenLink"
              href="#DestinationPort"
              title="DestinationPort"
            />
            <a-anchor-link
              class="childrenLink"
              href="#Other"
              title="DestinationPort"
            /> -->
            <a-anchor-link
              v-for="item in params.serviceProductTree"
              class="childrenLink"
              :title="item.serviceCategoryName"
              :href="
                item.code == '001'
                  ? '#Departure'
                  : item.code == '002'
                  ? '#Carrier'
                  : item.code == '003'
                  ? '#DestinationPort'
                  : item.code == '004'
                  ? '#Other'
                  : ''
              "
            ></a-anchor-link>
          </a-anchor-link>
        </a-anchor>
      </div>
      <div class="center" id="center" ref="centerRef">
        <el-collapse v-model="activeNames">
          <el-form
            :model="params.createForm"
            label-width="120px"
            class="editRuleForm"
            status-icon
            label-position="top"
            :rules="createRules"
            ref="ruleFormRef"
          >
            <div ref="ContactInfoRef">
              <el-collapse-item name="1" id="ContactInfo">
                <template #title>
                  <div class="collapseTitle">
                    {{ $t("booking.ContactInfo") }}
                  </div>
                </template>
                <ContactInfo
                  v-model:Obj="params.createForm"
                  :contactPersonList="params.contactPersonList"
                  :contactEmailList="params.contactEmailList"
                  :key="uploadKey"
                />
              </el-collapse-item>
            </div>
            <div ref="CargoInfoRef">
              <el-collapse-item name="2" id="CargoInfo">
                <template #title>
                  <div class="collapseTitle">{{ $t("booking.CargoInfo") }}</div>
                </template>
                <CargoInfo
                  v-model:Obj="createForm"
                  :freightData="params.freightData"
                />
              </el-collapse-item>
            </div>
            <div ref="servicesRef">
              <el-collapse-item name="3" class="collapse3" id="ServicesInfo">
                <template #title>
                  <div class="collapseTitle">
                    {{ $t("booking.ServicesInfo") }}
                  </div>
                </template>
                <ServerInfo
                  v-model:Obj="params.createForm"
                  :IncotermsData="params.IncotermsData"
                  :serviceProductTree="params.serviceProductTree"
                  :TransportModeData="params.TransportModeData"
                  :key="uploadKey"
                />
              </el-collapse-item>
            </div>
          </el-form>
        </el-collapse>
      </div>
      <div class="right"></div>
    </div>
  </div>

  <Dialog
    :visible="viewShow"
    @close="closeDiaLog('view')"
    :width="780"
    title="viewDetails"
    :isLine="true"
    :showBtn="false"
  >
    <Preview
      v-model:Obj="params.createForm"
      :contactPersonList="params.contactPersonList"
      :contactEmailList="params.contactEmailList"
      :key="uploadKey"
    />
  </Dialog>
</template>
<script setup lang="ts">
import { t } from "@/utils/i18n";
import _ from "lodash";
import ServerInfo from "./component/serverInfo.vue";
import ContactInfo from "./component/contactInfo.vue";
import CargoInfo from "./component/cargoInfo.vue";
import Preview from "./component/preview.vue";
import { useRoute } from "vue-router";
import { createBookingApi } from "@/api/model/iLogistics/createBooking/createBooking";
import { useUserStore } from "@/stores/state/users";
import { ruleEmail } from "@/utils/Validators";
import { emunApi } from "@/api/enmu";
import { onBeforeMount, watch } from "vue";
const route = useRoute();
const store = useUserStore();
const { proxy } = getCurrentInstance() as any;
const activeNames = ref(["1", "2", "3"]);
const viewShow = ref(false);
const centerRef = ref<HTMLElement | null>();
const servicesRef = ref<HTMLElement | null>();
const ContactInfoRef = ref<HTMLElement | null>();
const CargoInfoRef = ref<HTMLElement | null>();
const uploadKey = ref(false);
const isChildren = ref(false);
const ruleFormRef = ref<any>();
const createForm = ref<any>({
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
});
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
//联系人校验  (联系人和联系人邮箱必须两个都为下拉中的数据或者都为手动录入，不能一个下拉一个手动录入)
function contactPersonRule(rule: any, value: string, callback: Function) {
  if (value == "" || value == undefined) {
    callback(new Error(t("require", [t("booking.ContactPerson")])));
  } else {
    let personData = params.contactPersonList.filter((item) => {
      return item.personInChargeName == value;
    });
    let emailData = params.contactEmailList.filter((item) => {
      return item.contactPersonEmail == params.createForm.contactPersonEmail;
    });
    if (personData && personData.length > 0) {
      params.createForm.contactPersonEmail = personData[0].personInChargeEmail;
      callback();
    } else {
      if (emailData && emailData.length > 0) {
        params.createForm.contactPersonEmail = null;
        callback();
      } else {
        callback();
      }
    }
  }
}
//联系人邮箱校验 (联系人和联系人邮箱必须两个都为下拉中的数据或者都为手动录入，不能一个下拉一个手动录入)
function contactEmailRule(rule: any, value: string, callback: Function) {
  let reg = ruleEmail();
  if (value == "" || value == undefined) {
    callback(new Error(t("require", [t("booking.ContactEmail")])));
  } else if (!reg.test(value)) {
    callback(new Error(t("formatErr", [t("booking.ContactEmail")])));
  } else {
    let personData = params.contactPersonList.filter((item) => {
      return item.personInChargeName == params.createForm.contactPerson;
    });
    let emailData = params.contactEmailList.filter((item) => {
      return item.contactPersonEmail == value;
    });
    if (emailData && emailData.length > 0) {
      params.createForm.contactPerson = emailData[0].contactPerson;
      callback();
    } else {
      if (personData && personData.length > 0) {
        params.createForm.contactPerson = null;
        callback();
      } else {
        callback();
      }
    }
  }
}
//租户名称校验 (租户名称带出负责人，负责人带出邮箱地址，否则清空)
function groupNameRule(rule: any, value: string, callback: Function) {
  if (value) {
    createBookingApi
      .getTenantInfoByTenantName({
        tenantName: value,
      })
      .then((res) => {
        if (
          res.data.code == "0" &&
          res.data.data &&
          res.data.data.groupTenantId
        ) {
          params.selectedGroupData = res.data.data;
          params.createForm.groupTenantId = res.data.data.groupTenantId;
          callback();
        } else {
          params.selectedGroupData = {};
          params.createForm.personInChargeEmail = "";
          params.createForm.personInChargeName = "";
          params.createForm.personInCharge = "";
          params.selectedPersonInCharge = {};
          callback(new Error(t("booking.GroupNameInexistence")));
        }
      });
  } else {
    params.selectedGroupData = {};
    params.createForm.personInChargeEmail = "";
    params.createForm.personInChargeName = "";
    params.createForm.personInCharge = "";
    params.selectedPersonInCharge = {};
    callback();
  }
}
//负责人校验(租户名称带出负责人，负责人带出邮箱地址，否则清空)
function personInChargeRule(rule: any, value: string, callback: Function) {
  if (value) {
    if (params.selectedGroupData && params.selectedGroupData.groupTenantId) {
      createBookingApi
        .getUserInfoByUsername({
          username: value,
          tenantId: params.selectedGroupData.groupTenantId,
        })
        .then((res) => {
          if (res.data.code == "0" && res.data.data.personInCharge) {
            params.selectedPersonInCharge = res.data.data;
            params.createForm.personInCharge = res.data.data.personInCharge; //负责人id
            params.createForm.personInChargeEmail = //负责人邮箱
              res.data.data.personInChargeEmail;
            callback();
          } else {
            callback(new Error(t("booking.PersonInChargeErr")));
            params.createForm.personInChargeEmail = "";
            params.selectedPersonInCharge = {};
          }
        });
    } else {
      callback(new Error(t("booking.GroupNamePlc")));
      params.createForm.personInChargeEmail = "";
      params.selectedPersonInCharge = {};
    }
  }
}
const createRules = reactive<any>({
  contactPerson: [
    {
      required: true,
      trigger: ["blur", "change"],
      validator: contactPersonRule,
    },
  ],
  contactPersonEmail: [
    {
      required: true,
      validator: contactEmailRule,
      trigger: ["blur", "change"],
    },
  ],
  shipper: [
    {
      required: true,
      message: t("require", [t("booking.Shipper")]),
      trigger: ["blur", "change"],
    },
  ],
  consignee: [
    {
      required: true,
      message: t("require", [t("booking.Consignee")]),
      trigger: ["blur", "change"],
    },
  ],
  groupName: [
    {
      validator: groupNameRule,
      trigger: ["blur"],
    },
  ],
  personInChargeName: [
    {
      validator: personInChargeRule,
      trigger: ["blur"],
    },
  ],
  polCode: [
    {
      required: true,
      message: t("require", [t("Freight.POL")]),
      trigger: ["blur", "change"],
    },
  ],
  podCode: [
    {
      required: true,
      message: t("require", [t("Freight.POD")]),
      trigger: ["blur", "change"],
    },
  ],
  carrierCode: [
    {
      required: true,
      message: t("require", [t("Freight.Carrier")]),
      trigger: ["blur", "change"],
    },
  ],
});
const freightReady = ref<boolean>(false);
const init = () => {
  if (route.query && route.query.freightCode) {
    createBookingApi
      .searchAllTenantFreight({
        freightCode: route.query.freightCode,
      })
      .then((res) => {
        if (res.data.code == "0") {
          params.freightData = res.data.data.freightDTO;
          params.serviceProductTree = res.data.data.serviceProductTree;
          if (
            params.freightData.validStartDate &&
            params.freightData.validEndDate
          ) {
            //处理有效期回显
            params.createForm.validData = [
              res.data.data.freightDTO.validStartDate,
              res.data.data.freightDTO.validEndDate,
            ];
          }
          params.createForm = {
            aaa: 111,
          };
          uploadKey.value = !uploadKey.value;
          console.log(params);
          console.log(params.createForm);
          freightReady.value = true;
        }
      });
  }
  if (store.userInfo) {
    params.createForm.contactPerson = store.userInfo.username;
    params.createForm.contactPersonEmail = store.userInfo.userEmail;
  }

  //获取联系人下拉
  createBookingApi.getUserInfoListByUsername({}).then((res) => {
    if (res.data.code == "0") {
      params.contactPersonList = res.data.data;
      uploadKey.value = !uploadKey.value;
    }
  });
  //获取联系人邮箱下拉
  createBookingApi.getUserInfoListByEmail({}).then((res) => {
    if (res.data.code == "0") {
      params.contactEmailList = res.data.data;
      uploadKey.value = !uploadKey.value;
    }
  });
  //获取服务Incoterms枚举数据
  emunApi.getemunApi("IncotermsEnum").then((res: any) => {
    params.IncotermsData = res.data.data;
  });
  //获取服务内的物流服务方式枚举数据
  emunApi.getemunApi("TransportModeEnum").then((res: any) => {
    params.TransportModeData = res.data.data;
  });
};
onBeforeMount(() => {
  init();
});
watch(
  () => params.createForm,
  (val) => {
    params.createForm = val;
  },
  {
    deep: true,
    immediate: true,
  }
);
const handleSubmit = async (type: string, formEl?: any) => {
  console.log(2222, params.createForm);
  if (type == "preview") {
    viewShow.value = true;
  } else {
    if (!formEl) return;
    await formEl.validate((valid: any, fields) => {
      if (valid) {
        console.log(2222, params.createForm);
      }
    });
  }
};
const anchorClick = (e: Event, link: string) => {
  // if (
  //   link.href != "#ContactInfo" &&
  //   link.href != "#CargoInfo" &&
  //   link.href != "#ServicesInfo"
  // ) {
  //   isChildren.value = true;
  // } else {
  //   isChildren.value = false;
  // }
};
const closeDiaLog = (type?: string) => {
  viewShow.value = false;
};
// onMounted(() => {
//   let centerEl = window;
//   let ContactInfoEl = window;
//   let CargoInfoEl = window;

//   if (centerRef) {
//     centerEl = centerRef.value;
//   }

//   if (ContactInfoRef) {
//     ContactInfoEl = ContactInfoRef.value;
//   }
//   if (CargoInfoRef) {
//     CargoInfoEl = CargoInfoRef.value;
//   }
//   centerEl.addEventListener("scroll", function () {
//     if (
//       ContactInfoEl.clientHeight + CargoInfoEl.clientHeight + 40 <
//       centerEl.scrollTop
//     ) {
//       if (isChildren.value != true) {
//         isChildren.value = true;
//       }
//     } else {
//       isChildren.value = false;
//     }
//   });
// });
</script>

<style scoped lang="scss">
.content {
  height: calc(100vh - 161px);
  background-color: white;
  display: flex;
  flex-direction: column;
  :deep(.el-collapse-item__header) {
    font-size: 14px;
  }
  .header {
    display: flex;
    justify-content: space-between;
    margin-bottom: 18px;
    .title {
      width: calc(100% - 260px);
      height: 33px;
      opacity: 1;
      // padding: 24px 24px 0px 24px;
      /* 文字/二级加粗 */
      font-family: Poppins;
      font-size: 24px;
      font-weight: bold;
      line-height: 32px;
      letter-spacing: 0px;

      color: #17234d;
    }
    .btn {
      width: 260px;
      text-align: right;
    }
  }

  .box {
    width: 100%;
    height: calc(100% - 51px);
    display: flex;

    .left {
      width: 180px;
      height: 100%;
      // background-color: yellow;

      :deep(.ant-anchor-ink) {
        display: none;
      }

      :deep(.ant-anchor-wrapper) {
        height: 100%;
      }

      :deep(.ant-anchor-wrapper) {
        overflow: initial;
      }

      .bigLink {
        margin-right: 40px;
        margin-bottom: 16px;
        width: 140px;
        height: 50px;
        border-radius: 4px;
        padding: 0px 0px 0px 0px;
        line-height: 50px;
        display: inline-block;
        /* 白色 */
        background: #ffffff;

        box-sizing: border-box;
        /* 表格悬浮 */
        border: 1px solid #f0f3f9;

        /* 悬浮-深 */
        box-shadow: 0px 4px 16px 0px rgba(23, 35, 77, 0.14);

        :deep(.ant-anchor-link-title) {
          padding: 0px 0px 0px 12px !important;
        }
        :deep(a:hover) {
          color: #235bda;
        }
      }

      :deep(.ant-anchor-link-title-active) {
        /* 蓝色/背景 */
        background: rgba(35, 91, 218, 0.05);
        border-radius: 4px;
        box-sizing: border-box;
        /* 蓝色/默认 */
        border: 1px solid #235bda;

        /* 悬浮-深 */
        box-shadow: 0px 4px 16px 0px rgba(23, 35, 77, 0.14);
        /* 蓝色/默认 */
        color: #235bda;
        font-family: Poppins;
        font-size: 14px;
        font-weight: 800;
      }

      .service {
        background: rgba(35, 91, 218, 0.05);
        box-sizing: border-box;
        border: 1px solid #235bda;
        :deep(.ant-anchor-link-title) {
          /* 蓝色/默认 */
          color: #235bda;
          font-family: Poppins;
          font-size: 14px;
          font-weight: 800;
          background: rgba(35, 91, 218, 0.05);
          /* 悬浮-深 */
          box-shadow: 0px 4px 16px 0px rgba(23, 35, 77, 0.14);
        }
        :deep(.ant-anchor-link-title-active) {
          border: none;
        }
      }
      .childrenLink {
        // box-shadow: none;
        padding-top: 0px;
        margin-top: 17px;
        margin-bottom: 0px;
        // margin-left: 16px;
        :deep(.ant-anchor-link-title) {
          padding: 0px 0px 0px 12px !important;
          color: rgba(0, 0, 0, 0.85) !important;
          font-weight: normal;
          background: white !important;
          box-shadow: none;
        }

        :deep(.ant-anchor-link-title-active) {
          box-sizing: border-box;
          /* 蓝色/默认 */
          color: #235bda !important;
          font-size: 14px;
          border: none;
          box-shadow: none;
          font-weight: normal;
          background-color: white !important;
        }
      }
    }
    .center {
      width: calc(100% - 513px);
      height: 100%;
      overflow: auto;
      /* 白色 */
      background: #ffffff;

      :deep(.el-collapse) {
        border-bottom: 0;
        border-top: 0;
        padding: 8px;
      }

      :deep(.el-collapse-item) {
        margin-bottom: 20px;
        /* 悬浮-深 */
        box-shadow: 0px 4px 16px 0px rgba(23, 35, 77, 0.14);

        .el-collapse-item__content {
          padding: 0px 20px;
        }
      }

      .collapse3 {
        :deep(.el-collapse-item__content) {
          padding: 0px;
        }

        :deep(.el-collapse) {
          padding: 0px !important;
        }
      }

      :deep(.el-collapse-item__arrow) {
        margin: 0 21px 0 auto;
      }

      .collapseTitle {
        font-weight: 800;
        &::before {
          width: 12px;
          height: 15px;
          opacity: 1;
          content: "";
          background: #235bda;
          display: inline-block;
          vertical-align: middle;
          /* position: absolute; */
          margin-right: 7px;
          margin-top: -3px;
        }
      }

      .priceStyle {
        :deep(.el-input-number) {
          width: 100%;
          text-align: left;
          .el-input__inner {
            text-align: left;
          }
        }
      }
    }
    .right {
      width: 333px;
      height: 100%;
      background-color: red;
    }
  }

  :deep(.el-input__validateIcon) {
    display: none;
  }
}
</style>
