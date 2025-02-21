<!-- SPDX-FileCopyrightText: Copyright (c) 2022-2023 trobonox -->
<!-- -->
<!-- SPDX-License-Identifier: Apache-2.0 -->

<template>
  <div
    class="bg-elevation-1 max-h-column flex flex-col rounded-md p-2 shadow-lg"
    :class="columnSizeClass"
  >
    <div
      id="board-title"
      class="flex flex-row items-start justify-between gap-4"
      :class="titleTextClassZoom"
    >
      <h1
        v-if="!titleEditing"
        class="text-no-overflow ml-1 font-bold"
        @click="enableTitleEditing()"
      >
        {{ boardTitle }}
      </h1>

      <input
        v-if="titleEditing"
        ref="titleInput"
        v-model="titleNew"
        v-focus
        type="text"
        maxlength="1000"
        class="bg-elevation-2 border-accent text-no-overflow mr-2 w-full rounded-sm border-2 border-dotted px-2 outline-none"
        @blur="updateColumnTitle"
        @keypress.enter="
          updateColumnTitle();
          emitter.emit('columnActionDone');
        "
      >

      <ClickCounter
        @single-click="$emit('removeColumn', id)"
        @double-click="$emit('removeColumnNoConfirmation', id)"
      >
        <XMarkIcon
          class="text-dim-4 text-accent-hover mt-2 h-4 w-4 shrink-0 grow-0 cursor-pointer"
        />
      </ClickCounter>
    </div>

    <Container
      group-name="cards"
      :get-child-payload="getChildPayload"
      class="max-h-65vh custom-scrollbar mt-2 overflow-y-auto rounded-sm"
      drag-handle-selector=".kanbancard-drag"
      orientation="vertical"
      drag-class="cursor-grabbing"
      @drop="onDrop"
    >
      <Draggable
        v-for="(card, index) in cards"
        :key="index"
        :class="draggingEnabled ? 'kanbancard-drag' : 'nomoredragging'"
      >
        <KanbanCard
          class="mb-3 min-h-[30px] cursor-grab rounded-sm p-3"
          :index="index"
          :card="card"
          :zoom-level="zoomLevel"
          @remove-card="removeCard"
          @remove-card-with-confirmation="removeCardWithConfirmation"
          @enable-dragging="enableDragging"
          @disable-dragging="disableDragging"
          @open-kanban-modal="openKanbanModal"
          @set-card-title="setCardTitle"
        />
      </Draggable>
    </Container>

    <div
      v-if="cardAddMode"
      class="mt-2 flex flex-col"
    >
      <textarea
        id="newCardInput"
        ref="newCardInput"
        v-model="newCardName"
        v-focus
        v-resizable
        type="text"
        maxlength="5000"
        placeholder="Enter a card title..."
        class="bg-elevation-2 border-accent-focus mb-2 h-12 overflow-hidden rounded-sm p-1 focus:border-2 focus:border-dotted focus:outline-none"
        @keypress.enter="
          addCard();
          emitter.emit('columnActionDone');
        "
      />
      <div class="flex w-full flex-row justify-start gap-2">
        <button
          id="submitButton"
          class="text-buttons transition-button bg-accent rounded-md px-2 py-1"
          @click="
            addCard();
            emitter.emit('columnActionDone');
          "
        >
          Add Card
        </button>
        <button
          class="bg-elevation-3-hover transition-button rounded-md px-2 py-1"
          @click="
            cardAddMode = !cardAddMode;
            newCardName = '';
            draggingEnabled = true;
            emit('enableDragging');
            emitter.emit('columnActionDone');
          "
        >
          Cancel
        </button>
      </div>
    </div>

    <div
      v-if="!cardAddMode"
      class="text-dim-1 bg-elevation-3-hover mt-2 flex cursor-pointer flex-row gap-1 rounded-md py-1 font-medium"
      @click="enableCardAddMode()"
    >
      <PlusIcon class="h-6 w-6 p-0.5" />
      <h2>Add Card</h2>
    </div>
  </div>
</template>

<script setup lang="ts">
import emitter from "@/utils/emitter";
import { applyDrag } from "@/utils/drag-n-drop";
import { Card, Column } from "@/types/kanban-types";

import type { Ref } from "vue"
//@ts-ignore, sadly this library does not have ts typings
import { Container, Draggable } from "vue3-smooth-dnd";

import { XMarkIcon, PlusIcon } from "@heroicons/vue/24/solid";

const props = defineProps<{
    colIndex: number,
    id: string;
    title: string;
    cardsList: Array<Card>;
    zoomLevel: number;
}>();

const emit = defineEmits<{
  (e: "updateStorage", column: Column): void,
  (e: "removeColumn", columnId: string): void,
  (e: "removeColumnNoConfirmation", columnId: string): void,
  (e: "removeCardWithConfirmation", columnId: string, cardIndex: number, cardRef: Ref<HTMLDivElement | null>): void,
  (e: "enableDragging"): void,
  (e: "disableDragging"): void,
  (e: "openKanbanModal", columnId: string, cardIndex: number, el: Card ): void,
  (e: "setColumnEditIndex", columnId: number, eventType: string): void,
}>();

const titleInput: Ref<HTMLInputElement | null> = ref(null);
const newCardInput: Ref<HTMLInputElement | null> = ref(null);

const cards = ref<Card[]>(props.cardsList);
const newCardName = ref("");
const cardAddMode = ref(false);

const titleNew = ref(props.title);
const titleEditing = ref(false);

const draggingEnabled = ref(true);

const boardTitle = ref(props.title);

onMounted(() => {
    document.addEventListener("keydown", keyDownListener);

    emitter.on("enableColumnTitleEditing", (columnID) => {
        if (columnID === props.id) {
            enableTitleEditing();
        }
    });

    emitter.on("enableColumnCardAddMode", (columnID) => {
        if (columnID === props.id) {
            enableCardAddMode();
        }
    });

    emitter.on("resetColumnInputs", () => {
        cardAddMode.value = false;
        newCardName.value = "";
        titleEditing.value = false;
    });
});

const titleTextClassZoom = computed(() => {
    switch (props.zoomLevel) {
    case 0:
        return ["text-lg"]

    case -1:
        return ["text-md"]

    case 1:
        return ["text-xl"]

    case 2:
        return ["text-2xl"]

    default:
        return [""]
    }
})

const columnSizeClass = computed(() => {
    switch (props.zoomLevel) {
    case 0:
        return ["w-64"]

    case -1:
        return ["w-48"]

    case 1:
        return ["w-96"]

    case 2:
        return ["w-[560px]"]

    default:
        return [""]
    }
})

onBeforeUnmount(() => {
    document.removeEventListener("keydown", keyDownListener);
});

const keyDownListener = (e: { key: string; }) => {
    if (e.key === "Escape") {
        cardAddMode.value = false;
        newCardName.value = "";
        titleEditing.value = false;

        emitter.emit("columnActionDone");
    }
};

const onDrop = (dropResult: any) => {
    cards.value = applyDrag(cards.value, dropResult);
    updateStorage();
};

const getChildPayload = (index: number) => {
    return cards.value[index];
};

const enableDragging = () => {
    draggingEnabled.value = true;
    emit("enableDragging");
}

const disableDragging = () => {
    draggingEnabled.value = false;
    emit("disableDragging");
}

const enableTitleEditing = () => {
    emit("setColumnEditIndex", props.colIndex, "title-edit");
    disableDragging();

    titleEditing.value = true;
    titleNew.value = boardTitle.value;
}

const enableCardAddMode = () => {
    emit("setColumnEditIndex", props.colIndex, "card-add");
    emitter.emit("resetColumnInputs");
    disableDragging();

    cardAddMode.value = true;
}

const updateColumnTitle = () => {
    enableDragging();

    if (titleNew.value == null || !(/\S/.test(titleNew.value))) {
        titleNew.value = "";
        titleEditing.value = false;
        return;
    }

    boardTitle.value = titleNew.value;
    titleNew.value = "";

    titleEditing.value = false;
    updateStorage();
}

const addCard = () => {
    enableDragging();

    if (newCardName.value == null || !(/\S/.test(newCardName.value))) return;

    cards.value[cards.value.length] = { name: newCardName.value };

    newCardName.value = "";
    cardAddMode.value = false;
    updateStorage();
};

const removeCardWithConfirmation = (cardIndex: number, cardRef: Ref<HTMLDivElement | null>) => {
    emit("removeCardWithConfirmation", props.id, cardIndex, cardRef);
}

const removeCard = (index: number) => {
    cards.value.splice(index, 1);
    updateStorage();
};

const setCardTitle = (cardIndex: number, name: string) => {
    cards.value[cardIndex].name = name;
    updateStorage();
};

const setCardDescription = (cardIndex: number, description: string) => {
    cards.value[cardIndex].description = description;
    updateStorage();
};

const setCardColor = (cardIndex: number, color: string) => {
    cards.value[cardIndex].color = color;
    updateStorage();
};

const setCardTasks = (cardIndex: number, tasks: Array<{name: string, finished: boolean}>) => {
    cards.value[cardIndex].tasks = tasks;
    updateStorage();
};

const openKanbanModal = (_: any, index: number, el: Card) => {
    disableDragging();

    emit("openKanbanModal", props.id, index, el);
};

const closeModal = () => {
    enableDragging();
};

const updateStorage = () => {
    const column = {
        id: props.id,
        title: boardTitle.value,
        cards: cards.value,
    };

    emit("updateStorage", column);
};

defineExpose({ setCardTitle, setCardDescription, setCardColor, setCardTasks, removeCard, closeModal, enableDragging });
</script>

<style scoped>
.max-h-column {
   max-height: 75vh;
}
</style>
