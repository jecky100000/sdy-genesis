<template>
  <NavBar
    title="实名认证"
  />
  <div class=" pageCard-sm">
    <Space height="30" />
    <div class="text-base font-semibold">
      上传身份证照片
    </div>
    <Space height="8" />
    <div class=" text-xs2 text-grayDefault">
      认证通过后资料不可修改，平台会保护你的个人信息
    </div>
    <Space height="25" />
    <div class="flex items-center justify-center relative h-28">
      <div class="flex-auto h-full relative">
        <van-uploader :after-read="frontUpload">
          <van-image
            class="rounded-lg overflow-hidden"
            :width="parseInt($pxToPxRatio(165), 10)"
            :height="parseInt($pxToPxRatio(105), 10)"
            fit="cover"
            :src="frontUploadUrl || a2"
          />
        </van-uploader>
      </div>
      <Space width="15" />
      <div class="flex-auto h-full relative">
        <van-uploader :after-read="backUpload">
          <van-image
            class="rounded-lg overflow-hidden"
            :width="parseInt($pxToPxRatio(165), 10)"
            :height="parseInt($pxToPxRatio(105), 10)"
            fit="cover"
            :src="backUploadUrl || a1"
          />
        </van-uploader>
      </div>
    </div>
    <Space height="20" />
    <div class=" px-2.5 ring-1 rounded-lg2 ring-gray-200 flex items-center">
      <div class="flex-shrink-0 text-sm text-grayDefault">
        姓名：
      </div>
      <van-field
        v-model="name"
        class="text-sm"
        type="text"
        placeholder="请输入姓名"
      />
    </div>
    <Space height="10" />
    <div class=" px-2.5 ring-1 rounded-lg2 ring-gray-200 flex items-center">
      <div class="flex-shrink-0 text-sm text-grayDefault">
        身份证号：
      </div>
      <van-field
        v-model="number"
        class="text-sm"
        type="text"
        placeholder="请输入身份证"
      />
    </div>
    <div
      class="fixedBottomButton"
    >
      <van-button
        type="danger"
        block
        round
        :disabled="!name && !number && !frontUploadUrl && !backUploadUrl"
        :loading="loading"
        @click="submit"
      >
        提交
      </van-button>
    </div>
  </div>
  <van-popup
    v-model:show="errorPopup"
    :close-on-click-overlay="false"
    class="transparent overflow-visible"
  >
    <div class="px-9 bg-white rounded-lg2 w-80 text-blackDefault flex flex-col items-center">
      <Space height="23" />
      <van-image
        :width="parseInt($pxToPxRatio(141), 10)"
        :height="parseInt($pxToPxRatio(156), 10)"
        fit="cover"
        :src="b2"
      />
      <Space height="40" />
      <div class=" text-base font-semibold text-center">
        申请失败
      </div>
      <Space height="20" />
      <div class=" text-sm text-grayTip text-center">
        原因：{{ detail.failReason }}
      </div>
      <Space height="30" />
      <Icon
        class="absolute z-1 left-1/2 -bottom-4 transform -translate-x-1/2 translate-y-full"
        type="icon-close_circle"
        size="26"
        @click="() => errorPopup = false"
      />
    </div>
  </van-popup>
</template>
<script setup>
import { isIdentityCard } from 'validator';
import { Toast } from 'vant';
import { getCurrentInstance, ref } from 'vue';
import { useStore } from 'vuex';
import a1 from './images/a1.png';
import a2 from './images/a2.png';
import b2 from '@/assets/images/b2.png';

let {proxy} = getCurrentInstance();
let store = useStore();

let name = ref('');
let number = ref('');
let frontUploadUrl = ref('');
let backUploadUrl = ref('');
let detail = ref({});
let errorPopup = ref(false);

function getDetail() {
  proxy.$http('post', '/v1/user/authenticationDetails', {})
    .then(res => {
      if (res.data.status === 3) {
        errorPopup.value = true;
      } else {
        errorPopup.value = false;
      }
      detail.value = res.data;
      name.value = res.data.cardName;
      number.value = res.data.cardId;
      frontUploadUrl.value = res.data.frontUrl;
      backUploadUrl.value = res.data.backUrl;
    });
}
getDetail();

const frontUpload = (file) => {
  proxy.$http('file', '/v1/cdn/uploadImg', {
    file: file.file
  }).then((res) => {
    frontUploadUrl.value = res.data;
    getCardValue(file.file);
  });
};
function getCardValue(file) {
  proxy.$http('file', '/v1/user/verifyIdCard', {
    file: file,
  })
    .then(res => {
      name.value = res.data.name;
      number.value = res.data.idnumber;
    }).thenError(res => Toast(res.msg));
}
const backUpload = (file) => {
  proxy.$http('file', '/v1/cdn/uploadImg', {
    file: file.file
  }).then((res) => {
    backUploadUrl.value = res.data;
  });
};

let loading = ref(false);
let submit = proxy.$debounce(() => {
  if (!name.value) {
    Toast('请输入姓名');
    return;
  }
  if (!number.value) {
    Toast('请输入身份证号');
    return;
  }
  if (!isIdentityCard(number.value, ['zh-CN'])) {
    Toast('请输入正确的身份证号');
    return;
  }
  if (!frontUploadUrl.value) {
    Toast('请上传身份证正面照');
    return;
  }
  if (!backUploadUrl.value) {
    Toast('请上传身份证反面照');
    return;
  }
  loading.value = true;
  proxy.$http('post', '/v1/user/authentication', {
    'backUrl': backUploadUrl.value,
    'cardId': number.value,
    'cardName': name.value,
    'frontUrl': frontUploadUrl.value,
    'id': detail.value.id,
    'passportUrl': '',
    'regions': '中国',
    'type': '0'
  })
    .then(res => {
      Toast.success('提交成功');
      proxy.$router.back();
    }).thenError(res => {
      Toast(res.msg);
    }).all(res => {
      loading.value = false;
    });
});

</script>
<style lang="less" scoped>
</style>
