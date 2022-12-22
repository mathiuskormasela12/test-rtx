<template>
  <div class="about">
    <!-- 8.
      Display the data with a structured
      table in following columns,
      element-ui is ready for use
    -->
    <!-- property.name | property.location.country | property.reviews.summary.score -->
    <el-table :data="data" style="width: 100%" v-loading="loading">
      <el-table-column prop="id" label="Id" width="180" v-if="fromApi" />
      <el-table-column prop="name" label="Name" width="180" />
      <el-table-column v-if="!showNameOnly" prop="country" label="Country" width="180" />
      <el-table-column  v-if="!showNameOnly" prop="score" label="Score" />
      <el-table-column fixed="right" label="Operations" width="120" v-if="fromApi">
      <template #default="scope">
        <el-button
          link
          type="danger"
          size="small"
          @click.prevent="deleted(scope.row.id)"
        >
          Remove
        </el-button>
        <br /><br />
        <el-button text @click="getSingleDataFromExpressApi(scope.row.id)" link
          type="primary"
          size="small">
          Update
        </el-button>
      </template>
    </el-table-column>
    </el-table>
    <!-- 9. Do a simple pagination of 5 per page-->
    <el-pagination layout="prev, pager, next"
      :total="hotels.length"
      @current-change="setPage"
    >
    </el-pagination>
    <!-- 10. Hide the entry without country and/or score-->
    <el-button type="primary" @click="showNameOnlyFunc">Hide Country & Score Field</el-button>

    <el-button type="primary" @click="showDataFromExpressApiFunc">
      Show Data From Express API
    </el-button>

    <el-button type="primary" @click="saveDataToDb">
     Save To Database
    </el-button>

    <el-dialog v-model="centerDialogVisible" title="Warning" width="30%" center>
      <el-form
        :label-position="labelPosition"
        label-width="100px"
        :model="formLabelAlign"
        style="max-width: 460px"
      >
        <el-form-item label="Name">
          <el-input v-model="singleData.name" />
          <input type="hidden" v-model="singleData.id" />
        </el-form-item>
        <el-form-item label="Activity zone">
          <el-input v-model="singleData.country" />
        </el-form-item>
        <el-form-item label="Activity form">
          <el-input v-model.number="singleData.score" />
        </el-form-item>
      </el-form>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="centerDialogVisible = false">Cancel</el-button>
        <el-button type="primary" @click="update(singleData.id)">
          Update
        </el-button>
      </span>
    </template>
  </el-dialog>
  </div>
</template>

<script setup>
import {
  ref,
  onMounted,
  reactive,
  computed,
} from 'vue';
import axios from 'axios';

const hotels = reactive([]);
const page = ref(1);
const pageSize = ref(5);
const loading = ref(false);
const showNameOnly = ref(false);
const fromApi = ref(false);
const centerDialogVisible = ref(false);

const data = computed({
  get() {
    let raw = [];

    if (fromApi.value) {
      if (showNameOnly.value) {
        raw = hotels.map((item) => ({
          id: item.id ? item.id : '',
          name: item && item.property && item.property.name ? item.property.name : '',
        }));
      } else {
        raw = hotels.map((item) => ({
          id: item.id ? item.id : '',
          name: item && item.property && item.property.name ? item.property.name : '',
          country: item && item.property && item.property.location && item.property.location.country ? item.property.location.country : '',
          score: item && item.property && item.property.reviews && item.property.reviews.summary && item.property.reviews.summary.score ? item.property.reviews.summary.score : '',
        }));
      }
    } else if (showNameOnly.value) {
      raw = hotels.map((item) => ({
        name: item && item.property && item.property.name ? item.property.name : '',
      }));
    } else {
      raw = hotels.map((item) => ({
        name: item && item.property && item.property.name ? item.property.name : '',
        country: item && item.property && item.property.location && item.property.location.country ? item.property.location.country : '',
        score: item && item.property && item.property.reviews && item.property.reviews.summary && item.property.reviews.summary.score ? item.property.reviews.summary.score : '',
      }));
    }

    return raw.slice(pageSize.value * page.value - pageSize.value, pageSize.value * page.value);
  },

  set(val) {
    hotels.splice(0, hotels.length);
    hotels.push(...val);
  },
});

const getDataFromExternalApi = () => {
  fromApi.value = false;
  loading.value = true;

  axios.get('http://localhost:3000/api/fromSource')
    .then((res) => {
      data.value = res?.data?.outlets?.availability?.results;
      loading.value = false;
    }).catch((err) => {
      data.value = [];
      loading.value = false;
      console.log(err);
    });
};

const getDataFromExpressApi = () => {
  fromApi.value = true;
  loading.value = true;

  axios.get('http://localhost:3000/api/hotels')
    .then((res) => {
      data.value = res?.data?.results;
      loading.value = false;
    }).catch((err) => {
      data.value = [];
      loading.value = false;
      console.log(err);
    });
};

const singleData = reactive({
  id: 0,
  name: '',
  country: '',
  score: 0,
});

const getSingleDataFromExpressApi = (id) => {
  axios.get(`http://localhost:3000/api/hotel/${id}`)
    .then((res) => {
      if (res.data.results) {
        singleData.id = id;
        singleData.name = res.data.results.property.name;
        singleData.country = res.data.results.property.location.country;
        singleData.score = res.data.results.property.reviews.summary.score;
        centerDialogVisible.value = true;
      }
    }).catch((err) => {
      // eslint-disable-next-line no-alert
      window.alert(err.message);
      console.log(err);
    });
};

const showDataFromExpressApiFunc = () => {
  getDataFromExpressApi();
};

const update = (id) => {
  axios.put(`http://localhost:3000/api/hotel/${id}`, singleData)
    .then(() => {
      // eslint-disable-next-line no-alert
      window.alert('Updated');
      centerDialogVisible.value = false;
      showDataFromExpressApiFunc();
    }).catch((err) => {
      // eslint-disable-next-line no-alert
      window.alert(err.message);
      console.log(err);
      centerDialogVisible.value = false;
      showDataFromExpressApiFunc();
    });
};

const saveDataToDb = () => {
  loading.value = true;

  axios.post('http://localhost:3000/api/hotels', { hotels: data.value })
    .then(() => {
      // eslint-disable-next-line no-alert
      window.alert('Saved');
      showDataFromExpressApiFunc();
      loading.value = false;
    }).catch((err) => {
      showDataFromExpressApiFunc();
      loading.value = false;
      console.log(err);
      // eslint-disable-next-line no-alert
      window.alert(err.message);
    });
};

const deleted = (id) => {
  loading.value = true;

  axios.delete(`http://localhost:3000/api/hotels/${id}`)
    .then(() => {
      // eslint-disable-next-line no-alert
      window.alert('Deleted');
      showDataFromExpressApiFunc();
      loading.value = false;
    }).catch((err) => {
      showDataFromExpressApiFunc();
      loading.value = false;
      console.log(err);
      // eslint-disable-next-line no-alert
      window.alert(err.message);
    });
};

onMounted(() => {
  getDataFromExternalApi();
});

const setPage = (val) => {
  page.value = val;
};

const showNameOnlyFunc = () => {
  showNameOnly.value = !showNameOnly.value;
};

</script>

<style scoped>
.loading {
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
