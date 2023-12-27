<template>
  <q-page>
    <div class="q-pa-md">
      <StatsCards :items="stats" />
      <div class="row q-col-gutter-md q-my-sm justify-center">
        <div class="col-12">
          <MainSlider />
        </div>
      </div>
    </div>
    <div class="q-pa-md bg-white q-mt-md">
      <h5
        class="text-center column justify-center items-center text-weight-bold q-my-xs"
      >
        Announcements
        <q-btn
          rounded
          label="Create New"
          color="primary"
          @click="$refs.createAnnouncementRef.open()"
        />
      </h5>
      <AnnouncementSlider
        editable
        @edit="$refs.createAnnouncementRef.open($event)"
      />
    </div>
    <UsersDialog :type="usersType" ref="usersDialogRef" />
    <ForecastDialog ref="forecastDialogRef" />
    <DiseaseDialog ref="diseaseDialogRef" />
    <SoilRequirementDialog ref="soilRequirementDialogRef" />
    <!-- ======= -->
    <CreateAnnouncement ref="createAnnouncementRef" />
  </q-page>
</template>

<script setup>
import { alova, useRequest } from "src/boot/alova";
import CreateAnnouncement from "src/components/Admin/CreateAnnouncement.vue";
import AnnouncementSlider from "src/components/AnnouncementSlider.vue";
import ForecastDialog from "src/components/ForecastDialog.vue";
import MainSlider from "src/components/MainSlider.vue";
import StatsCards from "src/components/StatsCards.vue";
import { computed, onBeforeUnmount, ref } from "vue";
import { useUserStore } from "stores/user-store";
import DiseaseDialog from "src/components/DiseaseDialog.vue";
import helpers from "src/plugins/helpers";
import SoilRequirementDialog from "src/components/SoilRequirementDialog.vue";
import UsersDialog from "src/components/UsersDialog.vue";

const usersType = ref("farmer");
const usersDialogRef = ref();
const forecastDialogRef = ref();
const diseaseDialogRef = ref();
const soilRequirementDialogRef = ref();
const createAnnouncementRef = ref();

const userStore = useUserStore();
const user = computed({
  get: () => userStore.user,
  set: (u) => userStore.setUser(u),
});

// Define WEATHER_API_KEY key
const WAPI_KEY = process.env.WEATHER_API_KEY;
// Request Weather info
const { data: weather, send: checkWeather } = useRequest(
  () => {
    const instance = alova.Get(
      `https://api.weatherapi.com/v1/current.json?q=${user.value.city}&key=${WAPI_KEY}`,
      {
        localCache: {
          mode: "placeholder",
          expire: 3.6e6,
        },
      },
    );
    !instance.meta ? (instance.meta = {}) : null;
    instance.baseURL = "";
    instance.meta.noAuth = true;
    return instance;
  },
  {
    initialData: {
      current: { temp_c: 0 },
    },
  },
);

const weatherInterval = setInterval(() => {
  checkWeather();
}, 3.6e6);

const { data: dashboard } = useRequest(
  alova.Get(`overview`, {
    params: { user_id: user.value.id },
    localCache: {
      mode: "placeholder",
      expire: 3.6e6,
    },
    transformData: (data) => data.data,
  }),
  {
    initialData: {
      stats: {},
    },
  },
);

const loadUsers = (type = "farmer") => {
  usersType.value = type;
  usersDialogRef.value.toggle();
};

const stats = computed(() => [
  {
    to: { name: "people", params: { group: "farmers" } },
    icon: "people",
    color: "blue",
    label: "Farmers",
    count: dashboard.value.stats.farmers || 0,
  },
  {
    to: { name: "people", params: { group: "aggregators" } },
    icon: "fas fa-list",
    color: "red",
    label: "Aggregators",
    count: dashboard.value.stats.aggregators || 0,
  },
  {
    to: { name: "people", params: { group: "suppliers" } },
    icon: "fas fa-boxes-packing",
    color: "green",
    label: "Input Suppliers",
    count: dashboard.value.stats.suppliers || 0,
  },
  {
    to: { name: "people", params: { group: "processors" } },
    icon: "fas fa-microchip",
    color: "cyan",
    label: "Processors",
    count: dashboard.value.stats.processors || 0,
  },
  {
    to: { name: "people", params: { group: "offtakers" } },
    icon: "fas fa-plane-departure",
    color: "pink",
    label: "Offtakers",
    count: dashboard.value.stats.offtakers || 0,
  },
  {
    icon: "fas fa-truck-fast",
    color: "orange",
    label: "Logistics",
    count: dashboard.value.stats.logistics || 0,
    click: () => loadUsers("logistics"),
  },
  {
    icon: "fas fa-building-columns",
    color: "teal",
    label: "Research Institutions",
    count: dashboard.value.stats.researchers || 0,
    click: () => loadUsers("researchers"),
  },
  {
    sup: "° Celsius",
    icon: weather.value.current.condition?.icon
      ? `img:${weather.value.current.condition.icon}`
      : "cloud",
    color: "light-blue",
    label: "Weather Forecast",
    count: weather.value.current.temp_c,
    tooltip:
      weather.value.current.condition?.text + " (Click for more information)",
    click: () => forecastDialogRef.value.toggle(),
  },
  {
    icon: "coronavirus",
    color: "purple",
    danger: dashboard.value.stats.disease_outbreaks > 0,
    label: "Disease Outbreak",
    count: dashboard.value.stats.disease_outbreaks || 0,
    tooltip: "Click for more information",
    click: () => diseaseDialogRef.value.toggle(),
  },
  ...(dashboard.value.stats.current_prices || []).map((e) => {
    return {
      icon: e.icon || "fa-solid fa-wheat-awn",
      color: "blue-grey",
      label: `Grade ${e.item} (${helpers.money(
        e.price || 0,
      )}/bag | ${helpers.money(e.price_tons || 0)}/ton)`,
      count: `${parseInt(e.quantity || 0).toLocaleString(
        "us",
      )} Bags | ${parseInt(e.quantity_tons || 0).toLocaleString("us")} Tons`,
      count_size: "1.1em",
    };
  }),
]);

onBeforeUnmount(() => {
  clearInterval(weatherInterval);
});
</script>
