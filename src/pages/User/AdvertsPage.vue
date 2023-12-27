<template>
  <q-page>
    <q-card class="q-mt-sm no-shadow" bordered>
      <TitleSection
        separator
        class="q-pa-none"
        icon="fas fa-rectangle-ad"
        title="Advert Requests"
      >
        <template #button>
          <q-btn
            label="Create New"
            color="primary"
            @click="$refs.createAdRef.open()"
          />
        </template>
      </TitleSection>

      <q-card-section class="q-pa-none q-ma-none">
        <q-table
          row-key="id"
          class="no-shadow no-border"
          rows-per-page-label="Items per page"
          :rows="slides"
          :columns="sales_column"
          :loading="loading"
          v-model:pagination="pagination"
          @request="onRequest"
        >
          <template v-slot:body-cell-title="props">
            <q-td :props="props">
              <q-item>
                <q-item-section avatar>
                  <q-avatar rounded>
                    <img :src="props.row.image_url" />
                  </q-avatar>
                </q-item-section>

                <q-item-section>
                  <q-item-label class="text-weight-bold">
                    {{ props.value }}
                  </q-item-label>
                </q-item-section>
              </q-item>
            </q-td>
          </template>
          <template v-slot:body-cell-status="props">
            <q-td :props="props" class="text-center">
              <q-chip
                class="text-white text-capitalize"
                :color="props.value == 'approved' ? 'positive' : 'negative'"
                :label="props.value || 'Pending'"
              ></q-chip>
            </q-td>
          </template>
          <template v-slot:body-cell-actions="props">
            <q-td :props="props" class="text-left q-gutter-xs">
              <q-btn
                rounded
                size="sm"
                color="green"
                label="Request"
                :loading="requesting"
                @click="sendRequest(props.row)"
                v-if="props.row.status === 'draft'"
              />
              <q-btn
                rounded
                size="sm"
                color="primary"
                label="Edit"
                @click="$refs.createAdRef.open(props.row)"
              />
              <ContentRemover
                round
                size="sm"
                base-url="account/advert/requests"
                :id="props.value"
                :list="slides"
                @deleted="(e, l) => (slides = l)"
              />
            </q-td>
          </template>
          <template v-slot:no-data>
            <div class="full-width row flex-center text-negative q-gutter-sm">
              <q-icon size="2em" name="sentiment_dissatisfied" />
              <span> Nothing to show for now. </span>
            </div>
          </template>
        </q-table>
      </q-card-section>
    </q-card>
    <CreateAdvertRequest
      ref="createAdRef"
      @created="(e) => slides.unshift(e)"
      @update:item="
        (e) => {
          slides = slides.map((x) => {
            x.id == e.id && (x = e);
            return x;
          });
        }
      "
    />
  </q-page>
</template>

<script setup>
import { usePagination } from "@alova/scene-vue";
import { alova, useRequest } from "src/boot/alova";
import TitleSection from "src/components/TitleSection.vue";
import CreateAdvertRequest from "src/components/CreateAdvertRequest.vue";
import ContentRemover from "src/components/Admin/ContentRemover.vue";
import { ref } from "vue";
import helpers from "src/plugins/helpers";

const createAdRef = ref();
const sales_column = [
  {
    name: "title",
    label: "Title",
    field: "title",
    sortable: true,
    align: "left",
  },
  {
    name: "duration",
    label: "Duration",
    field: "duration_readable",
    sortable: true,
    align: "center",
  },
  {
    name: "status",
    label: "Status",
    field: "status",
    sortable: true,
    align: "center",
  },
  {
    name: "actions",
    label: "Actions",
    field: "id",
    sortable: false,
    align: "right",
  },
];

const pagination = ref({
  sortBy: "desc",
  descending: false,
  page: 1,
  rowsPerPage: 30,
  // rowsNumber: xx if getting data from a server
});

const {
  loading,
  data: slides,
  onSuccess,
  send: loadItems,
} = usePagination(
  () =>
    alova.Get(`account/advert/requests`, {
      params: {
        page: pagination.value.page,
        limit: pagination.value.rowsPerPage,
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
  },
);

onSuccess(({ data }) => {
  pagination.value.page = data.meta.current_page;
  pagination.value.rowsNumber = data.meta.total;
  pagination.value.rowsPerPage = data.meta.per_page;
});

const onRequest = (props) => {
  const { page, rowsPerPage, sortBy, descending } = props.pagination;
  pagination.value.page = page;
  pagination.value.sortBy = sortBy;
  pagination.value.descending = descending;
  pagination.value.rowsPerPage = rowsPerPage;
  loadItems();
};

const {
  loading: requesting,
  onSuccess: onRequestSent,
  send: sendRequest,
} = useRequest(
  (params) =>
    alova.Put(`account/advert/requests/${params.id}`, {
      ...params,
      send_now: true,
    }),
  {
    immediate: false,
  },
);

onRequestSent(({ data }) => {
  helpers.notify(data.message, data.status);
  slides.value = slides.value.map((e) => {
    if (e.id === data.data.id) {
      e = data.data;
    }
    return e;
  });
});
</script>
