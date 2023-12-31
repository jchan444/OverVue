<!-- this will display the active/selected component's state. Functionality will include being able to add more state and delete state -->
<!-- Functions:
- display all the component's state as a list
  - maybe with a delete button similar to props right now
- dropdown to add more state to component (multiselect)
- submit button (like props)
-->

<template>
  <div class="selection-container">
    <div id="action-select">
      <VueMultiselect
        v-model="selectAction"
        class="multiselect"
        placeholder="Select Action for Component"
        :multiple="true"
        :close-on-select="false"
        :max-height="180"
        :option-height="20"
        open-direction="top"
        :options="userActions"
        :searchable="false"
        @search-change="stopDelete($event)"
      >
      </VueMultiselect>
      <br />
      <q-btn
        v-if="selectAction.length"
        id="add-actions-btn"
        class="add-btn"
        color="secondary"
        label="Map Action(s)"
        @click="addActionToComp"
      />
    </div>
    <p v-if="!(componentMap[activeComponent] as Component).actions.length">
      No actions in component
    </p>
    <a
      v-else
      v-for="action in (componentMap[activeComponent] as Component).actions"
      :key="action"
    >
      <q-list class="list-item" dense bordered separator>
        <q-item clickable v-ripple class="list-item">
          <q-item-section>
            <div class="component-container">
              <div class="component-info">
                {{ action }}
              </div>
              <q-btn
                round
                flat
                icon="highlight_off"
                v-on:click.stop="deleteAction(action)"
              />
            </div>
          </q-item-section>
        </q-item>
      </q-list>
    </a>
    <br />
  </div>
</template>

<script setup lang="ts">
// new script for Composition API

import { computed } from "vue";
import { useStore } from "../../../store/main.js";
import { Component } from "../../../../types";
import VueMultiselect from "vue-multiselect";

const store = useStore();

const selectedActions = computed(() => store.selectedActions);
const userActions = computed(() => store.userActions);
const componentMap = computed(() => store.componentMap);
const activeComponent = computed(() => store.activeComponent);

//getters

const actionOptions = userActions;
//  actionOptions() {
//       return this.userActions;
//     },

const selectAction = computed({
  get() {
    return [...selectedActions.value];
  },
  set(value) {
    addActionSelected(value);
  },
});

// Methods

const addActionSelected: typeof store.addActionSelected = (payload) =>
  store.addActionSelected(payload);
const addActionToComponent: typeof store.addActionToComponent = (payload) =>
  store.addActionToComponent(payload);
const deleteActionFromComponent: typeof store.deleteActionFromComponent = (
  payload
) => store.deleteActionFromComponent(payload);

const stopDelete = (e: KeyboardEvent) => {
  if (e.code === "Backspace") e.stopPropagation();
};
const addActionToComp = () => {
  addActionToComponent([...selectedActions.value]);
};
const deleteAction = (action: string) => {
  deleteActionFromComponent(action);
};
</script>

<!-- <script>
import { mapState, mapActions } from "vuex";
import VueMultiselect from "vue-multiselect";

export default {
  name: "ActionsSubMenu",
  components: {
    VueMultiselect,
  },
  computed: {
    ...mapState(["selectedActions", "userActions", "componentMap", "activeComponent",]),
    actionOptions() {
      return this.userActions;
    },
    selectAction: {
      get() {
        console.log("Inside get!", this.selectedActions);
        return this.selectedActions;
      },
      set(value) {
        console.log("What is value?", value)
        this.addActionSelected(value);
      },
    },
  },
  methods: {
    ...mapActions([
      "addActionSelected",
      "addActionToComponent",
      "deleteActionFromComponent",
    ]),
    // Prevent Delete on changes to searchable multiselect
    stopDelete(e) {
      if (e.code === "Backspace") e.stopPropogation();
    },
    // adds an action to the currently selected component
    addActionToComp() {
      this.addActionToComponent(this.selectedActions);
    },
    // delete selected action from active component
    deleteAction(action) {
      this.deleteActionFromComponent(action);
    },
  },
};
</script> -->

<style lang="scss" scoped>
.selection-container {
  padding: 30px 0;
}

.component-container {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
}
</style>
