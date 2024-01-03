<template>
  <div
    class="character-icon"
    :style="{
      display: 'table',
      left: character.position.x + 'px',
      top: character.position.y + 'px',
      backgroundColor: characterColorMap[character.name],
      border: 'solid 5px black',
      width: IconSize + 'px',
      height: IconSize + 'px',
      textAlign: 'center',
      borderRadius: IconSize / 2 + 'px',
      filter: character.active ? 'drop-shadow(0px 0px 5px #fff)' : undefined,
    }"
    @mouseenter="inRange = true"
    @mouseleave="inRange = false"
  >
    <div
      :style="{
        display: 'table-cell',
        textAlign: 'center',
        verticalAlign: 'middle',
      }"
    >
      <span
        style="background-color: white; border: solid 1px black; color: black"
        >{{ character.name }}</span
      >
    </div>
  </div>
</template>

<script lang="ts">
export const characterNames = [
  "あなた",
  "ジナ",
  "SQ",
  "ラキオ",
  "ステラ",
  "しげみち",
  "セツ",
  "シピ",
  "コメット",
  "ジョナス",
  "ククルシカ",
  "オトメ",
  "沙明",
  "レムナン",
  "夕里子",
] as const;
export const characterColorMap: { [name in CharacterName]: string } = {
  あなた: "palegreen",
  ジナ: "purple",
  SQ: "red",
  ラキオ: "blue",
  ステラ: "green",
  しげみち: "yellowgreen",
  セツ: "cyan",
  シピ: "orange",
  コメット: "yellow",
  ジョナス: "brown",
  ククルシカ: "crimson",
  オトメ: "pink",
  沙明: "black",
  レムナン: "white",
  夕里子: "gray",
};
export type CharacterName = (typeof characterNames)[number];
export interface Position {
  x: number;
  y: number;
}
export interface Character {
  name: CharacterName;
  callDocter: boolean;
  callEngineer: boolean;
  isDoctor: boolean;
  isEngineer: boolean;
  isGurdian: boolean;
  isCareTaker: boolean;
  isAcBeliever: boolean;
  isBug: boolean;
  isGnosia: boolean;
  position: Position;
  active: boolean;
}
export const IconSize = 100;
</script>
<script setup lang="ts">
import { ref, toRefs } from "vue";

const props = defineProps<{
  character: Character;
}>();
const inRange = ref(false);
const { character } = toRefs(props);
</script>

<style lang="scss" scoped>
.character-icon {
  position: absolute;
  user-select: none;
}
</style>
