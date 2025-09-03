<script>
  import { getContext, onDestroy } from "svelte";
  import {
    SuperButton,
    SuperField,
    CellString,
  } from "@poirazis/supercomponents-shared";

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

  export let field = "Array Field";
  export let controlType = "select";
  export let role = "formInput";

  export let label;
  export let span = 6;
  export let placeholder;
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
  $: error = fieldState?.error;
  $: _instances = value?.length ? value : [""];

  $: $component.styles = {
    ...$component.styles,
    normal: {
      ...$component.styles.normal,
      "grid-column": "span " + span,
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
  <SuperField
    multirow
    {labelPos}
    {labelWidth}
    {field}
    {label}
    {error}
    {helpText}
  >
    <div class="cells">
      {#each _instances as _, idx}
        <div class="row">
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
        </div>
      {/each}
    </div>
  </SuperField>
</div>

<style>
  .cells {
    flex: auto;
    display: flex;
    flex-direction: column;
    align-items: stretch;
  }

  .row {
    flex: auto;
    display: flex;
    align-items: stretch;
    gap: 8px;
  }
</style>
