<script setup lang="ts">
import FileUpload from '~/components/FileUpload.vue'
import ExportToTailwind from '~/components/ExportToTailwind.vue'
import ExportAsImage from '~/components/ExportAsImage.vue'
import Swap from '~/icons/Swap.vue'
import Dropdown, { type DropdownChangeEvent } from 'primevue/dropdown'
import { useToast } from 'primevue/usetoast'
import { useImage } from '~/composables/useImage'
import { useColors } from '~/composables/useColors'
import { type ColorWithRgbAndHex } from '~/types/colors'

const toast = useToast()
const { imageSrc } = useImage()
const {
  imageColors,
  swapColors,
  primaryColor,
  accentColor,
  selectPrimaryColor,
  selectAccentColor,
  contrastRatio
} = useColors()

const dropdownOptions = computed(() =>
  imageColors.value.map(color => ({ label: color.hex, value: color }))
)

const handlePrimaryColorChange = (event: DropdownChangeEvent) => {
  const selectedValue = event.value as ColorWithRgbAndHex
  selectPrimaryColor(selectedValue)
}

const handleAccentColorChange = (event: DropdownChangeEvent) => {
  const selectedValue = event.value as ColorWithRgbAndHex
  selectAccentColor(selectedValue)
}

const copyToClipboard = async (text: string) => {
  try {
    await navigator.clipboard.writeText(text)
    toast.add({
      severity: 'success',
      summary: 'Copied to Clipboard',
      group: 'bl',
      life: 2000
    })
  } catch (err) {
    console.error('Error al copiar texto:', err)
  }
}
</script>

<template>
  <aside
    class="md:col-span-3 pt-22 pb-12 md:pt-18 w-full sticky px-3 border-r border-black/10 bg-gray-200 dark:border-white/10 dark:bg-neutral-950 overflow-y-auto h-100dvh"
  >
    <Toast position="bottom-left" group="bl" />
    <header class="flex flex-col gap-y-2">
      <picture
        v-if="imageSrc"
        class="flex w-full h-62 rounded-md overflow-hidden"
      >
        <img
          :src="imageSrc"
          alt="Uploaded Image Preview"
          class="w-full aspect-video object-cover object-center-center rounded-md transition-transform ease-in-out duration-300 scale-100 hover:scale-105"
        />
      </picture>
      <FileUpload />
    </header>
    <div class="pt-10 flex flex-col gap-y-4">
      <section
        class="font-medium text-xl text-gray-400 flex flex-col items-start justify-center gap-y-3"
      >
        <div class="w-full flex items-center justify-between">
          <Dropdown
            v-if="primaryColor"
            v-model="primaryColor"
            :options="dropdownOptions"
            checkmark
            @change="handlePrimaryColorChange"
            :highlightOnSelect="false"
            optionValue="value"
            class="size-20"
            :style="{ backgroundColor: primaryColor.hex }"
          >
            <template #value="slotProps">
              <div v-if="slotProps.value"></div>
            </template>
            <template #option="slotProps">
              <div
                :style="{ backgroundColor: slotProps.option.value.hex }"
                class="w-full h-6"
              ></div>
            </template>
          </Dropdown>
          <button
            @click="swapColors"
            class="rounded-full p-2 aspect-square bg-gray-300 dark:bg-neutral-900"
          >
            <Swap />
          </button>
          <Dropdown
            v-if="accentColor"
            v-model="accentColor"
            :options="dropdownOptions"
            checkmark
            @change="handleAccentColorChange"
            :highlightOnSelect="false"
            optionValue="value"
            class="size-20"
            :style="{ backgroundColor: accentColor.hex }"
          >
            <template #value="slotProps">
              <div v-if="slotProps.value"></div>
            </template>
            <template #option="slotProps">
              <div
                :style="{ backgroundColor: slotProps.option.value.hex }"
                class="w-full h-6"
              ></div>
            </template>
          </Dropdown>
        </div>
        <p v-if="imageColors">
          Contrast Ratio:
          <span
            :class="[
              Number(contrastRatio) > 4.5 ? 'text-green-500' : 'text-red-500'
            ]"
            >{{ contrastRatio }}</span
          >
        </p>
      </section>
      <section class="grid grid-cols-5 justify-content-center gap-y-1 gap-x-2">
        <div
          v-for="(color, index) in imageColors"
          :key="index"
          class="relative w-full aspect-square h-auto rounded-lg"
          :style="{ backgroundColor: color.hex }"
          aria-label="{{ checkIfColorSelected(color) ? 'Deselect' : 'Select' }} color with hex code {{ color.hex }} }}"
        ></div>
        <button
          v-for="(color, index) in imageColors"
          :key="index"
          class="bg-gray-300 dark:bg-neutral-900 rounded-md 2xl:px-3 py-1 opacity-60 hover:opacity-100 text-xs 2xl:text-base"
          @click="copyToClipboard(color.hex)"
        >
          {{ color.hex }}
        </button>
      </section>
      <div class="flex items-center justify-between gap-x-3">
        <ExportToTailwind />
        <ExportAsImage />
      </div>
    </div>
  </aside>
</template>
