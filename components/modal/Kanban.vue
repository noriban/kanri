<!-- SPDX-FileCopyrightText: Copyright (c) 2022-2023 trobonox <hello@trobo.tech> -->
<!-- -->
<!-- SPDX-License-Identifier: Apache-2.0 -->

<template>
  <Modal
    ref="barebonesModal"
    @closeModal="$emit('closeModal', columnID); titleEditing = false; taskAddMode = false"
  >
    <template #content>
      <div class="flex min-h-[40rem] w-[36rem] flex-col">
        <div class="mb-6">
          <div class="flex flex-row items-start justify-between gap-12">
            <h1
              v-if="!titleEditing"
              :v-model="title"
              class="text-no-overflow pointer-events-auto min-w-[64px] pr-5 text-2xl font-bold"
              @click="enableTitleEditing()"
            >
              {{ title }}
            </h1>
            <input
              v-if="titleEditing"
              ref="titleInput"
              v-model="title"
              v-focus
              type="text"
              maxlength="1000"
              class="bg-elevation-2 text-normal border-accent-focus pointer-events-auto text-xl focus:border-2 focus:border-dotted focus:outline-none"
              @blur="updateTitle"
              @keypress.enter="updateTitle"
            >
            <XMarkIcon
              class="text-accent-hover h-6 w-6 shrink-0 cursor-pointer"
              @click="$emit('closeModal')"
            />
          </div>
          <span class="text-md text-dim-3 mb-6">
            Edit all the things about your card!
          </span>
        </div>

        <div class="flex flex-col pr-10">
          <h2
            class="text-lg font-semibold"
          >
            Card Description
          </h2>
          <textarea
            id="cardDescription"
            v-model="description"
            name="cardDescription"
            cols="6"
            rows="60"
            maxlength="25000"
            placeholder="Enter a detailed description of your card here..."
            class="bg-elevation-2 border-accent-focus pointer-events-auto mt-1.5 h-36 w-full resize-none rounded-md p-2 shadow-lg focus:border-2 focus:border-dotted focus:outline-none"
            @focusin="emitter.emit('modalPreventClickOutsideClose')"
            @blur="updateDescription"
            @keypress.enter="updateDescription"
          />
          <h2
            class="mb-1 mt-6 text-lg font-semibold"
          >
            Tasks
          </h2>
          <div
            class="flex w-full flex-col gap-2"
          >
            <div
              v-if="tasks && tasks.length !== 0"
              class="flex max-h-[148px] w-full flex-col gap-1 overflow-auto pl-1"
            >
              <Container
                drag-handle-selector=".task-drag"
                orientation="vertical"
                lock-axis="y"
                drag-class="cursor-grabbing"
                @drop="onTaskDrop"
              >
                <Draggable
                  v-for="(task, index) in tasks"
                  :key="index"
                  :index="index"
                  :class="draggingEnabled ? 'task-drag' : 'nomoredragging'"
                >
                  <div class="mb-1 flex w-full flex-row items-center justify-between gap-4">
                    <div class="flex w-full flex-row items-center justify-start gap-4">
                      <input
                        v-model="task.finished"
                        type="checkbox"
                        class="h-5 w-5 shrink-0"
                        @change="updateCardTasks"
                      >
                      <input
                        v-if="taskEditMode && index === currentlyEditingTaskIndex"
                        v-model="currentlyEditingTaskName"
                        v-focus
                        type="text"
                        class="bg-elevation-2 border-accent w-full rounded-md border-b-2 border-dotted px-2 py-0.5 outline-none"
                        @blur="updateTask(index)"
                        @keypress.enter="updateTask(index)"
                      >
                      <ClickCounter
                        v-else
                        @double-click="enableTaskEditMode(index, task)"
                      >
                        <span
                          class="text-no-overflow-task w-full"
                        >{{ task.name }}</span>
                      </ClickCounter>
                    </div>
                    <div class="ml-1 flex shrink-0 flex-row gap-1">
                      <button
                        v-if="!taskEditMode"
                        class="shrink-0"
                        @click="enableTaskEditMode(index, task)"
                      >
                        <PhPencilSimple class="text-dim-2 text-accent-hover h-4 w-4" />
                      </button>
                      <button
                        v-if="taskEditMode && currentlyEditingTaskIndex === index"
                        class="shrink-0"
                        @click="updateTask(index)"
                      >
                        <PhCheck class="text-dim-2 text-accent-hover h-4 w-4" />
                      </button>
                      <button
                        class="shrink-0"
                        @click="deleteTask(index)"
                      >
                        <XMarkIcon class="text-dim-2 text-accent-hover h-4 w-4" />
                      </button>
                    </div>
                  </div>
                </Draggable>
              </Container>
            </div>
            <input
              v-if="taskAddMode"
              ref="newTaskInput"
              v-model="newTaskName"
              v-focus
              type="text"
              maxlength="1000"
              placeholder="Enter a task name..."
              class="bg-elevation-2 text-normal border-accent-focus pointer-events-auto rounded-md p-1 text-base focus:border-2 focus:border-dotted focus:outline-none"
              @keypress.enter="createTask"
            >
            <div
              v-if="taskAddMode"
              class="ml-0.5 mt-0.5 flex flex-row gap-4"
            >
              <button
                class="bg-accent rounded-md px-4 py-1"
                @click="createTask"
              >
                Add
              </button>
              <button @click="taskAddMode = false; newTaskName = ''">
                Cancel
              </button>
            </div>
            <button
              v-if="!taskAddMode"
              class="bg-elevation-1 bg-elevation-2-hover mr-8 flex h-min w-full cursor-pointer flex-row items-center gap-2 rounded-md py-2 pl-1 pr-2"
              @click="enableTaskAddMode"
            >
              <PlusIcon class="text-accent h-6 w-6" />
              <span>Add Task</span>
            </button>
          </div>
          <h2
            class="mb-2 mt-6 text-lg font-semibold"
          >
            Card Color
          </h2>
          <div class="flex flex-row gap-4">
            <button
              class="h-7 w-7 rounded-full bg-pink-600 py-1 pl-1.5 pr-1 hover:bg-pink-700"
              @click="setCardColor(columnID, cardID, 'bg-pink-600')"
            >
              <CheckIcon
                v-if="selectedColor === 'bg-pink-600'"
                class="h-4 w-4"
              />
            </button>
            <button
              class="h-7 w-7 rounded-full bg-red-600 py-1 pl-1.5 pr-1 hover:bg-red-700"
              @click="setCardColor(columnID, cardID, 'bg-red-600')"
            >
              <CheckIcon
                v-if="selectedColor === 'bg-red-600'"
                class="h-4 w-4"
              />
            </button>
            <button
              class="h-7 w-7 rounded-full bg-orange-600 py-1 pl-1.5 pr-1 hover:bg-orange-700"
              @click="setCardColor(columnID, cardID, 'bg-orange-600')"
            >
              <CheckIcon
                v-if="selectedColor === 'bg-orange-600'"
                class="h-4 w-4"
              />
            </button>
            <button
              class="h-7 w-7 rounded-full bg-yellow-600 py-1 pl-1.5 pr-1 hover:bg-yellow-700"
              @click="setCardColor(columnID, cardID, 'bg-yellow-600')"
            >
              <CheckIcon
                v-if="selectedColor === 'bg-yellow-600'"
                class="h-4 w-4"
              />
            </button>
            <button
              class="h-7 w-7 rounded-full bg-green-600 py-1 pl-1.5 pr-1 hover:bg-green-700"
              @click="setCardColor(columnID, cardID, 'bg-green-600')"
            >
              <CheckIcon
                v-if="selectedColor === 'bg-green-600'"
                class="h-4 w-4"
              />
            </button>
            <button
              class="h-7 w-7 rounded-full bg-teal-600 py-1 pl-1.5 pr-1 hover:bg-teal-700"
              @click="setCardColor(columnID, cardID, 'bg-teal-600')"
            >
              <CheckIcon
                v-if="selectedColor === 'bg-teal-600'"
                class="h-4 w-4"
              />
            </button>
            <button
              class="h-7 w-7 rounded-full bg-blue-600 py-1 pl-1.5 pr-1 hover:bg-blue-700"
              @click="setCardColor(columnID, cardID, 'bg-blue-600')"
            >
              <CheckIcon
                v-if="selectedColor === 'bg-blue-600'"
                class="h-4 w-4"
              />
            </button>
            <button
              class="h-7 w-7 rounded-full bg-purple-600 py-1 pl-1.5 pr-1 hover:bg-purple-700"
              @click="setCardColor( columnID, cardID, 'bg-purple-600')"
            >
              <CheckIcon
                v-if="selectedColor === 'bg-purple-600'"
                class="h-4 w-4"
              />
            </button>
            <button
              class="bg-elevation-2 bg-elevation-3-hover h-7 w-7 rounded-full py-1 pl-1.5 pr-1"
              @click="setCardColor(columnID, cardID, 'bg-elevation-2')"
            >
              <CheckIcon
                v-if="selectedColor === 'bg-elevation-2'"
                class="h-4 w-4"
              />
            </button>
          </div>
        </div>
      </div>
    </template>
  </Modal>
</template>

<script setup lang="ts">
import emitter from "@/utils/emitter"
import { Ref } from "vue";
//@ts-ignore
import { Container, Draggable } from 'vue3-smooth-dnd';
import { PlusIcon, XMarkIcon, CheckIcon } from "@heroicons/vue/24/solid";
import { PhCheck, PhPencilSimple } from "@phosphor-icons/vue";

const emit = defineEmits<{
    (e: "closeModal", columnID: string): void,
    (e: "setCardDescription", columnID: string, cardIndex: number, description: string): void,
    (e: "setCardTitle", columnID: string, cardIndex: number, title: string): void,
    (e: "setCardColor", columnID: string, cardIndex: number, color: string): void,
    (e: "setCardTasks", columnID: string, cardIndex: number, tasks: Array<{name: string, finished: boolean}>): void
}>();

const columnID = ref("");
const cardID = ref(0);
const title = ref("");
const description = ref("");
const tasks: Ref<Array<{ name: string, finished: boolean }>> = ref([]);
const selectedColor = ref("");

const titleEditing = ref(false);
const titleInput: Ref<HTMLInputElement | null> = ref(null);

const newTaskName = ref("");
const taskAddMode = ref(false);
const newTaskInput: Ref<HTMLInputElement | null> = ref(null);

const currentlyEditingTaskIndex = ref(-1);
const currentlyEditingTaskName = ref("");
const taskEditMode = ref(false);

const draggingEnabled = ref(true);

const enableTitleEditing = () => {
    emitter.emit("modalPreventClickOutsideClose");
    titleEditing.value = true;
}

const initModal = (
    columnIDParam: string, cardIdParam: number,
    titleParam: string,
    descriptionParam?: string,
    tasksParam?: Array<{ name: string, finished: boolean }>,
    selectedColorParam?: string
) => {
    newTaskName.value = "";

    columnID.value = columnIDParam;
    cardID.value = cardIdParam;
    title.value = titleParam;
    description.value = descriptionParam || "";
    tasks.value = tasksParam || [];
    selectedColor.value = selectedColorParam || "bg-elevation-2";
}

const enableTaskAddMode = () => {
    taskAddMode.value = true;
}

const createTask = () => {
    if (newTaskName.value == null || !(/\S/.test(newTaskName.value))) return;

    tasks.value.push({ name: newTaskName.value, finished: false });
    newTaskName.value = "";
    taskAddMode.value = false;

    updateCardTasks();
}

const deleteTask = (index: number) => {
    tasks.value.splice(index, 1);
    updateCardTasks();
}

const onTaskDrop = (dropResult: any) => {
    tasks.value = applyDrag(tasks.value, dropResult);
    updateCardTasks();
}

const enableTaskEditMode = (index: number, task: {name: string, finished: boolean}) => {
    taskEditMode.value = true;
    draggingEnabled.value = false;
    currentlyEditingTaskIndex.value = index;
    currentlyEditingTaskName.value = task.name;
}

const updateTask = (index: number) => {
    const taskNameEmpty = currentlyEditingTaskName.value == null || !(/\S/.test(currentlyEditingTaskName.value));
    if (taskNameEmpty || currentlyEditingTaskIndex.value === -1) return;

    tasks.value[index].name = currentlyEditingTaskName.value;

    currentlyEditingTaskIndex.value = -1;
    currentlyEditingTaskName.value = "";
    taskEditMode.value = false;
    draggingEnabled.value = true;

    updateCardTasks();
}

const updateCardTasks = () => {
    emit("setCardTasks", columnID.value, cardID.value, tasks.value);
}

const updateDescription = () => {
    emit("setCardDescription", columnID.value, cardID.value, description.value);
    emitter.emit("modalEnableClickOutsideClose");
}

const updateTitle = () => {
    emitter.emit("modalEnableClickOutsideClose");

    if (title.value == null || !(/\S/.test(title.value))) return;

    titleEditing.value = false;
    emit("setCardTitle", columnID.value, cardID.value, title.value);
}

const setCardColor = (columnID: string, cardID: number, color: string) => {
    selectedColor.value = color;
    emit('setCardColor', columnID, cardID, color);
}

defineExpose({ initModal });
</script>
