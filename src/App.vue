<template>
  <v-layout full-height>
    <v-app-bar>
      <v-app-bar-nav-icon variant="text" @click.stop="drawer = !drawer" />
      <v-toolbar-title>Gnosia tool</v-toolbar-title>
      <v-spacer />
      <v-select
        label="グノーシア人数"
        v-model.number="numGnosia"
        :items="[...Array(7).keys()]"
        style="max-width: 200px"
      />
      <v-btn
        icon
        variant="flat"
        @click="enableDoctor = !enableDoctor"
        :color="enableDoctor ? 'primary' : undefined"
        >ド</v-btn
      >
      <v-btn
        icon
        variant="flat"
        @click="enableEngineer = !enableEngineer"
        :color="enableEngineer ? 'primary' : undefined"
        >エ</v-btn
      >
      <v-btn
        icon
        variant="flat"
        @click="enableGurdian = !enableGurdian"
        :color="enableGurdian ? 'primary' : undefined"
        >守</v-btn
      >
      <v-btn
        icon
        variant="flat"
        @click="enableCareTaker = !enableCareTaker"
        :color="enableCareTaker ? 'primary' : undefined"
        >留</v-btn
      >
      <v-btn
        icon
        variant="flat"
        @click="enableAcBeliever = !enableAcBeliever"
        :color="enableAcBeliever ? 'primary' : undefined"
        >AC</v-btn
      >
      <v-btn
        icon
        variant="flat"
        @click="enableBug = !enableBug"
        :color="enableBug ? 'primary' : undefined"
        >バ</v-btn
      >
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

      <div id="whiteboard" @mousemove="mousemove" @mouseup="drop">
        <div style="display: inline-block; width: 50%; height: 100%">
          <div
            class="role"
            :style="{
              width: '100%',
              height: '50%',
              backgroundColor: 'darkgray',
            }"
            ref="crew"
          >
            乗員
          </div>
          <div
            class="role"
            :style="{
              width: '100%',
              height: '25%',
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
              height: '25%',
              backgroundColor: 'darkgreen',
              visibility: enableDoctor ? 'visible' : 'hidden',
            }"
          >
            ドクター
          </div>
        </div>
        <div style="display: inline-block; width: 50%; height: 100%">
          <div style="width: 100%; height: 40%">
            <div
              class="role"
              :style="{
                display: 'inline-block',
                width: '50%',
                height: '100%',
                backgroundColor: 'darkgreen',
                visibility: enableGurdian ? 'visible' : 'hidden',
              }"
            >
              守護天使
            </div>
            <div
              class="role"
              :style="{
                display: 'inline-block',
                width: '50%',
                height: '100%',
                backgroundColor: 'darkgoldenrod',
                visibility: enableCareTaker ? 'visible' : 'hidden',
              }"
            >
              留守番
            </div>
          </div>
          <div style="width: 100%; height: 40%">
            <div
              class="role"
              :style="{
                display: 'inline-block',
                width: '30%',
                height: '100%',
                backgroundColor: 'darkviolet',
                visibility: enableAcBeliever ? 'visible' : 'hidden',
              }"
            >
              AC主義者
            </div>
            <div
              class="role"
              :style="{
                display: 'inline-block',
                width: '30%',
                height: '100%',
                backgroundColor: 'darkmagenta',
                visibility: enableBug ? 'visible' : 'hidden',
              }"
            >
              バグ
            </div>
            <div
              ref="gnosia"
              class="role"
              style="
                display: inline-block;
                width: 40%;
                height: 100%;
                background-color: darkred;
              "
            >
              グノーシア
            </div>
          </div>
          <div
            class="role"
            style="width: 100%; height: 20%; background-color: black"
          >
            未使用
          </div>
        </div>
        <CharacterIcon
          v-for="(character, index) in characters"
          :ref="(el) => setCharacterRef(el as CharacterIconType, index)"
          :key="index"
          :character="character"
          @mousedown="drag(character.name)"
          @click="clickCharacter(character.name)"
        />
        <Arrow
          v-for="(relation, index) in relations"
          :key="index"
          :from="getCharacterIcon(relation.from)"
          :to="getCharacterIcon(relation.to)"
          :stroke-color="getStrokeColor(relation.type)"
        />
      </div>
      <v-container fluid style="position: fixed; bottom: 0">
        <v-row>
          <v-spacer />
          <v-col cols="auto">
            <v-btn icon @click="setRelationMode('庇う')">庇</v-btn>
          </v-col>
          <v-col cols="auto">
            <v-btn icon @click="setRelationMode('疑う')">疑</v-btn>
          </v-col>
          <v-col cols="auto">
            <v-btn icon @click="setRelationMode('人間だ')">人</v-btn>
          </v-col>
          <v-col cols="auto">
            <v-btn icon @click="setRelationMode('グノーシアだ')">グ</v-btn>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-layout>
</template>

<script setup lang="ts">
import { reactive, ref, watch } from "vue";
import { useTheme } from "vuetify";
import { mdiWeatherSunny, mdiWeatherNight, mdiReload } from "@mdi/js";
import Arrow from "./components/Arrow.vue";

type RelationType = "疑う" | "庇う" | "人間だ" | "グノーシアだ";
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
type CharacterIconType = InstanceType<typeof CharacterIcon>;
const crew = ref<HTMLElement | null>(null);
const gnosia = ref<HTMLElement | null>(null);
const characters = reactive<Character[]>(
  characterNames.map((name) => ({
    name,
    callDocter: false,
    callEngineer: false,
    isDoctor: false,
    isEngineer: false,
    isGurdian: false,
    isCareTaker: false,
    isGnosia: false,
    isAcBeliever: false,
    isBug: false,
    position: { x: 0, y: 0 },
    active: false,
  }))
);
const relations = reactive<Relation[]>([]);
const mounted = ref(false);
onMounted(() => (mounted.value = true));
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
      return "darkred";
  }
}
const currentCharacter = ref<Character | null>(null);
const numGnosia = ref(0);
const enableDoctor = ref(true);
const enableEngineer = ref(true);
const enableGurdian = ref(true);
const enableCareTaker = ref(true);
const enableAcBeliever = ref(true);
const enableBug = ref(true);
const relationMode = ref<RelationType | null>(null);
const relationFrom = ref<CharacterName | null>(null);

function setRelationMode(relationType: RelationType) {
  relationMode.value = relationType;
}
function clickCharacter(characterName: CharacterName) {
  if (relationMode.value != null) {
    if (relationFrom.value == null) {
      setRelationFrom(characterName);
    } else {
      setRelationTo(characterName);
    }
  }
}
function setRelationFrom(characterName: CharacterName) {
  relationFrom.value = characterName;
}
function setRelationTo(characterName: CharacterName) {
  relations.push({
    type: relationMode.value!,
    from: relationFrom.value!,
    to: characterName,
  });
  relationMode.value = null;
  relationFrom.value = null;
}

function drag(name: CharacterName) {
  currentCharacter.value = characters.find(
    (character) => character.name == name
  )!;
  currentCharacter.value.active = true;
}
function drop() {
  characters.forEach((character) => (character.active = false));
  currentCharacter.value = null;
}

function mousemove(e: MouseEvent) {
  if (currentCharacter.value) {
    currentCharacter.value.position.x += e.movementX;
    currentCharacter.value.position.y += e.movementY;
  }
}
function reset() {
  const crewWidth = crew.value!.clientWidth;
  const crewHeight = crew.value!.clientHeight;
  characters.forEach((character, index) => {
    character.active = false;
    character.callDocter = false;
    character.callEngineer = false;
    character.isDoctor = false;
    character.isEngineer = false;
    character.isGurdian = false;
    character.isCareTaker = false;
    character.isGnosia = false;
    character.isAcBeliever = false;
    character.isBug = false;
    character.position.x = (crewWidth / 6) * (index % 6);
    character.position.y =
      crewHeight - IconSize * 3 + IconSize * Math.floor(index / 6);
  });
}

watch(crew, reset);
watch(enableDoctor, reset);
watch(enableEngineer, reset);
watch(enableGurdian, reset);
watch(enableCareTaker, reset);
watch(enableAcBeliever, reset);
watch(enableBug, reset);
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