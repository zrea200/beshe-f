<template>
  <template v-if="user">
    <van-cell title="用户编号" :value="user.planetCode" />
    <van-cell
      title="昵称"
      is-link
      to="/user/edit"
      :value="user.username"
      @click="toEdit('username', '昵称', user.username)"
    />
    <van-cell title="账号" :value="user.userAccount" />
    <van-cell title="头像" is-link to="/user/edit">
      <img style="height: 48px" :src="user.avatarUrl" />
    </van-cell>
    <van-cell title="性别">
      <template #value>
        <van-field
          v-model="genderLabel"
          readonly
          clickable
          name="性别"
          :input-align="'right'"
          @click="showGenderPicker = true"
          style="padding: 0; background: transparent"
        />
        <van-popup v-model:show="showGenderPicker" position="bottom">
          <van-picker
            :columns="genderColumns"
            @confirm="onGenderConfirm"
            @cancel="showGenderPicker = false"
          />
        </van-popup>
      </template>
    </van-cell>
    <van-cell
      title="电话"
      is-link
      to="/user/edit"
      :value="user.phone"
      @click="toEdit('phone', '电话', user.phone)"
    />
    <van-cell
      title="邮箱"
      is-link
      to="/user/edit"
      :value="user.email"
      @click="toEdit('email', '邮箱', user.email)"
    />
    <!-- 调用 formatDate 函数格式化日期 -->
    <van-cell title="注册时间" :value="formatDate(user.createTime)" />
  </template>
</template>

<script setup lang="ts">
import { useRouter } from "vue-router";
import { onMounted, ref } from "vue";
import myAxios from "../plugins/myAxios";
import { Toast } from "vant";
import { getCurrentUser } from "../services/user";

const user = ref();

onMounted(async () => {
  user.value = await getCurrentUser();
  // 根据用户性别值设置对应的标签
  if (user.value?.gender === 0) {
    genderLabel.value = '男';
  } else if (user.value?.gender === 1) {
    genderLabel.value = '女';
  } else {
    genderLabel.value = '保密';
  }
});

const router = useRouter();

const showGenderPicker = ref(false);
const genderLabel = ref('保密');
const genderColumns = ['男', '女', '保密'];

const toEdit = (
  editKey: string,
  editName: string,
  currentValue: number | string
) => {
  router.push({
    path: "/user/edit",
    query: {
      editKey,
      editName,
      currentValue,
    },
  });
};

const onGenderConfirm = async (value: string) => {
  genderLabel.value = value;
  let genderValue = 3; // 默认为保密
  if (value === '男') {
    genderValue = 0;
  } else if (value === '女') {
    genderValue = 1;
  }
  // 直接发送请求更新性别
  const res = await myAxios.post('/user/update', {
    gender: genderValue
  });
  // if (res.code === 0) {
  //   Toast.success('修改成功');
  //   user.value.gender = genderValue;
  // } else {
  //   Toast.fail('修改失败');
  // }
  showGenderPicker.value = false;
};

// 定义格式化日期的函数
const formatDate = (dateStr: string | undefined) => {
  if (!dateStr) return "";
  const date = new Date(dateStr);
  const year = date.getFullYear();
  // getMonth() 返回的月份是从 0 开始的，所以要加 1，并使用 padStart(2, '0') 方法补零
  const month = String(date.getMonth() + 1).padStart(2, "0");
  const day = String(date.getDate()).padStart(2, "0");
  return `${year}-${month}-${day}`;
};
</script>

<style scoped>
:deep(.van-field__control) {
  text-align: right !important;
}
</style>
