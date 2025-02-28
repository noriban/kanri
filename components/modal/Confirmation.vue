<!-- SPDX-FileCopyrightText: Copyright (c) 2022-2023 trobonox <hello@trobo.tech> -->
<!-- -->
<!-- SPDX-License-Identifier: Apache-2.0 -->

<template>
  <Modal
    :blur-background="false"
    @closeModal="closeModal()"
  >
    <template #content>
      <main
        class="min-w-[28rem] max-w-[30rem]"
      >
        <div class="flex flex-row items-start justify-between">
          <h1 class="pointer-events-auto pr-5 text-2xl font-bold">
            {{ title || "Are you sure?" }}
          </h1>
          <XMarkIcon
            class="text-accent-hover h-6 w-6 cursor-pointer"
            @click="closeModal()"
          />
        </div>
        <p class="ml-0.5 mt-2 text-lg">
          {{ modalDescription }}
        </p>
        <section
          id="buttons"
          class="mt-8 flex w-full flex-row items-center justify-end gap-8"
        >
          <button
            class="text-accent-hover transition-button"
            @click="closeModal()"
          >
            {{ closeButtonText || "No" }}
          </button>
          <button
            class="bg-accent text-buttons transition-button rounded-md px-4 py-2"
            @click="$emit('confirmAction', boardIndex); closeModal()"
          >
            {{ confirmButtonText || "Yes" }}
          </button>
        </section>
      </main>
    </template>
  </Modal>
</template>

<script setup lang="ts">
import { XMarkIcon } from '@heroicons/vue/24/outline';
import emitter from "@/utils/emitter"

const props = defineProps<{
    title?: string;
    description?: string;
    closeButtonText?: string;
    confirmButtonText?: string;
}>();

const emit = defineEmits<{
    (e: "closeModal"): void,
    (e: "confirmAction", boardIndex?: number): void
}>();

const boardIndex = ref(-1);
const modalDescription = ref(props.description || "");

onMounted(() => {
    emitter.on("openBoardDeleteModal", (params: { index: number, description: string }) => {
        boardIndex.value = params.index;
        modalDescription.value = params.description;
    })

    emitter.on("openModalWithCustomDescription", (params: { description: string }) => {
        modalDescription.value = params.description;
    })
});

const closeModal = () => {
    emit("closeModal");
}
</script>
