<!--
Description:
  Contains edit functionality for selected component
  Functionality includes: delete component, set active component, search for component, change name of component,adjust layer of component,
  delete props, state, actions, HTML elements, or children of component.
  -->

<template>
  <div class="edit-component-div">
    <div>
      <button class="menu-link" @click="resetActiveComponent">
        <i class="fa fa-arrow-left" aria-hidden="true"></i>&nbsp; &nbsp; Create
        Component Menu
      </button>
      <q-card class="no-shadow" dark flat>
        <q-expansion-item default-opened label="Update Component">
          <q-input
            @keyup.enter="editCompName(newName)"
            v-on:keyup.delete.stop
            v-model="newName"
            :placeholder="activeComponent"
            color="white"
            dark
            dense
            outlined
            item-aligned
            padding="5px"
            class="input-add"
            style="margin-bottom: 30px"
            no-error-icon
            reactive-rules
            :rules="[
              (val) => {
                return (
                  !Object.keys(componentMap).includes(val) ||
                  val === activeComponent ||
                  'A component with this name already exists'
                );
              },
            ]"
          ></q-input>

          <VueMultiselect
            v-model="childrenSelected"
            placeholder="Add/remove children"
            :multiple="true"
            :close-on-select="false"
            :options="options"
            :show-labels="false"
            @remove="handleAddChild"
            @select="handleAddChild"
            :max-height="300"
            :option-height="20"
            :searchable="false"
          />
          <q-list
            class="accordBorder no-shadow"
            active-color="secondary"
            indicator-color="secondary"
          >
            <q-expansion-item group="accordion" label="HTML Elements">
              <div class="icon-container">
                <Icons
                  v-model="attributeModal"
                  v-if="attributeModal === false"
                  class="icons"
                  @getClickedIcon="addToSelectedElementList"
                  @activeElement="addToComponentElementList"
                  @activeHTML="addNestedHTML"
                  @activeLayer="addNestedNoActive"
                />
              </div>

              <div class="componentHTML">
                <HTMLQueue></HTMLQueue>
              </div>
              <br />
            </q-expansion-item>
            <InputHTMLMenu
              v-model="attributeModal"
              v-if="attributeModal === true && activeLayer.lineage.length === 0"
              class="htmlElement-selected"
            />
            <q-expansion-item group="accordion" label="Component Attributes">
              <AttributesSubMenu />
            </q-expansion-item>

            <q-expansion-item group="accordion" label="Props">
              <PropsSubMenu />
              <p v-if="!(activeComponentObj as Component).props.length">
                No props in component
              </p>
              <div v-else>
                <a
                  v-for="prop in (componentMap[activeComponent] as Component).props"
                  :key="prop"
                  v-on:click="onActivated(prop)"
                >
                  <q-list class="list=item" dense bordered separator>
                    <q-item clickable v-ripple class="list-item">
                      <q-item-section>
                        <div class="component-container">
                          <div class="component-info">
                            {{ prop }}
                          </div>
                          <q-btn
                            flat
                            icon="highlight_off"
                            v-on:click.stop="deleteProp(prop)"
                          />
                        </div>
                      </q-item-section>
                    </q-item>
                  </q-list>
                </a>
              </div>
            </q-expansion-item>

            <q-expansion-item group="accordion" label="State">
              <StateSubMenu />
            </q-expansion-item>
            <q-expansion-item group="accordion" label="Actions">
              <ActionsSubMenu />
            </q-expansion-item>
          </q-list>
          <q-btn
            id="create-component-btn"
            class="sidebar-btn"
            color="secondary"
            label="Done"
            @click="resetActiveComponent"
          />
          <q-btn
            id="exportButton"
            class="sidebar-btn"
            @click="useExportComponent"
            label="Export currently selected"
          />
          <q-btn
            id="deleteButton"
            class="sidebar-btn deleteAction"
            @click="deleteSelectedComp(activeComponentData)"
            label="Delete currently selected"
          />
        </q-expansion-item>
      </q-card>
    </div>
  </div>
</template>

<script setup lang="ts">
import { useExportComponent } from "../../composables/useExportComponent.js";
import VueMultiselect from "vue-multiselect";
import HTMLQueue from "../../right-sidebar/HTMLQueue.vue";
import Icons from "./Icons.vue";
import PropsSubMenu from "./PropsSubMenu.vue";
import StateSubMenu from "./StateSubMenu.vue";
import ActionsSubMenu from "./ActionsSubMenu.vue";
import AttributesSubMenu from "./AttributesSubMenu.vue";
import InputHTMLMenu from "./InputHTMLMenu.vue";
import { Component, HtmlElement } from "../../../../types";

import { ref, computed, watch } from "vue";
import { useStore } from "../../../store/main";

const store = useStore();

const cloneDeep = require("lodash.clonedeep");

const value = ref("");
const newName = ref("");
const childrenSelected = ref([""]);
const attributeModal = ref(false);

const routes = computed(() => store.routes);
const activeRoute = computed(() => store.activeRoute);
const activeComponent = computed(() => store.activeComponent);
const activeHTML = computed(() => store.activeHTML);
const activeLayer = computed(() => store.activeLayer);
const activeComponentObj = computed(() => store.activeComponentObj);
const componentMap = computed(() => store.componentMap);
const exportAsTypescript = computed(() => store.exportAsTypescript);
const attributeModalOpen = computed(() => store.attributeModalOpen);

const activeRouteDisplay = computed(() => routes.value[activeRoute.value]);

const activeComponentData = cloneDeep(activeComponentObj.value);

const options = computed(() => {
  if (activeComponent.value !== "") {
    newName.value = (activeComponentObj.value as Component).componentName;
    childrenSelected.value = [];
    childrenSelected.value = componentMap.value[activeComponent.value].children;
  } else {
    childrenSelected.value = [];
  }

  const compMap = componentMap.value;
  const activeComp = activeComponent.value;
  const val = activeRouteDisplay.value.map(
    (component) => component.componentName
  );

  const relatives = [...val];

  let parentalLineage: string[] = [];

  const findLineage = (children: string[]) => {
    children.forEach((el) => {
      parentalLineage.push(el);
      if (compMap[el].children.length > 0) {
        findLineage(compMap[el].children);
      }
      if (el !== activeComp) parentalLineage.pop();
      else return;
    });
  };

  findLineage(relatives);

  const optionOutput = val
    .filter((el) => !parentalLineage.includes(el))
    .filter((el) => el !== activeComp);

  return optionOutput;
});

const setActiveComponent: typeof store.setActiveComponent = (payload) =>
  store.setActiveComponent(payload);
// const deleteComponent: typeof store.deleteComponent = (payload) =>
//   store.deleteComponent(payload);
const deleteActiveComponent: typeof store.deleteActiveComponent = () =>
  store.deleteActiveComponent();
const editComponentName: typeof store.editComponentName = (payload) =>
  store.editComponentName(payload);
const updateComponentLayer: typeof store.updateComponentLayer = (payload) =>
  store.updateComponentLayer(payload);
const updateActiveComponentChildrenValue: typeof store.updateActiveComponentChildrenValue =
  (payload) => store.updateActiveComponentChildrenValue(payload);
const addToSelectedElementList: typeof store.addToSelectedElementList = (
  payload
) => store.addToSelectedElementList(payload);
const addToComponentElementList: typeof store.addToComponentElementList = (
  payload
) => store.addToComponentElementList(payload);
const addNestedHTML: typeof store.addNestedHTML = (payload) =>
  store.addNestedHTML(payload);
const addNestedNoActive: typeof store.addNestedNoActive = (payload) =>
  store.addNestedNoActive(payload);
const openAttributeModal: typeof store.openAttributeModal = () =>
  store.openAttributeModal();
const deleteProp: typeof store.deletePropsFromComponent = (payload) =>
  store.deletePropsFromComponent(payload);

const useExportComponentBound = () => {
  useExportComponent.bind(this)();
};

const handleAddChild = (value: string) => {
  updateActiveComponentChildrenValue(value);
};

const onActivated = (componentData: string) => {
  setActiveComponent(componentData);
  activeComponentData.isActive = true;
};

const deleteSelectedComp = (componentData: Component) => {
  if (componentData) deleteActiveComponent();
};

const handleLayer = (e: MouseEvent) => {
  e.preventDefault();
  const payload = {
    activeComponent: activeComponent.value,
    routeArray: activeRouteDisplay.value,
    activeComponentData: activeComponentData,
    z: activeComponentData.z,
  };
  if ((e.target as HTMLElement).innerText === "+") payload.z++;
  if ((e.target as HTMLElement).innerText === "-" && payload.z > 0) payload.z--;
  updateComponentLayer(payload);
};

const handleSelect = (componentName: string) => {
  setActiveComponent(componentName);
  value.value = "";
  activeComponentData.isActive = true;
};

const resetActiveComponent = () => {
  if (activeComponent.value !== "") setActiveComponent("");
};

const editCompName = (name: string) => {
  if (
    name &&
    name !== activeComponent.value &&
    activeComponent.value &&
    !componentMap.value[name]
  ) {
    editComponentName(name);
    newName.value = "";
    setActiveComponent(activeComponent.value);
  }
};

watch(attributeModalOpen, () => {
  attributeModal.value = attributeModalOpen.value;
});
</script>

<style lang="scss" scoped>
.edit-component-div {
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: stretch;
  margin: 4px;
}

.menu-link {
  background-color: $subprimary;
  color: $menutext;
  border: 1px solid $subprimary;
  margin-bottom: 20px;
  padding: 0;
}

.menu-link:hover {
  color: $primary;
}

.q-field {
  margin: 30px 0 10px;
}

.q-expansion-item {
  margin-bottom: 10px;
  background-color: $subprimary;
}

.q-expansion-item__content {
  padding: 20px 0;
}

.componentHTML {
  height: 100px;
  margin-top: 20px;
  padding: 10px;
  background-color: rgba($subsecondary, 0.5);
  overflow-y: scroll;
  border: 1px solid rgba(245, 245, 245, 0.3);
  border-radius: 5px;
  display: flex;
  flex-direction: column;
  align-items: center;
  resize: vertical;
}

.toggleRow {
  display: flex;
  /* align-items: center; */
  justify-content: space-between;
}

/* modifies entire container */
.edit-sidebar {
  padding: 0.5rem;
  border-radius: 5px;
  margin: 1rem;
}

/* modifies each list element */
.q-list {
  margin: 30px 0;
  border-radius: 5px;
  box-shadow: 0 1px 2px 0 rgba(0, 0, 0, 0.2), 0 3px 6px 0 rgba(0, 0, 0, 0.13);
}

.component-info {
  margin: auto 0;
}

/* modifies content in list element */
.component-container {
  display: flex;
  justify-content: space-between;
}

p {
  color: $menutext;
}

.title {
  font-size: 20px;
  font-weight: bold;
  color: $menutext;
}

.toggleText {
  color: $menutext;
}

.toggle {
  align-self: flex-end;
}

#deleteButton {
  width: 100%;
  margin-top: 30px;
  margin-bottom: 30px;
}

#exportButton,
#create-component-btn {
  background-color: $secondary;
  color: $menutext;
  width: 100%;
  margin-bottom: 30px;
}

.border-panel {
  display: flex;
  flex-direction: column;
  align-content: stretch;
  justify-content: flex-start;
  padding: 10px;
  border: 3px solid black;
  border-radius: 10px;
  background-color: $subsecondary;
}

.inner-div {
  display: flex;
  flex-direction: column;
  height: 94%;
  margin: 10px;
}

.htmlElement-selected {
  height: 100%;
}
</style>
