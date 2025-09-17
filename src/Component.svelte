<script>
  import { getContext, onDestroy } from "svelte";
  import { SuperField, CellString } from "@poirazis/supercomponents-shared";
  import { dndzone } from "svelte-dnd-action";
  import { generate } from "shortid";

  const { styleable, Provider, builderStore, memo, derivedMemo } =
    getContext("sdk");
  const component = getContext("component");

  const formContext = getContext("form");
  const formStepContext = getContext("form-step");
  const groupLabelPosition = getContext("field-group");
  const labelWidth = getContext("field-group-label-width");
  const groupColumns = getContext("field-group-columns");
  const groupDisabled = getContext("field-group-disabled");
  const formApi = formContext?.formApi;

  export let fieldType = "string"; // string or array
  export let field;
  export let fieldString;
  export let fieldArray;
  export let role = "formInput";

  export let label;
  export let span = 6;
  export let placeholder;
  export let defaultValue;
  export let disabled;
  export let readonly;
  export let validation;
  export let invisible = false;
  export let onChange;
  export let helpText;
  export let reorder = "handle";

  export let icon;
  export let labelPosition = "fieldGroup";
  export let showDirty;

  let formField;
  let formStep;
  let fieldState;
  let fieldApi;
  let fieldSchema;
  let value;
  let stringify = false;

  let zoneType = generate();
  let draggableItems = [];
  let inactive = true;
  let focusedRowIndex = -1;
  let cell;
  let cellApi = [];
  let cellValues = memo([]);

  $: if (defaultValue) cellValues.set(enrichValue(defaultValue));
  $: field =
    fieldType === "string" && fieldString
      ? fieldString
      : fieldType === "array" && fieldArray
        ? fieldArray
        : field;

  $: stringify = fieldType === "string";

  $: formStep = formStepContext ? $formStepContext || 1 : 1;
  $: labelPos = field
    ? groupLabelPosition && labelPosition == "fieldGroup"
      ? groupLabelPosition
      : labelPosition
    : false;

  $: formField = formApi?.registerField(
    field,
    fieldType === "array" ? "array" : "string",
    defaultValue,
    disabled,
    readonly,
    validation,
    formStep
  );

  $: unsubscribe = formField?.subscribe((value) => {
    fieldState = value?.fieldState;
    fieldApi = value?.fieldApi;
    fieldSchema = value?.fieldSchema;
  });

  $: value = fieldState?.value;
  $: error = fieldState?.error;

  $: cellValues.set(enrichValue(value));

  $: outputValue = derivedMemo(cellValues, ($cellValues) => {
    return stringify
      ? $cellValues.filter((x) => x).join(",")
      : $cellValues.filter((x) => x);
  });

  $: fieldApi?.setValue($outputValue);
  $: onChange?.({ value: $outputValue });

  $: draggableItems = $cellValues.map((item, index) => ({
    id: `item-${index}`,
    value: item,
    index,
  }));

  $: cellComponent = CellString;

  $: $component.styles = {
    ...$component.styles,
    normal: {
      ...$component.styles.normal,
      display:
        invisible && !$builderStore.inBuilder
          ? "none"
          : $component.styles.normal.display,
      opacity: invisible && $builderStore.inBuilder ? 0.6 : 1,
      "grid-column": groupColumns ? `span ${span}` : "span 1",
      "max-height": $component.styles.normal.height || "15rem",
    },
  };

  $: cellOptions = {
    disabled: disabled || groupDisabled || fieldState?.disabled,
    readonly: readonly || fieldState?.readonly,
    debounce: false,
    placeholder,
    defaultValue,
    error: fieldState?.error,
    role,
    icon,
    showDirty,
    clearIcon: false,
    role: "inlineInput",
    background: "var(--spectrum-global-color-gray-50)",
    optionsViewMode: "pills",
  };

  const enrichValue = (val) => {
    if (Array.isArray(val)) {
      return val.length ? [...val.filter((x) => x)] : [""];
    } else if (val) {
      return [
        ...val
          .split(",")
          .map((x) => x.trim())
          .filter((x) => x),
      ];
    } else {
      return [""];
    }
  };

  const handleChange = (e, index) => {
    cellApi.forEach((api) => api?.clearError());

    // Check for duplicates, excluding the current index
    const isDuplicate = $cellValues.some(
      (val, i) => i !== index && val === e.detail.toString() && val !== ""
    );

    if ($cellValues[index] !== e.detail && !isDuplicate) {
      $cellValues[index] = e.detail.toString();
    } else if (isDuplicate) {
      cellApi[index]?.setError("Duplicate value");
      cellApi[index]?.focus();
    } else if (!e.detail) {
      cellApi = cellApi.splice(index, 1);
      $cellValues = $cellValues.splice(index, 1);
    }
  };

  const validateInstances = (idx) => {
    if (!inactive) return;

    let filtered = $cellValues.filter((x) => x);
    if (filtered.length === 0) filtered = [""];
    cellValues.set([...filtered]);
    cellApi.forEach((api, index) => {
      if (api) api.clearError();
    });
  };

  const moveItem = (fromIndex, toIndex) => {
    if (toIndex < 0 || toIndex >= $cellValues.length) return;
    const item = $cellValues.splice(fromIndex, 1)[0];
    $cellValues.splice(toIndex, 0, item);
    $cellValues = [...$cellValues];

    // Update focused row index if it was moved
    if (focusedRowIndex === fromIndex) {
      focusedRowIndex = toIndex;
    } else if (
      focusedRowIndex >= 0 &&
      focusedRowIndex > fromIndex &&
      focusedRowIndex <= toIndex
    ) {
      focusedRowIndex--; // Adjust if affected by move
    }

    cellApi[focusedRowIndex].focus();
  };

  const updateRowOrder = (e) => {
    focusedRowIndex = -1; // Use -1 instead of undefined for consistency
    draggableItems = e.detail.items;
  };

  const handleFinalize = (e) => {
    inactive = true;
    updateRowOrder(e);
    $cellValues = draggableItems.map((item) => item.value);
  };

  const handleDragStart = () => {
    inactive = false;
    focusedRowIndex = -1; // Clear focus to fix drag issues when cell is focused
  };

  const handleDragEnd = () => {
    inactive = true;
  };

  const handleRemove = (idx) => {
    $cellValues.splice(idx, 1);
    if ($cellValues.length === 0) {
      $cellValues.push("");
    }
    $cellValues = [...$cellValues];

    // Update focused row index
    if (focusedRowIndex >= $cellValues.length) {
      focusedRowIndex = $cellValues.length - 1;
    } else if (focusedRowIndex > idx) {
      focusedRowIndex--;
    } else if (focusedRowIndex === idx) {
      focusedRowIndex = -1;
    }
  };

  const handleAdd = () => {
    cellApi.forEach((api) => api?.clearError());

    const lastValue = ($cellValues[$cellValues.length - 1] || "")
      .toString()
      .trim();
    if (!lastValue) {
      cellApi[$cellValues.length - 1]?.setError("Value cannot be empty");
      cellApi[$cellValues.length - 1]?.focus();
      return;
    } else {
      $cellValues = [...$cellValues, ""];
      focusedRowIndex = $cellValues.length - 1;
    }

    // Check for duplicates before adding a new row
    const isDuplicate = $cellValues
      .slice(0, -1)
      .some((val) => val.toString().trim() === lastValue);
    if (isDuplicate) {
      cellApi[$cellValues.length - 1]?.setError("Duplicate value");
      cellApi[$cellValues.length - 1]?.focus();
      return;
    }
  };

  const moveRowUp = (index) => {
    if (index > 0 && reorder !== "disabled") {
      const newIndex = index - 1;
      moveItem(index, newIndex);
      focusedRowIndex = newIndex;
    }
  };

  const moveRowDown = (index) => {
    if (index < $cellValues.length - 1 && reorder !== "disabled") {
      const newIndex = index + 1;
      moveItem(index, newIndex);
      focusedRowIndex = newIndex;
    }
  };

  const handleKeyDown = (event) => {
    // Handle Enter key to add a row if the last row is focused and not empty
    if (
      event.key === "Enter" &&
      focusedRowIndex === $cellValues.length - 1 &&
      $cellValues[focusedRowIndex]?.trim()
    ) {
      event.preventDefault();
      event.stopPropagation();
      handleAdd(); // Add a new row
    }

    // Only handle keyboard shortcuts when reordering is enabled
    if (reorder === "disabled") return;

    // Check for Cmd/Ctrl + Up/Down
    if ((event.metaKey || event.ctrlKey) && !event.shiftKey && !event.altKey) {
      if (event.key === "ArrowUp" || event.key === "ArrowDown") {
        event.preventDefault();
        event.stopPropagation();

        if (event.key === "ArrowUp") {
          moveRowUp(focusedRowIndex);
        } else if (event.key === "ArrowDown") {
          moveRowDown(focusedRowIndex);
        }
      }
    }
  };

  onDestroy(() => {
    fieldApi?.deregister();
    unsubscribe?.();
  });
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-noninteractive-tabindex -->
<!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
<div use:styleable={$component.styles}>
  <Provider data={{ value: $cellValues.filter((x) => x) }} /> ssss
  <SuperField
    multirow
    {labelPos}
    {labelWidth}
    {field}
    {label}
    {error}
    {helpText}
  >
    <div bind:this={cell} class="cell">
      {#if reorder !== "disabled"}
        <div
          class="cells"
          class:readonly={readonly || disabled}
          tabindex="-1"
          use:dndzone={{
            items: draggableItems,
            dropTargetStyle: {
              outline: "none",
              border: "1px dashed var(--spectrum-global-color-blue-700)",
            },
            dragDisabled:
              readonly || disabled || inactive || reorder === "disabled",
            type: zoneType,
            dropFromOthersDisabled: true,
          }}
          on:finalize={handleFinalize}
          on:consider={updateRowOrder}
        >
          {#each draggableItems as draggableItem, idx (draggableItem.id)}
            <!-- svelte-ignore a11y-no-static-element-interactions -->
            <div
              class="row"
              tabindex={readonly || disabled ? -1 : 0}
              class:focused={focusedRowIndex === idx}
              on:focusin={() =>
                (focusedRowIndex = readonly || disabled ? -1 : idx)}
              on:focusout={(e) => {
                if (!cell.contains(e.relatedTarget)) validateInstances();
                focusedRowIndex = -1;
              }}
              on:keydown={handleKeyDown}
            >
              {#if reorder === "handle" || reorder === "full"}
                <!-- svelte-ignore a11y-interactive-supports-focus -->
                <div
                  class="drag-handle"
                  class:readonly={readonly || disabled}
                  style={inactive ? "cursor: grab" : "cursor: grabbing"}
                  role="button"
                  tabindex="-1"
                  on:mousedown={handleDragStart}
                  on:mouseup={handleDragEnd}
                  on:mouseleave={handleDragEnd}
                >
                  <i
                    class={focusedRowIndex == idx
                      ? "ph ph-pencil-simple"
                      : "ph ph-dots-six-vertical"}
                  ></i>
                </div>
              {/if}
              <svelte:component
                this={cellComponent}
                bind:cellApi={cellApi[idx]}
                {cellOptions}
                {fieldSchema}
                value={draggableItem.value}
                autofocus={focusedRowIndex === idx}
                on:change={(e) => handleChange(e, idx)}
              />
              <div class="action-buttons">
                {#if reorder === "full"}
                  <button
                    class="action-button"
                    disabled={readonly || disabled || idx === 0}
                    on:click={() => moveItem(idx, idx - 1)}
                    aria-label="Move up"
                  >
                    <i class="ph ph-caret-up"></i>
                  </button>
                  <button
                    class="action-button"
                    disabled={readonly ||
                      disabled ||
                      idx === $cellValues.length - 1}
                    on:click={() => moveItem(idx, idx + 1)}
                    aria-label="Move down"
                  >
                    <i class="ph ph-caret-down"></i>
                  </button>
                {/if}
                <button
                  class="action-button"
                  class:delete={idx < $cellValues.length - 1}
                  disabled={readonly || disabled}
                  on:click={() => {
                    if (idx < $cellValues.length - 1) {
                      handleRemove(idx);
                    } else {
                      handleAdd();
                    }
                  }}
                  aria-label={idx < $cellValues.length - 1 ? "Remove" : "Add"}
                >
                  <i
                    class={idx < $cellValues.length - 1
                      ? "ph ph-trash-simple"
                      : "ph ph-plus"}
                  ></i>
                </button>
              </div>
            </div>
          {/each}
        </div>
      {:else}
        <div class="cells" tabindex="-1">
          {#each $cellValues as _, idx}
            <!-- svelte-ignore a11y-no-static-element-interactions -->
            <div
              class="row"
              class:focused={focusedRowIndex === idx}
              on:keydown={handleKeyDown}
              on:focusin={() => (focusedRowIndex = idx)}
              on:focusout={(e) => {
                if (!cell.contains(e.relatedTarget)) validateInstances();
                focusedRowIndex = -1;
              }}
            >
              <svelte:component
                this={cellComponent}
                bind:cellApi={cellApi[idx]}
                {cellOptions}
                {fieldSchema}
                value={$cellValues[idx]}
                autofocus={focusedRowIndex === idx}
                on:change={(e) => handleChange(e, idx)}
              />
              <div class="action-buttons">
                <button
                  class="action-button"
                  disabled={readonly || disabled}
                  on:click={() => {
                    if (idx < $cellValues.length - 1) {
                      handleRemove(idx);
                    } else {
                      handleAdd();
                    }
                  }}
                  aria-label={idx < $cellValues.length - 1 ? "Remove" : "Add"}
                >
                  <i
                    class={idx < $cellValues.length - 1
                      ? "ph ph-trash-simple"
                      : "ph ph-plus"}
                  ></i>
                </button>
              </div>
            </div>
          {/each}
        </div>
      {/if}
    </div>
  </SuperField>
</div>

<style>
  .cell {
    display: flex;
    flex-direction: column;
    align-items: stretch;
    gap: 0.25rem;
    width: 100%;
  }
  .cells {
    flex: auto;
    display: flex;
    flex-direction: column;
    align-items: stretch;
    border: 1px solid var(--spectrum-global-color-gray-300);
    background-color: var(--spectrum-global-color-gray-50);
    border-radius: 2px;
    overflow: auto;
    min-height: 2rem;
    max-height: 100%;

    &:focus-within:not(.readonly):not(.disabled) {
      border-color: var(--spectrum-global-color-blue-400);
    }

    &:focus-within.readonly {
      border: 1px dashed var(--spectrum-global-color-blue-400);
    }

    &.readeonly {
      border-color: var(--spectrum-global-color-gray-200);
    }
    &.disabled {
      border-color: var(--spectrum-global-color-gray-300);
      background-color: var(--spectrum-global-color-gray-100);
    }
  }

  .row {
    flex: auto;
    display: flex;
    align-items: stretch;
    user-select: none; /* Prevent text selection during drag */
    border-bottom: 1px solid var(--spectrum-global-color-gray-300);

    &.focused {
      & > .drag-handle {
        color: var(--spectrum-global-color-blue-400) !important;
      }
    }

    &:focus {
      outline: none;
    }

    &:last-child {
      border-bottom: none;
    }
  }

  .drag-handle {
    display: flex;
    align-items: center;
    justify-content: center;
    min-width: 26px;
    height: 30px;
    cursor: grab;
    color: var(--spectrum-global-color-gray-700);
    transition: background-color 0.2s;
    user-select: none; /* Prevent text selection */
    border-right: 1px solid var(--spectrum-global-color-gray-100);
    font-weight: 600;

    &:hover:not(.readonly) {
      color: var(--spectrum-global-color-gray-900);
    }

    &.readonly {
      cursor: default;
      opacity: 0.5;
    }

    &:active:not(.readonly) {
      cursor: grabbing;
      background-color: var(--spectrum-global-color-gray-200);
    }
  }

  .action-buttons {
    display: flex;
    flex-direction: row;
    gap: 4px;
    align-items: center;
    padding: 0rem 0.25rem;
  }

  .action-button {
    all: unset;
    cursor: pointer;
    border-radius: 4px;
    width: 24px;
    height: 24px;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: transparent;
    color: var(--spectrum-global-color-gray-700);
    font-size: 12px;
    transition:
      background-color 0.2s,
      color 0.2s;

    &:hover:not(:disabled) {
      background-color: var(--spectrum-global-color-gray-100);
      color: var(--spectrum-global-color-gray-900);
    }

    &:hover.delete {
      color: var(--spectrum-global-color-red-600);
    }
    &:disabled {
      cursor: not-allowed;
      opacity: 0.5;
    }

    &:focus {
      outline: 1px dashed var(--spectrum-global-color-blue-600);
    }
  }
</style>
