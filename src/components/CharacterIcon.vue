<template>
  <div
    class="character-icon"
    :style="{
      display: 'table',
      left: character.position.x + 'px',
      top: character.position.y + 'px',
      backgroundColor: characterColorMap[character.name],
      border: 'none',
      width: IconSize + 'px',
      height: IconSize + 'px',
      textAlign: 'center',
      borderRadius: IconSize / 2 + 'px',
      filter: character.active ? 'drop-shadow(0px 0px 5px #fff)' : '',
    }"
    @mouseenter="inRange = true"
    @mouseleave="inRange = false"
  >
    <div
      v-if="character.status != '通常'"
      class="filter"
      :style="{
        position: 'absolute',
        width: IconSize + 'px',
        height: IconSize + 'px',
        borderRadius: IconSize / 2 + 'px',
        backgroundColor: character.status == '消滅' ? 'darkred' : 'black',
        opacity: 0.8,
      }"
    ></div>
    <div
      :style="{
        display: 'table-cell',
        textAlign: 'center',
        verticalAlign: 'top',
        paddingTop: '2em',
      }"
    >
      <span
        :style="{
          backgroundColor: teamColor,
          border: 'solid 1px black',
          color: teamTextColor,
          fontSize: 'larger',
        }"
        >{{ character.name }}</span
      >
    </div>
    <StatusIcon
      :status="character.status"
      style="position: absolute; left: 0; bottom: 0"
    />
    <RaceIcon
      :race="character.race"
      style="position: absolute; right: 0; bottom: 0"
    />
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
  "ｸｸﾙｼｶ",
  "オトメ",
  "沙明",
  "レムナン",
  "夕里子",
] as const;
export type CharacterName = (typeof characterNames)[number];
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
  ｸｸﾙｼｶ: "crimson",
  オトメ: "pink",
  沙明: "black",
  レムナン: "white",
  夕里子: "gray",
};

export interface Position {
  x: number;
  y: number;
}
export interface Character {
  name: CharacterName;
  isAlly: boolean;
  isEnemy: boolean;
  position: Position;
  active: boolean;
  race: RaceType;
  status: StatusType;
}
export const IconSize = 100;
</script>
<script setup lang="ts">
import { computed, ref, toRefs } from "vue";
import RaceIcon, { RaceType } from "./RaceIcon.vue";
import StatusIcon, { StatusType } from "./StatusIcon.vue";

const props = defineProps<{
  character: Character;
}>();
const inRange = ref(false);
const { character } = toRefs(props);
const isEnemy = computed(() => {
  return (
    ["AC主義者", "バグ", "グノーシア"].includes(character.value.race) ||
    character.value.isEnemy
  );
});
const isAlly = computed(() => {
  return character.value.race == "人間";
});
const teamColor = computed(() => {
  if (isEnemy.value) {
    return "darkred";
  }
  if (isAlly.value) {
    return "limegreen";
  }
  return "white";
});
const teamTextColor = computed(() => {
  if (isEnemy.value || isAlly.value) {
    return "white";
  }
  return "black";
});
</script>

<style lang="scss" scoped>
.character-icon {
  position: absolute;
  user-select: none;
}
</style>
