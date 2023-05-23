<template>
  <div
    :style="styleScrollbar"
    class="virtual-scrollbar d-flex flex-column align-items-center"
    v-if="haveScroll"
  >
    <div
      class="virtual-scrollbar__main position-relative h-100 w-100"
      @mousedown="mousedownMain($event)"
      ref="wrapper"
    >
      <div
        class="virtual-scrollbar__track position-absolute"
        :style="{
          height: !horizontal ? trackSize + 'px' : 'calc(100% - 2px)',
          width: horizontal ? trackSize + 'px' : 'calc(100% - 2px)',
          left: horizontal ? trackLeft + 'px' : '50%',
          top: !horizontal ? trackTop + 'px' : '50%',
          transform: horizontal ? 'translateY(-50%)' : 'translateX(-50%)',
        }"
        ref="track"
        @mousedown="
          $event.stopPropagation();
          mousedownTrack($event);
        "
      ></div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue';

const props = defineProps<{ horizontal: boolean; id: string }>();

// Ref element
const wrapper = ref(null);
const track = ref(null);

// Properties
const positionScroll = ref<number>(0);
const movingTrack = ref<boolean>(false);
const startX = ref<number>(0);
const startY = ref<number>(0);
const trackTop = ref<number>(0);
const trackLeft = ref<number>(0);
const clientSize = ref<number>(0);
const scrollSize = ref<number>(0);

const thumbSize = ref<number>(0);
const trackSize = ref<number>(0);
const elementScrollable = ref<HTMLElement>(null);
const haveScroll = ref<boolean>(false);

onMounted(() => {
  console.log('mount')
})

onUnmounted(() => {
  console.log('unmount')
})

/**
 * Xử lý khi click vào main scrollbar
 * @createdby ntdung 23.05.2023
 **/
function mousedownMain(e) {
  let size = track.value?.getBoundingClientRect();
  if (props.horizontal) {
    if (e.clientX < size.left) {
      var newLeft = trackLeft.value - trackSize.value;
      trackLeft.value = newLeft ? newLeft : 0;
    }
    if (e.clientX > size.right) {
      var newLeft = trackLeft.value + trackSize.value;
      trackLeft.value =
        newLeft <= thumbSize.value - trackSize.value
          ? newLeft
          : thumbSize.value - trackSize.value;
    }
    elementScrollable.value?.scrollLeft =
      (trackLeft.value / thumbSize.value) * scrollSize.value;
  } else {
    if (e.clientY < size.top) {
      var newTop = trackTop.value - trackSize.value;
      trackTop.value = newTop ? newTop : 0;
    }
    if (e.clientY > size.top) {
      var newTop = trackTop.value + trackSize.value;
      trackTop.value =
        newTop <= thumbSize.value - trackSize.value
          ? newTop
          : thumbSize.value - trackSize.value;
    }
    elementScrollable.value?.scrollTop =
      (trackTop.value / thumbSize.value) * scrollSize.value;
  }
}

/**
 * Xử lý khi nhấn xuống track
 * @createdby ntdung 23.05.2023
 **/
function mousedownTrack(e) {
  movingTrack.value = true;
  startY.value =
    e.clientY - parseInt(getComputedStyle(track.value?).top);
  startX.value =
    e.clientX - parseInt(getComputedStyle(track.value?).left);
}

/**
 * Đặt kicks thước cho scrollbar
 * createdby ntdung5 23.05.2023
 */
function setSizeScrollbar() {
  var scrollable = document.getElementById(props.id);
  if (props.horizontal) {
    clientSize.value = scrollable.clientWidth;
    scrollSize.value = scrollable.scrollWidth;
    thumbSize.value = wrapper.value?.getBoundingClientRect().width;
  } else {
    clientSize.value = scrollable.clientHeight;
    scrollSize.value = scrollable.scrollHeight;
    thumbSize.value = wrapper.value?.getBoundingClientRect().height;
  }
  trackSize.value = (clientSize.value / scrollSize.value) * thumbSize.value;
}

/**
 * Kiểm tra xem có scroll không
 * createdby ntdung5 23.05.2023
 */
function checkHaveScroll() {
  if (elementScrollable.value) {
    let scrollWidth = elementScrollable.value.scrollWidth;
    let width = elementScrollable.value.getBoundingClientRect().width;
    if (scrollWidth > width) {
      haveScroll.value = true;
    } else {
      haveScroll.value = false;
    }
  } else {
    haveScroll.value = false;
  }
}

/**
 * Style scrollbar
 * @createdby ntdung 23.05.2023
 **/
const styleScrollbar = computed(() => {
  if (props.horizontal) {
    return {
      height: '18px',
      width: '100%',
    };
  } else {
    return {
      width: '18px',
      height: '100%',
    };
  }
});

/**
 * Xóa vùng đang chọn
 * createdby ntdung5 13.06.2022
 */
function clearSelection() {
  if (window.getSelection()) {
    if (window.getSelection().empty) {
      window.getSelection().empty();
    } else if (window.getSelection().removeAllRanges) {
      window.getSelection().removeAllRanges();
    }
  }
  // else if (document.selection) {
  //   document.selection.empty();
  // }
}

/**
 * Kiểm tra đã ra ngoài wrapper
 * @createdby ntdung 23.05.2023
 **/
function outWrapper(e) {
  let offset = props.horizontal
    ? e.clientX - startX.value
    : e.clientY - startY.value;
  if (offset >= 0 && offset <= thumbSize.value - trackSize.value) {
    return {
      out: false,
    };
  }
  return {
    out: true,
  };
}
</script>

<style scoped lang="scss">
.virtual-scrollbar {
  &__main {
    background-color: #eee;
  }
  &__track {
    background-color: #ccc;
    cursor: pointer;
    top: 10px;
    left: 50%;
    &:hover {
      background-color: #aaa;
    }
    &:active {
      background-color: #888;
    }
  }
}
</style>