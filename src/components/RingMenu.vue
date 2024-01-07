<template>
  <div
    class="ring-menu"
    :style="{
      left: left + width / 2 + 'px',
      top: top + height / 2 + 'px',
      visibility: open ? 'visible' : 'hidden',
    }"
  >
    <div
      class="ring-menu-item"
      v-for="(item, index) in items"
      :key="index"
      :style="{
        transformOrigin: 'center center',
        transform:
          ' translate(' +
          itemPositions[index].left +
          'px, ' +
          itemPositions[index].top +
          'px)',
        opacity: open ? 1 : 0,
        zIndex: 1000,
      }"
    >
      <v-badge location="top center" :color="item.badgeColor">
        <template v-if="item.badge" v-slot:badge>
          <v-icon :icon="item.badge" />
        </template>
        <v-btn
          v-if="item.text"
          @click.stop="item.onClick"
          icon
          size="small"
          elevation="5"
        >
          {{ item.text }}
        </v-btn>
        <v-btn
          v-else
          @click.stop="item.onClick"
          :icon="item.icon"
          :color="item.iconColor"
          size="small"
          elevation="5"
        />
      </v-badge>
    </div>
  </div>
</template>
<script lang="ts">
export interface RingMenuItem {
  icon?: string;
  iconColor?: string;
  text?: string;
  badge?: string;
  badgeColor?: string;
  onClick: () => void;
}
</script>
<script setup lang="ts">
import { computed } from "vue";
import { watch } from "vue";
import { onMounted, ref, toRefs } from "vue";
import { VBtn } from "vuetify/components";

const props = defineProps<{
  open: boolean;
  items: RingMenuItem[];
  el: HTMLElement | null;
  radius: number;
}>();
const { open, el, items, radius } = toRefs(props);
const left = ref(0);
const top = ref(0);
const width = ref(0);
const height = ref(0);
const itemLength = computed(() => items.value.length);
const itemPositions = computed<{
  [index: number]: { left: number; top: number };
}>(() => {
  return items.value.reduce(
    (prev, _, index) => {
      const { itemWidth, itemHeight } = {
        itemWidth: 48,
        itemHeight: 48,
      };
      if (open.value) {
        prev[index] = {
          left: Math.floor(
            Math.cos((index * Math.PI * 2) / itemLength.value) *
              (radius.value + itemWidth / 2) -
              itemWidth / 2
          ),
          top: Math.floor(
            Math.sin((index * Math.PI * 2) / items.value.length) *
              (radius.value + itemHeight / 2) -
              itemHeight / 2
          ),
        };
      } else {
        prev[index] = { left: -itemWidth / 2, top: -itemHeight / 2 };
      }
      return prev;
    },
    {} as {
      [index: number]: { left: number; top: number };
    }
  );
});

function updatePosition() {
  if (!el.value) {
    return;
  }
  left.value = el.value.offsetLeft;
  top.value = el.value.offsetTop;
  width.value = el.value.offsetWidth;
  height.value = el.value.offsetHeight;
}
let observer: MutationObserver | null = null;
onMounted(() => {
  console.log("ring menu onMounted");
  if (el.value) {
    observer = new MutationObserver(updatePosition);
    observer.observe(el.value, { attributes: true });
  }
});
watch(el, (el) => {
  console.log("ring menu change element");
  observer && observer.disconnect();
  if (el) {
    observer = new MutationObserver(updatePosition);
    observer.observe(el, { attributes: true });
  }
});
watch(open, () => {
  console.log("ring menu change open");
  updatePosition();
});
</script>
<style lang="scss" scoped>
.ring-menu {
  position: absolute;
}
.ring-menu-item {
  position: absolute;
  transition: all 0.3s ease-out;
}
</style>
