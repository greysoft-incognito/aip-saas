<template>
  <q-page>
    <div class="q-pb-md q-pt-none">
      <StatsCards :items="stats" />
    </div>
    <UsersDialog
      :title="usersType"
      :type="usersType"
      :group="route.params.group"
      ref="usersDialogRef"
    >
      <template #title>
        <span class="text-weight-bold">
          {{ route.params.group.titleCase() }}
        </span>
        |
        <span class="text-capitalize">
          {{ titles[usersType] }}
        </span>
      </template>
    </UsersDialog>
  </q-page>
</template>

<script setup>
import { alova, useRequest } from "src/boot/alova";
import StatsCards from "src/components/StatsCards.vue";
import { computed, ref } from "vue";
import { useUserStore } from "stores/user-store";
import UsersDialog from "src/components/UsersDialog.vue";
import { useRoute } from "vue-router";
import { peopleGroups, peopleTitles, peopleIcons } from "src/plugins/constants";

const route = useRoute();
const usersType = ref("farmer");
const usersDialogRef = ref();

const userStore = useUserStore();
const user = computed({
  get: () => userStore.user,
  set: (u) => userStore.setUser(u),
});

const groups = peopleGroups;
const titles = peopleTitles;

const icons = peopleIcons;

const { data: dashboard } = useRequest(
  alova.Get(`overview`, {
    params: {
      user_id: user.value.id,
      group: route.params.group,
    },
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

const loadUsers = (type) => {
  usersType.value = type || "farmer";
  usersDialogRef.value.toggle();
};

const stats = computed(() =>
  (groups[route.params.group] || [route.params.group]).map((e) => {
    return {
      key: e,
      icon: icons[e] || icons[route.params.group],
      color: "blue",
      label: (titles[e] || e).titleCase(),
      count: dashboard.value.stats[e] || 0,
      click: () => loadUsers(e),
    };
  }),
);
</script>
