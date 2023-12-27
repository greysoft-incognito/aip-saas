<template>
  <q-page>
    <q-card class="q-mt-sm no-shadow" bordered>
      <TitleSection separator title="Soil Requirements" class="q-pa-none">
      </TitleSection>

      <q-card-section class="q-pa-none q-ma-none">
        <q-table
          row-key="id"
          class="no-shadow no-border"
          rows-per-page-label="Items per page"
          :rows="sales_data"
          :columns="sales_column"
          :loading="loading"
          v-model:pagination="pagination"
          @request="onRequest"
        >
          <template v-slot:body-cell-crop="props">
            <q-td :props="props">
              <q-item-section>
                <q-item-label>
                  {{ props.value }}
                </q-item-label>
                <q-item-label>
                  Applies to:
                  <span class="text-weight-bold">
                    {{ props.row.location }}
                  </span>
                </q-item-label>
              </q-item-section>
            </q-td>
          </template>
          <template v-slot:body-cell-water="props">
            <q-td :props="props">
              <q-item-section>
                <q-item-label>
                  {{ props.value }}
                </q-item-label>
                <q-item-label v-if="props.row.period">
                  Suggested Planting Period:
                  <span class="text-weight-bold">
                    {{ props.row.period }}
                  </span>
                </q-item-label>
              </q-item-section>
            </q-td>
          </template>
          <template v-slot:body-cell-details="props">
            <q-td :props="props" class="text-left">
              {{ helpers.trunc(props.value, 50) }}
            </q-td>
          </template>
          <template v-slot:body-cell-temperature="props">
            <q-td :props="props">
              <q-chip
                color="primary"
                class="text-white text-capitalize q-ma-none"
                :label="props.value"
              ></q-chip>
            </q-td>
          </template>
          <template v-slot:no-data>
            <div class="full-width row flex-center text-negative q-gutter-sm">
              <q-icon size="2em" name="sentiment_dissatisfied" />
              <span>
                {{
                  search
                    ? `No results found for search: "${search}".`
                    : "Nothing to show for now."
                }}
              </span>
            </div>
          </template>
        </q-table>
      </q-card-section>
    </q-card>
  </q-page>
</template>

<script setup>
import { usePagination } from "@alova/scene-vue";
import { alova } from "src/boot/alova";
import TitleSection from "src/components/TitleSection.vue";
import helpers from "src/plugins/helpers";
import { ref } from "vue";

const sales_column = [
  {
    name: "crop",
    label: "Crop",
    field: "crop",
    sortable: false,
    align: "left",
  },
  {
    name: "details",
    label: "Info",
    field: "details",
    sortable: true,
    align: "left",
  },
  {
    name: "temperature",
    label: "Temperature",
    field: "temperature",
    sortable: true,
    align: "right",
    classes: "text-bold",
  },
  {
    name: "water",
    label: "Requirements",
    field: "water",
    sortable: true,
    align: "left",
  },
];

const search = ref("");
const searching = ref(false);
const pagination = ref({
  sortBy: "desc",
  descending: false,
  page: 1,
  rowsPerPage: 30,
  // rowsNumber: xx if getting data from a server
});
const {
  loading,
  data: sales_data,
  onSuccess,
  onError,
  send: loadItems,
} = usePagination(
  () =>
    alova.Get(`soil/requirements`, {
      params: {
        page: pagination.value.page,
        limit: pagination.value.rowsPerPage,
        search: search.value || undefined,
        [`order[${pagination.value.sortBy}]`]: pagination.value.descending
          ? "desc"
          : "asc",
      },
      localCache: {
        mode: "placeholder",
        expire: 3.6e6,
      },
    }),
  {
    total: (data) => data.meta.total,
    append: false,
    initialData: {
      meta: { total: 0 },
      data: [],
    },
    initialPageSize: 30,
    watchingStates: [searching],
  },
);

onSuccess(({ data }) => {
  searching.value = false;
  pagination.value.page = data.meta.current_page;
  pagination.value.rowsNumber = data.meta.total;
  pagination.value.rowsPerPage = data.meta.per_page;
});

onError(() => {
  searching.value = false;
});

const onRequest = (props) => {
  const { page, rowsPerPage, sortBy, descending } = props.pagination;
  pagination.value.page = page;
  pagination.value.sortBy = sortBy;
  pagination.value.descending = descending;
  pagination.value.rowsPerPage = rowsPerPage;
  loadItems();
};
</script>

<style lang="sass" scoped>
.custom-toggle
  border: 1px solid $primary
</style>
