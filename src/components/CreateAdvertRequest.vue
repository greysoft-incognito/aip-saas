<template>
  <CustomDialog
    :title="`${request.id ? 'Update' : 'Create'} Advert Request: ${form.title}`"
    @before-hide="reset"
    v-model="toggle"
  >
    <q-form class="q-col-gutter-md row" @submit="1" style="min-width: 400px">
      <div class="col-12 flex justify-center">
        <TUploader
          v-model="image"
          label="Advert Image"
          accept=".jpg,.png,.jpeg"
          width="200px"
          :preview="request.image_url"
        />
      </div>
      <div class="col-12">
        <q-input
          filled
          lazy-rules
          hide-bottom-space
          type="text"
          v-model="form.title"
          label="Title"
          :error="!!errors.title"
          :error-message="errors.title"
        />
      </div>
      <div class="col-12">
        <q-input
          filled
          lazy-rules
          hide-bottom-space
          type="text"
          v-model="form.line1"
          label="Line 1"
          :error="!!errors.line1"
          :error-message="errors.line1"
        />
      </div>
      <div class="col-12">
        <q-input
          filled
          lazy-rules
          hide-bottom-space
          type="text"
          v-model="form.line2"
          label="Line 2"
          :error="!!errors.line2"
          :error-message="errors.line2"
        />
      </div>
      <div class="col-12">
        <q-input
          filled
          lazy-rules
          hide-bottom-space
          type="text"
          label="Link"
          v-model="form.line3"
          :error="!!errors.line3"
          :error-message="errors.line3"
        >
          <template #append>
            <q-btn
              round
              color="primary"
              icon="person"
              @click="
                form.line3 = $router.resolve({
                  name: 'profile',
                  params: { user: user.username },
                }).href
              "
            />
          </template>
        </q-input>
      </div>
      <div class="col-12">
        <q-input
          filled
          lazy-rules
          hide-bottom-space
          label="Duration (Hours)"
          type="number"
          v-model="form.duration"
          :error="!!errors.duration"
          :error-message="errors.duration"
        />
      </div>
      <div class="col-12">
        <q-checkbox
          v-model="form.hide_texts"
          label="Hide Texts"
          :true-value="1"
          :false-value="0"
        />
      </div>
    </q-form>
    <template #actions>
      <q-btn
        color="primary"
        :label="`${request.id ? 'Update' : 'Create'} Request`"
        :loading="loading"
        @click="create"
      />
    </template>
  </CustomDialog>
</template>

<script setup>
import { axios } from "src/boot/alova";
import { useForm } from "@alova/scene-vue";
import { computed, ref, watch, watchEffect } from "vue";
import CustomDialog from "./CustomDialog.vue";
import helpers from "src/plugins/helpers";
import TUploader from "./TUploader.vue";
import { useUserStore } from "src/stores/user-store";

const emit = defineEmits(["update:modelValue", "update:item", "created"]);
const props = defineProps({
  modelValue: {
    type: Boolean,
  },
  data: {
    type: Object,
    default: () => ({
      title: "",
      hide_texts: 0,
    }),
  },
});

const user = computed(() => useUserStore().user);
const image = ref(null);
const errors = computed(() => error.value?.errors || {});
const toggle = ref(props.modelValue);
const request = ref(props.data);

const open = (i) => {
  request.value = i || props.data;
  toggle.value = true;
};

const {
  form,
  error,
  reset,
  loading,
  send: create,
  onSuccess,
} = useForm(
  (fd) => {
    const a = axios.Post(
      `account/advert/requests${
        request.value.id ? "/" + request.value.id : ""
      }`,
      {
        ...fd,
        image: image.value,
        _method: request.value.id ? "PUT" : "POST",
      },
      {
        enableUpload: true,
        headers: {
          "Content-Type": "multipart/form-data",
        },
      },
    );
    a.meta = {
      noContentType: true,
    };
    return a;
  },
  {
    initialForm: {
      image: null,
      title: request.value.title,
      line1: request.value.line1,
      line2: request.value.line2,
      line3: request.value.line3,
      duration: request.value.duration || 24,
      hide_texts: request.value.hide_texts ? 1 : 0,
    },
    initialData: {},
    store: true,
  },
);

// Success handler
onSuccess(({ data }) => {
  // Notify the user
  helpers.notify(data.message, "success");
  if (request.value.id) {
    emit("update:item", data.data);
  } else {
    emit("created", data.data);
  }
  request.value = data.data;
});

watchEffect(() => {
  emit("update:modelValue", toggle.value);
});

watch(request, (i) => {
  form.value = {
    image: null,
    title: i.title,
    line1: i.line1,
    line2: i.line2,
    line3: i.line3,
    duration: i.duration || 24,
    hide_texts: i.hide_texts ? 1 : 0,
  };
});
defineExpose({ open });
</script>
