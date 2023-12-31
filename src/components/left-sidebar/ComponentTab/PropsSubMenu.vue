<template>
  <div>
    <q-input
      @keyup.enter="createNewProp(textProps)"
      color="white"
      dark
      outlined
      bottom-slots
      v-model="textProps"
      label="Create new prop"
      dense
      class="input-add"
      v-on:keyup.delete.stop
    >
      <template v-slot:append>
        <q-btn flat icon="add" @click="createNewProp(textProps)" />
      </template>
    </q-input>

    <div id="props-select">
      <VueMultiselect
        v-model="selectProps"
        class="multiselect"
        placeholder="Select from existing props"
        :multiple="true"
        :close-on-select="false"
        :max-height="180"
        :option-height="20"
        open-direction="top"
        :options="propsOptions"
        :searchable="false"
        @search-change="stopDelete($event)"
      >
        <template v-slot:noResult>No props found.</template>
      </VueMultiselect>
    </div>
    <br />
    <div>
      <q-btn
        v-if="selectProps.length"
        id="add-props-btn"
        class="add-btn"
        color="secondary"
        label="Add Prop(s)"
        @click="addPropsToComp"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import VueMultiselect from "vue-multiselect";
import { useStore } from "../../../store/main";
import { ref, computed } from "vue";

const store = useStore();
const textProps = ref("");

const selectedProps = computed(() => store.selectedProps);
const userProps = computed(() => store.userProps);

const propsOptions = userProps.value;
const selectProps = computed({
  get() {
    return [...selectedProps.value];
  },
  set(value) {
    addPropsSelected(value);
  },
});

const createProp: typeof store.createProp = (payload) =>
  store.createProp(payload);
const addPropsSelected: typeof store.addPropsSelected = (payload) =>
  store.addPropsSelected(payload);
const addPropsToComponent: typeof store.addPropsToComponent = (payload) =>
  store.addPropsToComponent(payload);

const stopDelete = (e: KeyboardEvent) => {
  if (e.code === "Backspace") e.stopPropagation();
};

const createNewProp = (text: string) => {
  if (![...userProps.value].includes(text) && text) {
    createProp(text);
    textProps.value = "";
  }
};

const addPropsToComp = () => addPropsToComponent([...selectedProps.value]);
</script>

<!-- <script>
import { mapState, mapActions } from "vuex";
import VueMultiselect from "vue-multiselect";

export default {
  name: "PropsSubMenu",
  components: {
    VueMultiselect,
  },
  data() {
    return {
      textProps: "",
    };
  },
  computed: {
    ...mapState(["selectedProps", "userProps"]),
    propsOptions() {
      return this.userProps;
    },
    selectProps: {
      get() {
        return this.selectedProps;
      },
      set(value) {
        this.addPropsSelected(value);
      },
    },
  },

  methods: {
    ...mapActions(["createProp", "addPropsSelected", "addPropsToComponent"]),
    // Prevent Delete on changes to searchable multiselect
    stopDelete(e) {
      if (e.code === "Backspace") e.stopPropogation();
    },
    // Create's a new prop that will be stored in the userProps array within store, and it will be added to the props drop-down menu
    createNewProp(text) {
      if (!this.userProps.includes(text) && text) {
        this.createProp(text);
        this.textProps = "";
      }
    },
    addPropsToComp() {
      this.addPropsToComponent(this.selectedProps);
    },
  },
};
</script> -->

<style lang="scss" scoped>
.q-field {
  margin: 30px 0 10px;
}
</style>
