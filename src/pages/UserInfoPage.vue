<template>
  <QPage>
    <QInnerLoading loading="true" color="black"/>
    <div class="user-info-page">
      <div>
        <QImg height="500px" width="400px" :src="userInfo.avatar" />
      </div>
      <div>
        <span class="label bg-amber">
          <span class="left-detail">first name: </span>
          {{ userInfo.first_name }}
        </span>
      </div>
      <div>
        <span class="label bg-amber">
          <span class="left-detail">last name: </span>
          {{ userInfo.last_name }}
        </span>
      </div>
      <div>
        <span class="label bg-amber">
          <span class="left-detail">email: </span>
          {{ userInfo.email }}
        </span>
      </div>
      <QBtn @click="handleBack" color="secondary">Powrót</QBtn>
      <QBtn @click="handleFavorite">{{ favoriteButtonText }}</QBtn>
    </div>
  </QPage>
</template>

<script setup>
import axios from 'axios';
import { onMounted, ref, computed } from 'vue';
import { useRouter, useRoute } from 'vue-router';
import { QBtn, QPage, QImg, QInnerLoading } from 'quasar';

const route = useRoute();
const router = useRouter();
const userID = ref(null);
const isFavorite = ref(false);
const loading = ref(false);
const userInfo = ref({
  avatar: null,
  email: null,
  first_name: null,
  id: null,
  last_name: null
});

const favoriteButtonText = computed(() => isFavorite?.value ? 'Usuń z ulubionych' : 'Dodaj do ulubionych');

const handleFavorite = () => {
  const favoriteUserList = JSON.parse(localStorage.getItem('favorite-user-list')) || [];
  if (favoriteUserList.some((user) => user.id === userID.value))
    favoriteUserList.some((user) => user.id === userID.value)
      ? favoriteUserList.filter((user) => user.id !== userID.value)
      : [...favoriteUserList, userInfo.value]
  isFavorite.value = !isFavorite.value
};

const handleBack = () => {
  router.push('/');
};

const getData = async (id) => {
  loading.value = true;
  const res = await axios.get(`https://reqres.in/api/users/${id}`);
  if (res.status === 200) {
    console.log("rezult w get data", res.data);
    userInfo.value = res.data?.data;

  }
  loading.value = false;
};

onMounted(async () => {
  userID.value = Number(route.params?.id)
  const favoriteUserList = JSON.parse(localStorage.getItem('favorite-user-list')) || [];
  isFavorite.value = favoriteUserList.some((user) => user.id === userID.value)
  await getData(userID.value);
});
</script>

