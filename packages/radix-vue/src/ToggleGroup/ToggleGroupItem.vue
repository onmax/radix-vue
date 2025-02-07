<script lang="ts">
interface ToggleGroupItemProps extends PrimitiveProps {
  value?: string;
  disabled?: boolean;
}
</script>

<script setup lang="ts">
import { inject, computed, onMounted } from "vue";
import {
  PrimitiveButton,
  usePrimitiveElement,
  type PrimitiveProps,
} from "@/Primitive";
import {
  TOGGLE_GROUP_INJECTION_KEY,
  type ToggleGroupProvideValue,
} from "./ToggleGroupRoot.vue";
import { useArrowNavigation } from "../shared";

const injectedValue = inject<ToggleGroupProvideValue>(
  TOGGLE_GROUP_INJECTION_KEY
);

const props = defineProps<ToggleGroupItemProps>();

const { primitiveElement, currentElement } = usePrimitiveElement();

onMounted(() => {
  injectedValue?.itemsArray.value?.push(currentElement.value!);
});

const state = computed(() => {
  if (injectedValue?.type === "multiple") {
    return injectedValue?.modelValue?.value?.includes(props.value!)
      ? "on"
      : "off";
  } else {
    return injectedValue?.modelValue?.value === props.value ? "on" : "off";
  }
});

const checkedState = computed(() => {
  if (injectedValue?.type === "multiple") {
    return injectedValue?.modelValue?.value?.includes(props.value!);
  } else {
    return injectedValue?.modelValue?.value === props.value;
  }
});

function handleKeydown(e: KeyboardEvent) {
  if (!injectedValue?.rovingFocus || injectedValue?.rootDisabled) {
    return;
  }
  if (
    e.key === "ArrowLeft" ||
    e.key === "ArrowRight" ||
    e.key === "ArrowUp" ||
    e.key === "ArrowDown"
  ) {
    e.preventDefault();
  }
  const newSelectedElement = useArrowNavigation(
    e,
    currentElement.value!,
    injectedValue?.parentElement.value!,
    { loop: injectedValue?.loop }
  );

  if (newSelectedElement) {
    newSelectedElement.focus();
    injectedValue!.currentFocusedElement!.value = newSelectedElement;
  }
}

const getTabIndex = computed(() => {
  if (!injectedValue?.currentFocusedElement?.value) {
    return injectedValue?.modelValue?.value === props.value ? "0" : "-1";
  } else
    return injectedValue?.currentFocusedElement?.value === currentElement.value
      ? "0"
      : "-1";
});

const getRole = computed(() => {
  return injectedValue?.type === "multiple" ? "checkbox" : "radio";
});

function handleChangeValue() {
  if (injectedValue?.rootDisabled) {
    return;
  }
  injectedValue?.changeModelValue(props.value!);
  injectedValue!.currentFocusedElement.value = currentElement.value;
}
</script>

<template>
  <PrimitiveButton
    :as-child="props.asChild"
    ref="primitiveElement"
    type="button"
    :role="getRole"
    :data-state="state"
    :disabled="props.disabled || injectedValue?.rootDisabled"
    :data-disabled="props.disabled ? '' : undefined"
    :aria-checked="checkedState"
    :data-orientation="injectedValue?.orientation"
    @click="handleChangeValue"
    @keydown="handleKeydown"
    data-radix-vue-collection-item
    :tabindex="getTabIndex"
  >
    <slot />
  </PrimitiveButton>
</template>
