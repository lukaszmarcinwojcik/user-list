<template>
  <div class="wrapper">
    <QPage>
      <QTable
        :loading="loading"
        :rows="filteredUserList"
        :columns="columns"
        selection="multiple"
        v-model:pagination="pagination"
        v-model:selected="favoriteUserList"
        @request="handleRequest"
        @row-click="handleClick"
      >
        <template #loading>
          <QInnerLoading
            showing
            color="secondary"
          />
        </template>
        <template #top-right>
          <QInput
            dense
            outlined
            aria-placeholder="search"
            v-model="search"
          />
        </template>
      </QTable>
    </QPage>
  </div>
</template>

<script setup>
import axios from 'axios';
import { onMounted, ref, computed, watch } from 'vue';
import { useRouter} from 'vue-router';
import { QPage, QTable, QInnerLoading, QInput } from 'quasar';

const router = useRouter();

const columns = [
  { name: 'first_name', label: 'first name', field: 'first_name', align: 'left' },
  { name: 'last_name', label: 'last name', field: 'last_name', align: 'left' },
  { name: 'email', label: 'email', field: 'email', align: 'left' }
];

const loading = ref(false);
const search = ref('');
const userList = ref([]);
const favoriteUserList = ref([]);
const pagination = ref({
  page: 1,
  rowsPerPage: 3,
  rowsNumber: 0
});

const filteredUserList = computed(()=>
  userList.value.filter(
    ({ first_name, last_name, email })=>[first_name, last_name, email]
    .some(val=>{
      return val.toLowerCase().includes(search.value.toLowerCase().trim());
    })
  )
);

const getData = async (page = 1)=>{
  loading.value = true;
  const res = await axios.get(`https://reqres.in/api/users`, { params: { page } });
  if (res.status === 200) {
    userList.value = res.data?.data;

    pagination.value.page = res.data?.page;
    pagination.value.rowsPerPage = res.data?.per_page;
    pagination.value.rowsNumber = res.data?.total;
  }
  loading.value = false;
};

const handleRequest = async (props)=>{
  await getData(props.pagination.page);
};

const handleClick = async (event, row) => {
  router.push(`user/${row.id}`)
};

watch(
  () => ({
    favoriteUserList: favoriteUserList.value,
  }),
  () => {
    if (loading.value) {
      return;
    }
    localStorage.setItem('favorite-user-list', JSON.stringify(favoriteUserList.value));
  },
);

onMounted(async () => {
  favoriteUserList.value = JSON.parse(localStorage.getItem('favorite-user-list')) || [];
  await getData();
});
</script>

<style scoped lang="scss">
.wrapper {
  padding: 100px 30px;
}
</style>
