<template v-if="content">
    <div class="ww-input-radio" :style="style" ww-responsive="ww-input-radio">
        <div v-for="(option, index) in options" :key="index" class="ww-input-radio__container" :class="{ activeRadio: option.value === value, unactiveRadio: option.value !== value }"
            @click="handleManualInput($event, option.value)">
            <wwLayoutItemContext v-if="option" :index="index" is-repeat>
                <input
                    :id="`${wwElementState.name}-${uniqueId}-${option.label}`"
                    :value="option.value"
                    :checked="option.value === value"
                    class="ww-input-radio__radio"
                    :class="{ editing: isEditing }"
                    type="radio"
                    :name="wwElementState.name"
                    :required="content.required"
                />
                <component
                    :is="isEditing ? 'div' : 'label'"
                    :for="`${wwElementState.name}-${uniqueId}-${option.label}`"
                >
                    <wwElement v-bind="content.choicesElement" :ww-props="{ text: option.label }" />
                </component>
            </wwLayoutItemContext>
        </div>
    </div>
</template>

<script>
import { computed } from 'vue';

export default {
    props: {
        content: { type: Object, required: true },
        /* wwEditor:start */
        wwEditorState: { type: Object, required: true },
        /* wwEditor:end */
        wwElementState: { type: Object, required: true },
        uid: { type: String, required: true },
    },
    emits: ['trigger-event', 'update:content:effect', 'update:sidepanel-content'],
    setup(props) {
        const { value: variableValue, setValue } = wwLib.wwVariable.useComponentVariable({
            uid: props.uid,
            name: 'value',
            defaultValue: computed(() => (props.content.value === undefined ? '' : props.content.value)),
        });
        return { variableValue, setValue, uniqueId: wwLib.wwUtils.getUid() };
    },
    computed: {
        isEditing() {
            /* wwEditor:start */
            return this.wwEditorState.editMode === wwLib.wwEditorHelper.EDIT_MODES.EDITION;
            /* wwEditor:end */
            // eslint-disable-next-line no-unreachable
            return false;
        },
        value() {
            if (!this.options.some(option => option.value === this.variableValue)) return null;
            return this.variableValue;
        },
        options() {
            if (!this.content.options) return [];
            let data = wwLib.wwCollection.getCollectionData(this.content.options) || [];
            if (data && !Array.isArray(data) && typeof data === 'object') {
                data = new Array(data);
            } else if ((data && !Array.isArray(data)) || typeof data !== 'object') {
                return [];
            }

            return data
                .filter(item => !!item)
                .map(item => {
                    if (typeof item !== 'object') return { label: item, value: item };
                    return {
                        label: wwLib.wwLang.getText(_.get(item, this.content.labelField || 'label') || ''),
                        value: _.get(item, this.content.valueField || 'value'),
                    };
                });
        },
        style() {
            return {
                flexDirection: this.content.direction,
                justifyContent: this.content.justifyContent,
                alignItems: this.content.alignItems,
                rowGap: this.content.rowGap,
                columnGap: this.content.columnGap,
                flexWrap:
                    this.content.direction === 'column'
                        ? 'nowrap'
                        : this.content.flexWrap === false
                        ? 'nowrap'
                        : 'wrap',
            };
        },
    },
    watch: {
        /* wwEditor:start */
        'wwEditorState.boundProps.options'(isBind) {
            if (!isBind) this.$emit('update:content:effect', { labelField: null, valueField: null });
        },
        /* wwEditor:end */
        'content.value'(newValue) {
            newValue = `${newValue}`;
            if (newValue === this.value) return;
            this.setValue(newValue);
            this.$emit('trigger-event', { name: 'initValueChange', event: { value: newValue } });
        },
    },
    methods: {
        handleManualInput(event, clickedValue) {
            const value = event.target.value ?? clickedValue; // Clicked value because you can click on a div
            if (value === this.value) return;
            this.setValue(value);
            this.$emit('trigger-event', { name: 'change', event: { domEvent: event, value: [value] } }); // Only in emit wrap value in []test
        },
    },
};
</script>

<style lang="scss" scoped>
.ww-input-radio {
    display: flex;

    &__container {
        padding: 0.4rem 0;
        display: grid;
        grid-template-columns: 1em auto;
        gap: 12px;
        align-items: center;
    }
    &__container label {
        padding: 0.4rem 0;
    }
    &__radio {
        outline: none;
        margin-right: 0.4rem;
        /* wwEditor:start */v
        &.editing {
            pointer-events: none;
        }
        /* wwEditor:end */
    }
}
.unactiveRadio {
    background: white;
    border-radius: 200px;
    border: none;
    color: black;
    padding: 0px 16px;
    text-align: center;
    text-decoration: none;
    margin: 4px 2px;
    border-radius: 32px;
    display: flex;
    gap: 12px;
    cursor: pointer;
    transition: background 200ms ease-in-out;
}
.unactiveRadio:hover {
    background: rgb(227, 227, 227);
}
.unactiveRadio input {
    outline: none;
    margin-right: 0.4rem;
    appearance: none;
    margin: 0;
    font: inherit;
    color: black;
    padding: 8px;
    border:1px solid black;
    border-radius: 50%;
    transform: translateY(-0.075em);
    display: grid;
    place-content: center;
    cursor: pointer;
}

.activeRadio {
    background: linear-gradient(#ef407c, #f2736c);
    border-radius: 200px;
    border: none;
    color: white;
    padding: 0px 16px;
    text-align: center;
    text-decoration: none;
    margin: 4px 2px;
    cursor: pointer;
    border-radius: 32px;
    display: flex;
    gap: 12px;
}

.activeRadio input {
    outline: none;
    margin-right: 0.4rem;
    appearance: none;
    margin: 0;
    font: inherit;
    color: #d5d5d5;
    padding: 4px;
    border:1px solid white;
    border-radius: 50%;
    transform: translateY(-0.075em);
    display: grid;
    place-content: center;
}
.activeRadio input::before {
    content: "";
    width: 0.65em;
    height: 0.65em;
    border-radius: 50%;
    transform: scale(0);
    transition: 120ms transform ease-in-out;
    box-shadow: inset 1em 1em white;
    cursor: pointer;
}
.activeRadio input:checked::before {
    transform: scale(1);
}
label {
  cursor: pointer;
}

</style>