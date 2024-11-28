<script>
  import CellString from "./../../bb_super_components_shared/src/lib/SuperTableCells/CellString.svelte";
  import { getContext, onDestroy } from "svelte";
  import SuperButton from "../../bb_super_components_shared/src/lib/SuperButton/SuperButton.svelte";
  import "../../bb_super_components_shared/src/lib/SuperTableCells/CellCommon.css";
  import "../../bb_super_components_shared/src/lib/SuperFieldsCommon.css";

  const { styleable, enrichButtonActions } = getContext("sdk");
  const component = getContext("component");
  const allContext = getContext("context");

  const formContext = getContext("form");
  const formStepContext = getContext("form-step");
  const groupLabelPosition = getContext("field-group");
  const labelWidth = getContext("field-group-label-width");
  const groupColumns = getContext("field-group-columns");
  const groupDisabled = getContext("field-group-disabled");
  const formApi = formContext?.formApi;

  export let field;
  export let controlType = "select";
  export let role = "formInput";

  export let buttons = [];

  export let label;
  export let span = 6;
  export let placeholder = "Choose Options";
  export let defaultValue;
  export let disabled;
  export let readonly;
  export let validation;
  export let onChange;
  export let helpText;

  export let icon;
  export let labelPosition = "fieldGroup";
  export let showDirty;
  export let autofocus;

  export let reorderOnly;

  let formField;
  let formStep;
  let fieldState;
  let fieldApi;
  let fieldSchema;
  let value;
  let showHelp;

  if (defaultValue) value = defaultValue.split(",");

  $: multirow = controlType != "select" && controlType != "inputSelect";
  $: formStep = formStepContext ? $formStepContext || 1 : 1;
  $: labelPos =
    groupLabelPosition && labelPosition == "fieldGroup"
      ? groupLabelPosition
      : labelPosition;

  $: formField = formApi?.registerField(
    field,
    "array",
    defaultValue?.split[","],
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
  $: _instances = value?.length ? value : [""];

  $: $component.styles = {
    ...$component.styles,
    normal: {
      ...$component.styles.normal,
      "grid-column": span < 7 ? "span " + span : "span " + groupColumns * 6,
    },
  };

  $: cellOptions = {
    disabled: disabled || groupDisabled || fieldState?.disabled,
    readonly: readonly || fieldState?.readonly,
    debounce: false,
    placeholder,
    defaultValue,
    error: fieldState.error,
    role,
    icon,
    showDirty,
    reorderOnly,
  };

  onDestroy(() => {
    fieldApi?.deregister();
    unsubscribe?.();
  });

  const handleChange = (e, index) => {
    _instances[index] = e.detail;
    let validInstances = _instances.filter((x) => x);

    onChange?.({ value: validInstances });
    fieldApi?.setValue([...validInstances]);

    if (!_instances.length) _instances.push("");
  };
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-noninteractive-tabindex -->
<!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
<div use:styleable={$component.styles}>
  <div
    class="superField"
    class:left-label={labelPos == "left"}
    class:multirow={controlType != "select"}
    style:--label-width={labelPos == "left"
      ? labelWidth
        ? labelWidth
        : "6rem"
      : "auto"}
  >
    {#if labelPos}
      <label for="superCell" class="superlabel" class:left={labelPos == "left"}>
        {#if helpText && labelPos != "left"}
          <!-- svelte-ignore a11y-no-static-element-interactions -->
          <i
            class={"ri-question-line"}
            on:mouseenter={() => (showHelp = true)}
            on:mouseleave={() => (showHelp = false)}
          />
        {/if}
        <span>
          {#if showHelp && helpText}
            {helpText}
          {:else}
            {label || field}
          {/if}
        </span>
        {#if fieldState.error}
          <div class="error" class:left={labelPos == "left"}>
            {fieldState.error}
          </div>
        {/if}
      </label>
    {/if}
    {#each _instances as _, idx}
      <div class="inline-cells" class:multirow>
        <CellString
          {cellOptions}
          {fieldSchema}
          value={_instances[idx]}
          {autofocus}
          on:change={(e) => handleChange(e, idx)}
        />
        <span style="align-self: center">
          <SuperButton
            quiet
            size="S"
            icon={idx < _instances.length - 1
              ? "ri-delete-bin-line"
              : "ri-add-line"}
            on:click={() => {
              if (idx < _instances.length - 1) {
                _instances.splice(idx, 1);
                _instances = [..._instances];
                let validInstances = _instances.filter((x) => x);
                fieldApi?.setValue([validInstances]);
              } else if (_instances[idx]) {
                _instances = [..._instances, ""];
              }
            }}
          />
        </span>
        {#if buttons?.length && controlType != "list"}
          <div class="inline-buttons" class:vertical={multirow}>
            {#each buttons as { text, onClick, quiet, disabled, type, size }}
              <SuperButton
                {quiet}
                {disabled}
                {size}
                {type}
                {text}
                on:click={enrichButtonActions(onClick, $allContext)({ value })}
              />
            {/each}
          </div>
        {/if}
      </div>
    {/each}
  </div>
</div>
