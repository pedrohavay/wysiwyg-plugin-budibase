<script lang="ts">
    import 'quill/dist/quill.snow.css';

    import { onMount, getContext, onDestroy } from "svelte";
    import Quill from "quill";

    export let fieldText;
    export let fieldType;
    export let label;
    export let readOnly;
    export let validationText;

    const { styleable } = getContext("sdk");
    const component = getContext("component");
    const formContext = getContext("form");
    const formStepContext = getContext("form-step");
    const fieldGroupContext = getContext("field-group");

    let fieldState;
    let fieldApi;
    let errors;
    let content;
    let unsubscribe;
    let quill;

    const formApi = formContext?.formApi;
    const labelPos = fieldGroupContext?.labelPosition || "above";
    let configErrors = "";

    $: {
        configErrors = "";

        if (!fieldText) {
            configErrors = "Please select a form field\n";
        }

        if (fieldType !== "text") {
            configErrors = configErrors + "Select a field type\n";
        }

        if (validationText) {
            configErrors =
                configErrors +
                "Can only use Text validation with Text typed field\n";
        }
    }

    $: formStep = formStepContext ? $formStepContext || 1 : 1;
    $: labelClass =
        labelPos === "above" ? "" : `spectrum-FieldLabel--${labelPos}`;

    $: if (formApi && fieldText && !configErrors) {
        const formField = formApi.registerField(
            fieldText,
            fieldType,
            "null",
            false,
            validationText,
            formStep,
        );

        unsubscribe = formField?.subscribe((value) => {
            fieldState = value?.fieldState;
            fieldApi = value?.fieldApi;
        });
    }

    onMount(() => {
        if (fieldState?.value !== undefined && content === undefined) {
            content = fieldState?.value;
        }

        quill = new Quill("#editor", {
            theme: "snow",
            readOnly: readOnly,
        });

        if (content) {
            quill.root.innerHTML = content;
        }

        quill.on("text-change", () => {
            const value = quill.root.innerHTML;
            fieldApi.setValue(value);
        });
    });

    onDestroy(() => {
        fieldApi?.deregister();
        unsubscribe?.();
    });
</script>

<div class="spectrum-Form-item" use:styleable={$component.styles}>
    {#if !formContext}
        <div class="placeholder">
            Form components need to be wrapped in a form
        </div>
    {:else if !configErrors}
        <label
            class:hidden={!label}
            for={fieldState?.fieldId}
            class={`spectrum-FieldLabel spectrum-FieldLabel--sizeM spectrum-Form-itemLabel ${labelClass}`}
        >
            {label || " "}
        </label>
        <div class="spectrum-Form-itemField">
            <div class="container">
                <div id="editor"></div>
            </div>
        </div>
        {#if errors}
            <div class="error">{errors}</div>
        {/if}
        {#if fieldState?.error}
            <div class="error">{fieldState.error}</div>
        {/if}
    {:else}
        <div class="error">{configErrors}</div>
    {/if}
</div>

<style>
    .container {
        display: flex;
        min-height: 42px;
        align-items: center;
        resize: both;
        overflow: hidden;
    }
    .placeholder {
        color: var(--spectrum-global-color-gray-600);
    }
    label {
        white-space: nowrap;
        display: block;
    }
    label.hidden {
        padding: 0;
    }
    .spectrum-Form-item {
        display: block;
    }
    .spectrum-Form-itemField {
        width: 100%;
        align-items: center;
        display: block;
    }
    .error {
        color: var(
            --spectrum-semantic-negative-color-default,
            var(--spectrum-global-color-red-500)
        );
        font-size: var(--spectrum-global-dimension-font-size-75);
        margin-top: var(--spectrum-global-dimension-size-75);
    }
    .spectrum-FieldLabel--right,
    .spectrum-FieldLabel--left {
        padding-right: var(--spectrum-global-dimension-size-200);
    }
    #editor {
        height: 200px;
        border: 1px solid #ccc;
    }
</style>
