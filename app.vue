<script setup lang="ts">
import { Swiper, SwiperSlide } from 'swiper/vue';
import { Keyboard, Pagination, Navigation } from 'swiper/modules';

const slides = [
  'https://img.fcbayern.com/image/upload/f_auto,q_auto,w_1280/eCommerce/produkte/53822_3',
  'https://img.fcbayern.com/image/upload/f_auto,q_auto,w_1280/eCommerce/produkte/53822',
  'https://img.fcbayern.com/image/upload/f_auto,q_auto,w_1280/eCommerce/produkte/53822_1',
  'https://img.fcbayern.com/image/upload/f_auto,q_auto,w_1280/eCommerce/produkte/53822_2',
];
</script>

<template>
  <main>
    <h2>Gallery</h2>
    <p>
      Open this project on your phone, and notice how we cannot swipe between
      slides or scroll the page from within the Slider/3D preview.
    </p>

    <Swiper
      :keyboard="true"
      :css-mode="true"
      :lazy-preload-prev-next="2"
      :pagination="{ enabled: true, clickable: true }"
      :modules="[Pagination, Keyboard]"
    >
      <SwiperSlide>
        <iframe
          src="https://viewer.rooom.com/product/6e4b03fd05271278164ced0680b777?autostart=1"
          allowfullscreen
          :mozallowfullscreen="true"
          :webkitallowfullscreen="true"
        ></iframe>
      </SwiperSlide>

      <SwiperSlide v-for="slide in slides" :key="`slide-${slide}`">
        <img :src="slide" />
      </SwiperSlide>
    </Swiper>

    <p>
      We can fix the swiping issue by adding Arrows (left/right). But we cannot fix the scrolling issue.
    </p>

    <div>
      <br />
      <br />
      <br />
      <br />
      <br />
      <br />
      <br />
      <br />
      <br />
    </div>

    <p>
     FCB ROOM demo.
    </p>
  </main>
</template>

<style lang="scss">
iframe {
  width: 100%;
  height: 500px;
  border: none;
}
img {
  width: auto;
  height: 500px;
}
</style>
