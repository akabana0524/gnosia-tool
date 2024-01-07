<template>
  <v-layout full-height>
    <v-app-bar>
      <v-app-bar-nav-icon variant="text" @click.stop="drawer = !drawer" />
      <v-toolbar-title>Gnosia whiteboard</v-toolbar-title>
      <v-spacer />
      <v-select
        label="グノーシア人数"
        v-model.number="numGnosia"
        :items="[...Array(7).keys()]"
        style="max-width: 200px"
      />
      <v-btn
        v-for="(v, k) in enableMap"
        :key="k"
        icon
        variant="flat"
        @click="v.value = !v.value"
        :color="v.value ? 'primary' : undefined"
      >
        {{ k }}
      </v-btn>
      <v-btn variant="flat" :icon="mdiReload" @click.stop="reset" />
      <v-btn
        :icon="darkTheme ? mdiWeatherNight : mdiWeatherSunny"
        @click.stop="darkTheme = !darkTheme"
      />
    </v-app-bar>
    <v-main>
      <v-navigation-drawer v-model="drawer">
        <v-list :items="['hoge', 'fuga']"></v-list>
      </v-navigation-drawer>

      <div
        id="whiteboard"
        @mousemove="mousemove"
        @mouseup="mouseup"
        style="user-select: none"
        @click="clickCharacter(null)"
      >
        <div style="display: inline-block; width: 80%; height: 100%">
          <div style="width: 100%; height: 60%">
            <div style="display: inline-block; width: 70%; height: 100%">
              <div
                class="role"
                :style="{
                  width: '100%',
                  height: '100%',
                  backgroundColor: 'darkgray',
                }"
                ref="crew"
              >
                乗員
              </div>
            </div>
            <div style="display: inline-block; width: 30%; height: 100%">
              <div
                class="role"
                :style="{
                  display: 'inline-block',
                  width: '100%',
                  height: '50%',
                  backgroundColor: 'darkgoldenrod',
                  visibility: enableGurdian ? 'visible' : 'hidden',
                }"
              >
                守護天使
              </div>
              <div
                class="role"
                :style="{
                  display: 'inline-block',
                  width: '100%',
                  height: '50%',
                  backgroundColor: 'darkgreen',
                  visibility: enableCareTaker ? 'visible' : 'hidden',
                }"
              >
                留守番
              </div>
            </div>
          </div>
          <div
            class="role"
            :style="{
              width: '100%',
              height: '20%',
              backgroundColor: 'darkblue',
              visibility: enableEngineer ? 'visible' : 'hidden',
            }"
          >
            エンジニア
          </div>
          <div
            class="role"
            :style="{
              width: '100%',
              height: '20%',
              backgroundColor: 'darkgreen',
              visibility: enableDoctor ? 'visible' : 'hidden',
            }"
          >
            ドクター
          </div>
        </div>
        <div style="display: inline-block; width: 20%; height: 100%">
          <div
            class="role"
            style="width: 100%; height: 100%; background-color: black"
          >
            未使用
          </div>
        </div>
        <CharacterIcon
          v-for="(character, index) in characters"
          :ref="(el) => setCharacterRef(el as CharacterIconType, index)"
          :key="index"
          :character="character"
          @mousedown="mouswdownCharacter(character.name)"
          @click.stop="clickCharacter(character.name)"
          :style="{ zIndex: character.zIndex }"
        />
        <Arrow
          v-for="(relation, index) in relations"
          :key="index"
          :from="getCharacterIcon(relation.from)"
          :to="getCharacterIcon(relation.to)"
          :stroke-color="getStrokeColor(relation.type)"
          :padding-start="IconSize / 2"
          :padding-end="IconSize / 2"
          @dblclick="removeRelation(index)"
        />
        <RingMenu
          v-model:open="isRingMenuOpen"
          :el="ringMenuTarget"
          :items="ringMenuItems"
          :radius="70"
          :sytle="{ zIndex: 1000 }"
        />
      </div>
    </v-main>
  </v-layout>
</template>

<script setup lang="ts">
import { computed, reactive, ref, watch } from "vue";
import { useTheme } from "vuetify";
import {
  mdiWeatherSunny,
  mdiWeatherNight,
  mdiReload,
  mdiArrowTopRightThin,
  mdiPen,
} from "@mdi/js";
import Arrow from "./components/Arrow.vue";

const relationTypes = ["疑う", "庇う", "人間だ", "グノーシアだ"] as const;
type RelationType = (typeof relationTypes)[number];
interface Relation {
  from: CharacterName;
  to: CharacterName;
  type: RelationType;
}

/*
 * 初期設定を覚えてオンオフでレイアウトを切り替えれる
 * グノーシアの人数、状況がわかる
 *
 */
const drawer = ref(false);

const darkTheme = ref(true);
const theme = useTheme();
watch(
  darkTheme,
  (darkTheme) => {
    theme.global.name.value = darkTheme ? "dark" : "light";
  },
  { immediate: true }
);

import CharacterIcon, {
  CharacterName,
  characterNames,
  Character,
  IconSize,
} from "./components/CharacterIcon.vue";
import { onMounted } from "vue";
import {
  RaceType,
  raceTypeColor,
  raceTypeIconMap,
  raceTypes,
} from "./components/RaceIcon.vue";
import {
  StatusType,
  statusIconTypeMap,
  statusTypeColor,
  statusTypes,
} from "./components/StatusIcon.vue";
import RingMenu, { RingMenuItem } from "./components/RingMenu.vue";
import { nextTick } from "vue";
type CharacterIconType = InstanceType<typeof CharacterIcon>;
const crew = ref<HTMLElement | null>(null);
const characters = reactive<Character[]>(
  characterNames.map((name) => ({
    name,
    isAlly: false,
    isEnemy: false,
    race: "不明",
    status: "通常",
    position: { x: 0, y: 0 },
    active: false,
    zIndex: 100,
  }))
);
const relations = reactive<Relation[]>([]);
function removeRelation(index: number) {
  relations.splice(index, 1);
}

const characterIconRefs = ref<(HTMLElement | null)[]>(
  characterNames.map((_) => null)
);
function setCharacterRef(characterIcon: CharacterIconType, index: number) {
  characterIconRefs.value[index] = characterIcon.$el;
}
function getCharacterIcon(characterName: CharacterName): HTMLElement {
  return characterIconRefs.value[
    characters.findIndex((character) => character.name == characterName)
  ]!;
}
function getStrokeColor(relationType: RelationType) {
  switch (relationType) {
    case "庇う":
      return "cyan";
    case "疑う":
      return "darkmagenta";
    case "人間だ":
      return "lightgray";
    case "グノーシアだ":
      return "red";
  }
}
const isRingMenuOpen = ref(false);
const ringMenuItems = computed<RingMenuItem[]>(() => [
  ...statusTypes.map((statusType) => ({
    icon: statusIconTypeMap[statusType],
    iconColor: statusTypeColor[statusType],
    badge: mdiPen,
    badgeColor: "primary",
    onClick: () => {
      setCurrentCharacterStatus(statusType);
      clickCharacter(currentCharacter.value!.name);
    },
  })),
  ...raceTypes
    .filter((raceType) => enableAcBeliever.value || raceType != "AC主義者")
    .filter((raceType) => enableBug.value || raceType != "バグ")
    .map((raceType) => ({
      icon: raceTypeIconMap[raceType],
      iconColor: raceTypeColor[raceType],
      badge: mdiPen,
      badgeColor: "secondary",
      onClick: () => setCurrentCharacterRace(raceType),
    })),
  ...relationTypes.map((relationType) => ({
    text: relationType.charAt(0),
    badge: mdiArrowTopRightThin,
    badgeColor: "warning",
    onClick: () => setRelationMode(relationType),
  })),
]);
const ringMenuTarget = ref<HTMLElement | null>(null);
const currentCharacter = ref<Character | null>(null);
const numGnosia = ref(0);
const enableDoctor = ref(true);
const enableEngineer = ref(true);
const enableGurdian = ref(true);
const enableCareTaker = ref(true);
const enableAcBeliever = ref(true);
const enableBug = ref(true);
const enableMap = {
  エ: enableEngineer,
  ド: enableDoctor,
  守: enableGurdian,
  留: enableCareTaker,
  AC: enableAcBeliever,
  バ: enableBug,
};
const mousedown = ref(false);
const dragging = ref(false);
const relationMode = ref<RelationType | null>(null);
const relationFrom = ref<CharacterName | null>(null);

function setRelationMode(relationType: RelationType) {
  relationMode.value = relationType;
  const characterName = currentCharacter.value!.name;
  relationFrom.value = characterName;
  characters.find((character) => character.name == characterName)!.active =
    true;
}
function setCurrentCharacterRace(raceType: RaceType) {
  characters.find(
    (character) => character.name == currentCharacter.value!.name
  )!.race = raceType;
}
function setCurrentCharacterStatus(statusType: StatusType) {
  characters.find(
    (character) => character.name == currentCharacter.value!.name
  )!.status = statusType;
}

function clickCharacter(characterName: CharacterName | null) {
  if (characterName) {
    if (relationMode.value != null) {
      relations.push({
        type: relationMode.value!,
        from: relationFrom.value!,
        to: characterName,
      });
      resetMode();
    } else {
      setCurrentCharacter(characterName);
      isRingMenuOpen.value = false;
      ringMenuTarget.value = null;
      nextTick(() => {
        ringMenuTarget.value = getCharacterIcon(characterName);
        isRingMenuOpen.value = true;
      });
    }
  } else {
    resetMode();
  }
}
function resetMode() {
  relationMode.value = null;
  relationFrom.value = null;
  characters.forEach((character) => (character.active = false));
  isRingMenuOpen.value = false;
  ringMenuTarget.value = null;
}

function setCurrentCharacter(characterName: CharacterName | null) {
  if (characterName) {
    currentCharacter.value = characters.find(
      (character) => character.name == characterName
    )!;
    currentCharacter.value.active = true;
    currentCharacter.value.zIndex = 100;
    characters
      .filter((character) => character.name != characterName)
      .forEach((character) => character.zIndex--);
  } else {
    characters.forEach((character) => (character.active = false));
    currentCharacter.value = null;
  }
}

function mouswdownCharacter(name: CharacterName) {
  setCurrentCharacter(name);
  mousedown.value = true;
}

function mouseup() {
  if (dragging.value) {
    setCurrentCharacter(null);
    dragging.value = false;
  }
  mousedown.value = false;
}

function mousemove(e: MouseEvent) {
  if (mousedown.value && currentCharacter.value) {
    currentCharacter.value.position.x += e.movementX;
    currentCharacter.value.position.y += e.movementY;
    dragging.value = true;
  }
}
function reset() {
  const crewWidth = crew.value!.clientWidth;
  const crewHeight = crew.value!.clientHeight;
  characters.forEach((character, index) => {
    character.active = false;
    character.isAlly = false;
    character.isEnemy = false;
    character.race = "不明";
    character.status = "通常";
    character.position.x = (crewWidth / 6) * (index % 6);
    character.position.y =
      crewHeight - IconSize * 3 + IconSize * Math.floor(index / 6);
    character.zIndex = 100;
  });
  relations.splice(0, relations.length);
}

watch(crew, reset);
watch(enableDoctor, reset);
watch(enableEngineer, reset);
watch(enableGurdian, reset);
watch(enableCareTaker, reset);
watch(enableAcBeliever, reset);
watch(enableBug, reset);

const mounted = ref(false);
onMounted(() => (mounted.value = true));
</script>

<style lang="scss">
#app {
  padding: 0;
  margin: 0;
  width: 100%;
  height: 100%;
  max-width: none;
}

#whiteboard {
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
  bottom: 0;
}
.role {
  display: inline-block;
  text-align: left;
  vertical-align: bottom;
}
</style>
