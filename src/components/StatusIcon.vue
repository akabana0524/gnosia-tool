<template>
  <v-icon :icon="statusIconTypeMap[status]" style="background-color: black" />
</template>

<script lang="ts">
import { mdiArmFlex, mdiSnowflake, mdiVanish } from "@mdi/js";
export const statusTypes = ["通常", "コールドスリープ", "消滅"] as const;
export type StatusType = (typeof statusTypes)[number];
export const statusIconTypeMap: { [type in StatusType]: string } = {
  通常: mdiArmFlex,
  コールドスリープ: mdiSnowflake,
  消滅: mdiVanish,
};
</script>

<script setup lang="ts">
import { toRefs } from "vue";

const props = defineProps<{ status: StatusType }>();
const { status } = toRefs(props);
</script>
