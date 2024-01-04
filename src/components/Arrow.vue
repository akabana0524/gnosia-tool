<template>
  <div
    class="arrow"
    :style="{
      position: 'absolute',
      left: 0,
      top: 0,
      transformOrigin: 'left center',
      transform:
        'translate(' +
        position.x +
        'px, ' +
        (position.y - (strokeWidth * arrowHeadScale) / 2) +
        'px) rotate(' +
        degree +
        'deg) translate(' +
        paddingStart +
        'px, 0px)',
    }"
  >
    <svg
      ref="svg"
      :width="Math.max(length - paddingStart - paddingEnd, 0) + 'px'"
      :height="strokeWidth * arrowHeadScale + 'px'"
      style="vertical-align: top"
    >
      <defs>
        <marker
          :id="'arrow-' + randomId"
          viewBox="0 0 10 10"
          refX="5"
          refY="5"
          :markerWidth="arrowHeadScale"
          :markerHeight="arrowHeadScale"
          orient="auto-start-reverse"
        >
          <path d="M 0 0 L 10 5 L 0 10 z" :fill="strokeColor" />
        </marker>
      </defs>
      <line
        :x1="strokeWidth / 2"
        :y1="(strokeWidth * arrowHeadScale) / 2"
        :x2="
          length -
          (strokeWidth * arrowHeadScale) / 2 -
          (paddingStart + paddingEnd)
        "
        :y2="(strokeWidth * arrowHeadScale) / 2"
        :stroke="strokeColor"
        :stroke-width="strokeWidth"
        stroke-linecap="round"
        :marker-end="'url(#arrow-' + randomId + ')'"
      ></line>
    </svg>
  </div>
</template>
<script setup lang="ts">
import { onMounted, reactive, ref } from "vue";
import { computed } from "vue";
import { toRefs } from "vue";

const props = withDefaults(
  defineProps<{
    from: HTMLElement;
    to: HTMLElement;
    strokeColor: string;
    paddingStart?: number;
    paddingEnd?: number;
  }>(),
  { paddingStart: 0, paddingEnd: 0 }
);
const randomId = crypto.randomUUID();
const { from, to, strokeColor, paddingStart, paddingEnd } = toRefs(props);
const arrowHeadScale = ref(6);
const strokeWidth = ref(5);
const mounted = ref(false);
let observer = null;
interface Rect {
  x: number;
  y: number;
  w: number;
  h: number;
}
function defaultRect(): Rect {
  return {
    x: 0,
    y: 0,
    w: 0,
    h: 0,
  };
}
function calcRect(el: HTMLElement) {
  return {
    x: el.offsetLeft,
    y: el.offsetTop,
    w: el.clientWidth,
    h: el.clientHeight,
  };
}
function copyTo(src: Rect, dst: Rect) {
  dst.x = src.x;
  dst.y = src.y;
  dst.w = src.w;
  dst.h = src.h;
}
const fromRect = reactive<Rect>(defaultRect());
const toRect = reactive<Rect>(defaultRect());
function setFromElement() {
  copyTo(calcRect(from.value), fromRect);
  copyTo(calcRect(to.value), toRect);
}
onMounted(() => {
  mounted.value = true;
  setFromElement();
  observer = new MutationObserver(setFromElement);
  observer.observe(from.value, { attributes: true });
  observer.observe(to.value, { attributes: true });
});
const position = computed(() => {
  const result = {
    x: fromRect.x + fromRect.w / 2,
    y: fromRect.y + fromRect.h / 2,
  };
  return result;
});
const vector = computed(() => {
  if (!mounted.value) {
    return { x: 1, y: 0 };
  }
  const { x: fromCenterX, y: fromCenterY } = position.value;
  const toCenterX = toRect.x + toRect.w / 2;
  const toCenterY = toRect.y + toRect.h / 2;
  const x = toCenterX - fromCenterX;
  const y = toCenterY - fromCenterY;
  return { x, y };
});
const length = computed(() => {
  const result = Math.sqrt(vector.value.x ** 2 + vector.value.y ** 2);
  return result;
});
const degree = computed(() => {
  var dot = 1 * vector.value.x + 0 * vector.value.y;
  let cross = 1 * vector.value.y - 0 * vector.value.x;

  var absA = Math.sqrt(0 ** 2 + 1 ** 2);
  var absB = Math.sqrt(vector.value.x ** 2 + vector.value.y ** 2);

  var cos = dot / (absA * absB);
  var sin = cross / (absA * absB);

  var radian = Math.acos(cos) * (sin > 0 ? 1 : -1);
  var degree = (radian * 180) / Math.PI;
  return degree;
});
</script>

<style lang="scss">
.arrow:hover {
  filter: drop-shadow(0px 0px 5px #fff);
}
</style>
