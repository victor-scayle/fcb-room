<script setup lang="ts">
import { onBeforeUnmount, onMounted, ref } from 'vue';
import { Swiper, SwiperSlide } from 'swiper/vue';
import type { Swiper as SwiperClass } from 'swiper';
import { Keyboard, Pagination, Navigation } from 'swiper/modules';
import 'swiper/css';
import 'swiper/css/pagination';
import 'swiper/css/navigation';

const slides = [
  'https://img.fcbayern.com/image/upload/f_auto,q_auto,w_1280/eCommerce/produkte/53822_3',
  'https://img.fcbayern.com/image/upload/f_auto,q_auto,w_1280/eCommerce/produkte/53822',
  'https://img.fcbayern.com/image/upload/f_auto,q_auto,w_1280/eCommerce/produkte/53822_1',
  'https://img.fcbayern.com/image/upload/f_auto,q_auto,w_1280/eCommerce/produkte/53822_2',
];

const viewerProductId = '6e4b03fd05271278164ced0680b777';
const tapViewerActive = ref(false);
const scrollPointerEnabled = ref(true);
const tapViewerFrame = ref<HTMLIFrameElement | null>(null);
let scrollTimer: number | null = null;
let viewerScriptPromise: Promise<void> | null = null;
let tapViewerClickTimer: number | null = null;
let lastTapViewerClickAt = 0;
let touchZonesSwiper: SwiperClass | null = null;
let zoneTouchStartX = 0;
let zoneTouchStartY = 0;
let zoneTouchHandled = false;

type ViewerApi = {
  on: (event: string, callback: (...args: unknown[]) => void) => void;
};

const bindTapViewerEvents = (api: ViewerApi) => {
  api.on('click', () => {
    const now = Date.now();
    if (now - lastTapViewerClickAt < 350) {
      tapViewerActive.value = false;
      lastTapViewerClickAt = 0;
      if (tapViewerClickTimer !== null) {
        window.clearTimeout(tapViewerClickTimer);
        tapViewerClickTimer = null;
      }
      return;
    }

    lastTapViewerClickAt = now;
    if (tapViewerClickTimer !== null) {
      window.clearTimeout(tapViewerClickTimer);
    }
    tapViewerClickTimer = window.setTimeout(() => {
      lastTapViewerClickAt = 0;
      tapViewerClickTimer = null;
    }, 450);
  });
};

const onTouchZonesSwiper = (swiper: SwiperClass) => {
  touchZonesSwiper = swiper;
};

const onZoneTouchStart = (event: TouchEvent) => {
  const touch = event.touches[0];
  if (!touch) {
    return;
  }
  zoneTouchStartX = touch.clientX;
  zoneTouchStartY = touch.clientY;
  zoneTouchHandled = false;
};

const onZoneTouchMove = (event: TouchEvent) => {
  if (!touchZonesSwiper || zoneTouchHandled) {
    return;
  }
  const touch = event.touches[0];
  if (!touch) {
    return;
  }
  const deltaX = touch.clientX - zoneTouchStartX;
  const deltaY = touch.clientY - zoneTouchStartY;
  if (Math.abs(deltaX) > Math.abs(deltaY) && Math.abs(deltaX) > 12) {
    event.preventDefault();
    zoneTouchHandled = true;
    if (deltaX > 0) {
      touchZonesSwiper.slidePrev();
    } else {
      touchZonesSwiper.slideNext();
    }
  }
};

const onZoneTouchEnd = () => {
  zoneTouchHandled = false;
};

const loadViewerScript = () => {
  if (viewerScriptPromise) {
    return viewerScriptPromise;
  }
  viewerScriptPromise = new Promise((resolve, reject) => {
    if (document.querySelector('script[data-rooom-viewer-api="true"]')) {
      resolve();
      return;
    }
    const script = document.createElement('script');
    script.src = 'https://static.rooom.com/viewer-api/product-viewer-latest.min.js';
    script.async = true;
    script.dataset.rooomViewerApi = 'true';
    script.onload = () => resolve();
    script.onerror = () => reject(new Error('Failed to load rooom viewer API'));
    document.head.appendChild(script);
  });
  return viewerScriptPromise;
};

const initViewer = async (iframe: HTMLIFrameElement) => {
  if (iframe.dataset.viewerInitialized === 'true') {
    return;
  }
  await loadViewerScript();
  const ProductViewer = (
    window as Window & {
      ProductViewer?: new (
        el: HTMLIFrameElement
      ) => { init: (id: string, opts: Record<string, unknown>) => void };
    }
  ).ProductViewer;
  if (!ProductViewer) {
    return;
  }
  const viewer = new ProductViewer(iframe);
  viewer.init(viewerProductId, {
    onSuccess: (api: ViewerApi) => {
      if (iframe === tapViewerFrame.value) {
        bindTapViewerEvents(api);
      }
    },
    onError: () => undefined,
  });
  iframe.dataset.viewerInitialized = 'true';
};

const activateTapViewer = async () => {
  if (tapViewerFrame.value) {
    await initViewer(tapViewerFrame.value);
  }
  tapViewerActive.value = true;
};

const deactivateTapViewer = async () => {
  tapViewerActive.value = false;
};

const handleScroll = () => {
  tapViewerActive.value = false;
  scrollPointerEnabled.value = false;

  if (scrollTimer !== null) {
    window.clearTimeout(scrollTimer);
  }

  scrollTimer = window.setTimeout(() => {
    scrollPointerEnabled.value = true;
  }, 150);
};

onMounted(() => {
  scrollPointerEnabled.value = true;
  void loadViewerScript().then(() => {
    document
      .querySelectorAll<HTMLIFrameElement>('iframe[data-viewer-api="true"]:not([data-viewer-lazy="true"])')
      .forEach((iframe) => {
        void initViewer(iframe);
      });
  });
  window.addEventListener('scroll', handleScroll, { passive: true });
});

onBeforeUnmount(() => {
  window.removeEventListener('scroll', handleScroll);
  if (scrollTimer !== null) {
    window.clearTimeout(scrollTimer);
  }
  if (tapViewerClickTimer !== null) {
    window.clearTimeout(tapViewerClickTimer);
  }
});
</script>

<template>
  <main>
    <h2>Tap-to-Activate Overlay</h2>
    <p>
      Tap the viewer to activate it. Double-tap the viewer to deactivate. Any scroll
      re-disables interaction so page scroll and swipe between slides keep working.
    </p>

    <Swiper
      :keyboard="true"
      :css-mode="true"
      :lazy-preload-prev-next="2"
      :pagination="{ enabled: true, clickable: true }"
      :modules="[Pagination, Keyboard]"
    >
      <SwiperSlide>
        <div class="viewer-shell">
          <iframe
            ref="tapViewerFrame"
            class="viewer-frame"
            :class="{ 'viewer-frame--inactive': !tapViewerActive }"
            data-viewer-api="true"
            data-viewer-lazy="true"
            allow="autoplay; fullscreen; vr"
            allowfullscreen
            :mozallowfullscreen="true"
            :webkitallowfullscreen="true"
            @click.stop="deactivateTapViewer"
          ></iframe>
          <button
            v-if="!tapViewerActive"
            class="viewer-overlay"
            type="button"
            @click.stop="activateTapViewer"
            @touchend.stop.prevent="activateTapViewer"
          >
            Tap to activate viewer
          </button>
        </div>
      </SwiperSlide>

      <SwiperSlide v-for="slide in slides" :key="`tap-slide-${slide}`">
        <img :src="slide" />
      </SwiperSlide>
    </Swiper>

    <h2>Pointer-Events Toggle on Scroll</h2>
    <p>
      While the page is scrolling, the viewer is temporarily non-interactive. After a
      short pause, interaction is restored.
    </p>

    <Swiper
      :keyboard="true"
      :css-mode="true"
      :lazy-preload-prev-next="2"
      :pagination="{ enabled: true, clickable: true }"
      :navigation="true"
      :modules="[Pagination, Keyboard, Navigation]"
    >
      <SwiperSlide>
        <div class="viewer-shell">
          <iframe
            class="viewer-frame"
            :class="{ 'viewer-frame--inactive': !scrollPointerEnabled }"
            data-viewer-api="true"
            allow="autoplay; fullscreen; vr"
            allowfullscreen
            :mozallowfullscreen="true"
            :webkitallowfullscreen="true"
          ></iframe>
        </div>
      </SwiperSlide>

      <SwiperSlide v-for="slide in slides" :key="`scroll-slide-${slide}`">
        <img :src="slide" />
      </SwiperSlide>
    </Swiper>

    <h2>Touch Zones at the Edges</h2>
    <p>
      The top and bottom zones (20% each) allow scrolling the page while keeping the
      center area fully interactive for the viewer.
    </p>

    <Swiper
      @swiper="onTouchZonesSwiper"
      :keyboard="true"
      :css-mode="true"
      :lazy-preload-prev-next="2"
      :pagination="{ enabled: true, clickable: true }"
      :navigation="true"
      :modules="[Pagination, Keyboard, Navigation]"
    >
      <SwiperSlide>
        <div class="viewer-shell">
          <iframe
            class="viewer-frame"
            data-viewer-api="true"
            allow="autoplay; fullscreen; vr"
            allowfullscreen
            :mozallowfullscreen="true"
            :webkitallowfullscreen="true"
          ></iframe>
          <div
            class="viewer-scroll-zone viewer-scroll-zone--top"
            @touchstart="onZoneTouchStart"
            @touchmove="onZoneTouchMove"
            @touchend="onZoneTouchEnd"
            @touchcancel="onZoneTouchEnd"
          ></div>
          <div
            class="viewer-scroll-zone viewer-scroll-zone--bottom"
            @touchstart="onZoneTouchStart"
            @touchmove="onZoneTouchMove"
            @touchend="onZoneTouchEnd"
            @touchcancel="onZoneTouchEnd"
          ></div>
        </div>
      </SwiperSlide>

      <SwiperSlide v-for="slide in slides" :key="`zone-slide-${slide}`">
        <img :src="slide" />
      </SwiperSlide>
    </Swiper>

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
     FCB ROOM demo 2.
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
.viewer-shell {
  position: relative;
  width: 100%;
  height: 500px;
}
.viewer-frame {
  width: 100%;
  height: 100%;
  z-index: 1;
}
.viewer-frame--inactive {
  pointer-events: none;
}
.viewer-overlay {
  position: absolute;
  inset: 0;
  z-index: 2;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 0;
  background: rgba(0, 0, 0, 0.35);
  color: #fff;
  font-size: 16px;
  font-weight: 600;
  text-align: center;
  cursor: pointer;
}
.viewer-scroll-zone {
  position: absolute;
  left: 0;
  right: 0;
  height: 20%;
  z-index: 2;
  background: transparent;
  touch-action: pan-y;
}
.viewer-scroll-zone--top {
  top: 0;
}
.viewer-scroll-zone--bottom {
  bottom: 0;
}
</style>
