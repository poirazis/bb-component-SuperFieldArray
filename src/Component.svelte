<script>
  import CellString from "./../../bb_super_components_shared/src/lib/SuperTableCells/CellString.svelte";
  import { getContext, onDestroy } from "svelte";
  import SuperButton from "../../bb_super_components_shared/src/lib/SuperButton/SuperButton.svelte";
  import SuperFieldLabel from "../../bb_super_components_shared/src/lib/SuperFieldLabel/SuperFieldLabel.svelte";
  import "../../bb_super_components_shared/src/lib/SuperTableCells/CellCommon.css";
  import "../../bb_super_components_shared/src/lib/SuperFieldsCommon.css";

  const { styleable, enrichButtonActions, Provider } = getContext("sdk");
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
  $: labelPos = field
    ? groupLabelPosition && labelPosition == "fieldGroup"
      ? groupLabelPosition
      : labelPosition
    : false;

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
    error: fieldState?.error,
    role,
    icon,
    showDirty,
    reorderOnly,
    clearIcon: false,
  };

  onDestroy(() => {
    fieldApi?.deregister();
    unsubscribe?.();
  });

  const handleChange = (e, index) => {
    _instances[index] = e.detail;
    _instances = _instances.filter((x) => x);

    onChange?.({ value: _instances });
    fieldApi?.setValue(_instances);

    if (!_instances.length) _instances.push("");
  };

  const validateInstances = () => {
    _instances = _instances.filter((x) => x);
    if (!_instances.length) _instances.push("");
  };
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-noninteractive-tabindex -->
<!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
<div use:styleable={$component.styles}>
  <Provider data={{ value: _instances.filter((x) => x) }} />
  <div class="superField" class:left-label={labelPos == "left"}>
    <SuperFieldLabel
      {labelPos}
      {labelWidth}
      {label}
      {helpText}
      error={fieldState?.error}
    />
    <div class="cells" class:left-label={labelPos == "left"}>
      {#each _instances as _, idx}
        <div class="inline-cells" class:multirow>
          <CellString
            {cellOptions}
            {fieldSchema}
            value={_instances[idx]}
            {autofocus}
            on:change={(e) => handleChange(e, idx)}
            on:focusout={validateInstances}
          />
          <span style="align-self: center">
            <SuperButton
              quiet
              size="M"
              disabled={readonly || disabled}
              type={idx < _instances.length - 1 ? "warning" : "cta"}
              icon={idx < _instances.length - 1
                ? "ri-close-line"
                : "ri-add-line"}
              on:click={() => {
                if (idx < _instances.length - 1) {
                  _instances.splice(idx, 1);
                  _instances = [..._instances];
                  let validInstances = _instances.filter((x) => x);
                  fieldApi?.setValue(validInstances);
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
                  on:click={enrichButtonActions(
                    onClick,
                    $allContext
                  )({ value })}
                />
              {/each}
            </div>
          {/if}
        </div>
      {/each}
    </div>
  </div>
</div>

<style>
  .cells {
    flex: auto;
    display: flex;
    flex-direction: column;
  }
</style>
