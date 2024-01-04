<template>
  <v-avatar :color="statusTypeColor[status]">
    <v-icon :icon="statusIconTypeMap[status]" />
  </v-avatar>
</template>

<script lang="ts">
import { mdiHeart, mdiSnowflake, mdiVanish } from "@mdi/js";
export const statusTypes = ["通常", "コールドスリープ", "消滅"] as const;
export type StatusType = (typeof statusTypes)[number];
export const statusIconTypeMap: { [type in StatusType]: string } = {
  通常: mdiHeart,
  コールドスリープ: mdiSnowflake,
  消滅: mdiVanish,
};
const statusTypeColor: { [type in StatusType]: string } = {
  通常: "white",
  コールドスリープ: "indigo-darken-4",
  消滅: "grey-darken-3",
};
</script>

<script setup lang="ts">
import { toRefs } from "vue";

const props = defineProps<{ status: StatusType }>();
const { status } = toRefs(props);
</script>
