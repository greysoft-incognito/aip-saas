<template>
  <swiper-container
    :loop="true"
    :css-mode="true"
    :navigation="true"
    :centered-slides="true"
    :autoplay-delay="5000"
    :autoplay-pause-on-mouse-enter="true"
    :autoplay-disable-on-interaction="true"
    v-if="(slides || data || []).length"
  >
    <swiper-slide :key="slide.id" v-for="slide in slides || data">
      <div
        class="relative full-width q-pa-md q-ma-sm"
        :style="{
          height: '180px',
          zIndex: 2,
          backgroundSize: 'contain',
          backgroundImage: `url(${slide.image_url})`,
          backgroundRepeat: 'no-repeat',
          backgroundPosition: 'center center',
        }"
      >
        <div
          class="absolute-top-left full-width column items-center q-px-md full-height"
          :class="`justify-${!slide.hide_texts ? 'center' : 'end'}`"
        >
          <div
            class="absolute flex justify-center items-center"
            v-if="!slide.loaded"
            style="top: 0; bottom: 0; left: 0; right: 0"
          >
            <q-spinner
              size="xl"
              color="primary"
              class="q-ma-sm"
              style="opacity: 0.5"
            />
          </div>
          <template v-if="!slide.hide_texts">
            <h3
              class="text-h3 text-weight-bold text-center text-grey-4 q-my-xs"
              v-html="helpers.nlText(slide.title, 5)"
            ></h3>
            <h4 class="text-h4 text-weight-bold text-center text-white q-my-xs">
              {{ slide.line1 }}
            </h4>
            <h6 class="text-h6 text-white text-weight-bold text-center q-my-xs">
              {{ slide.line2 }}
            </h6>
          </template>
          <div
            class="full-width flex justify-center"
            :class="{ 'q-mb-sm': slide.hide_texts }"
            v-if="slide.line3"
          >
            <q-btn
              color="primary"
              label="Visit Advertiser"
              :to="slide.line3"
              :href="slide.line3.includes('http') ? slide.line3 : undefined"
              :target="slide.line3.includes('http') ? '_blank' : undefined"
            />
          </div>
          <img
            :src="slide.image_url"
            style="height: 0px; width: 0px"
            v-if="!slide.loaded"
            @load="slide.loaded = true"
          />
        </div>
      </div>
      <div
        class="absolute"
        :style="{
          top: 0,
          left: 0,
          right: 0,
          scale: '1.1',
          bottom: 0,
          zIndex: 1,
          filter: 'blur(20px) brightness(0.5)',
          backgroundSize: 'cover',
          backgroundImage: `url(${slide.image_url})`,
          backgroundRepeat: 'no-repeat',
          backgroundPosition: 'center center',
        }"
      ></div>
    </swiper-slide>
  </swiper-container>
</template>

<script setup>
import { alova, useRequest } from "src/boot/alova";
import helpers from "src/plugins/helpers";

const props = defineProps({
  slides: {
    type: Array,
  },
});

const { data } = useRequest(
  alova.Get(`slides`, {
    params: {
      ads: true,
    },
    localCache: {
      mode: "placeholder",
      expire: 3.6e6,
    },
    transformData: (data) => data.data,
  }),
  {
    initialData: [],
    immediate: !props.slides,
  },
);
</script>

<style scoped lang="scss">
swiper-container {
  width: 100%;
  height: 100%;

  &::part(pagination) {
    .swiper-button-prev {
      background-color: red;
    }
  }
  &::part(wrapper) {
    height: 265px;
    overflow-y: hidden;
  }
  &::part(button-next),
  &::part(button-prev) {
    &:hover {
      background-color: rgba(28, 27, 27, 0.515);
    }
    color: rgba(0, 0, 0, 0);
    height: 100%;
    top: 0;
    bottom: 0;
    position: absolute;
    margin-top: 0;
    width: 50px;
  }
  &::part(button-next) {
    right: 0;
    [viewBox] {
      display: none !important;
    }
  }
  &::part(button-prev) {
    left: 0;
  }
}
swiper-slide {
  text-align: center;
  font-size: 18px;
  background: #fff;
  display: flex;
  justify-content: center;
  align-items: center;
}
swiper-slide img {
  display: block;
  width: 100%;
  height: 100%;
  object-fit: cover;
}
</style>
