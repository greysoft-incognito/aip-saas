<template>
  <custom-dialog
    v-model="opened"
    @show="send"
    card-styles="min-width: 500px"
    card-section-height="550px"
  >
    <template #title>
      <slot name="title">
        <span class="text-weight-bold">
          {{ group ? group.titleCase() : "People" }}
        </span>
        |
        <span class="text-capitalize">
          {{ title || helpers.pluralize(type) }}
        </span>
      </slot>
    </template>

    <template #top>
      <q-card-section v-if="!!loading">
        <q-inner-loading showing color="primary" />
      </q-card-section>
    </template>

    <template #bottom>
      <q-card-section v-if="!users?.length && !loading">
        <div class="full-width column items-center text-info q-gutter-sm">
          <q-icon size="2em" name="sentiment_satisfied" />
          There are no
          {{
            (title || helpers.pluralize(peopleTitles[type])).toLowerCase()
          }}
          for now!
        </div>
      </q-card-section>
    </template>

    <q-list bordered separator v-if="users?.length">
      <q-item
        clickable
        v-ripple
        :key="user.id"
        :to="{ name: 'profile', params: { user: user.username } }"
        v-for="(user, i) in users"
        v-intersection="(e) => onIntersect(e, i === users.length - 1, i)"
      >
        <q-item-section avatar>
          <q-avatar>
            <img :src="user.avatar" />
          </q-avatar>
        </q-item-section>

        <q-item-section>
          <q-item-label class="text-weight-bold">
            {{ user.fullname }}
          </q-item-label>
          <q-item-label>
            {{ helpers.trunc(user.address, 30) }}
          </q-item-label>
        </q-item-section>
      </q-item>
    </q-list>
  </custom-dialog>
</template>

<script setup>
import { usePagination } from "@alova/scene-vue";
import { alova } from "src/boot/alova";
import helpers from "src/plugins/helpers";
import { ref } from "vue";
import CustomDialog from "./CustomDialog.vue";
import { peopleTitles } from "src/plugins/constants";

const props = defineProps({
  type: {
    type: String,
    default: "farmer",
  },
  group: String,
  title: String,
});

const {
  send,
  page,
  loading,
  data: users,
  isLastPage,
} = usePagination(
  (page, limit) =>
    alova.Get(`users`, {
      params: {
        page,
        limit,
        type: props.type,
        group: props.group ? props.group : undefined,
      },
      localCache: {
        mode: "placeholder",
        expire: 20000,
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
    immediate: false,
  },
);

const opened = ref(false);
const toggle = () => {
  opened.value = !opened.value;
};

const onIntersect = ({ isIntersecting }, isLast) => {
  if (isLast && isIntersecting && !isLastPage.value) {
    page.value++;
  }
};

defineExpose({ toggle });
</script>
