<template>
  <v-icon
    :icon="raceTypeIconMap[race]"
    :style="{ backgroundColor: 'black', color: raceTypeColor[race] }"
  />
</template>
<script lang="ts">
import { mdiBomb, mdiHelp, mdiHuman, mdiPirate, mdiSkull } from "@mdi/js";
export const raceTypes = [
  "不明",
  "人間",
  "AC主義者",
  "バグ",
  "グノーシア",
] as const;
export type RaceType = (typeof raceTypes)[number];
export const raceTypeIconMap: { [type in RaceType]: string } = {
  不明: mdiHelp,
  人間: mdiHuman,
  AC主義者: mdiPirate,
  バグ: mdiBomb,
  グノーシア: mdiSkull,
};
export const raceTypeColor: { [type in RaceType]: string } = {
  不明: "white",
  人間: "white",
  AC主義者: "red",
  バグ: "red",
  グノーシア: "red",
};
</script>

<script setup lang="ts">
import { toRefs } from "vue";

const props = defineProps<{ race: RaceType }>();
const { race } = toRefs(props);
</script>
