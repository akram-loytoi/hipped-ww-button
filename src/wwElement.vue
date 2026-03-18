<template>
    <component
        :is="tag"
        class="ww-button"
        :class="{ button: tag, '-link': hasLink && !isEditing, active: isActive }"
        :type="buttonType"
        :style="buttonStyle"
        :data-ww-flag="'btn-' + content.buttonType"
        :disabled="content.disabled || content.isLoading || isSuccess"
        v-bind="properties"
        @focus="isReallyFocused = true"
        @blur="onBlur($event)"
        @mousedown="onMouseActivate"
        @mouseup="onMouseDeactivate"
        @mouseleave="onMouseDeactivate"
        @touchstart="onTouchActivate"
        @touchend="onTouchDeactivate"
        @touchcancel="onTouchDeactivate"
        @keydown.enter="onKeyActivate"
        @keydown.space="onKeyActivate"
        @keyup.enter="onKeyDeactivate"
        @keyup.space="onKeyDeactivate"
        @keydown="onKeyDown"
        @keyup="onKeyUp"
    >
        <template v-if="content.isLoading">
            <wwElement
                v-if="content.hasLoadingIcon && content.loadingIcon"
                v-bind="content.loadingIcon">
            </wwElement>
            <span v-else class="ww-button__spinner"></span>
        </template>
        <template v-else-if="isSuccess">
            <wwElement
                v-if="content.hasSuccessIcon && content.successIcon"
                v-bind="content.successIcon">
            </wwElement>
            <span v-else class="ww-button__checkmark">✓</span>
        </template>
        <wwElement v-if="content.hasLeftIcon && content.leftIcon && !content.isLoading && !isSuccess" v-bind="content.leftIcon"></wwElement>
        <wwText tag="span" :text="displayText"></wwText>
        <wwElement v-if="content.hasRightIcon && content.rightIcon && !content.isLoading && !isSuccess" v-bind="content.rightIcon"></wwElement>
    </component>
</template>

<script>
import { computed } from 'vue';
const TEXT_ALIGN_TO_JUSTIFY = {
    center: 'center',
    right: 'flex-end',
    left: 'flex-start',
    justify: 'center',
};
export default {
    props: {
        content: { type: Object, required: true },
        wwFrontState: { type: Object, required: true },
        wwElementState: { type: Object, required: true },
        /* wwEditor:start */
        wwEditorState: { type: Object, required: true },
        /* wwEditor:end */
    },
    emits: [
        'update:content',
        'update:content:effect',
        'change-menu-visibility',
        'change-borders-style',
        'add-state',
        'remove-state',
        'trigger-event',
    ],
    setup(props) {
        /* wwEditor:start */
        const { createElement } = wwLib.useCreateElement();
        /* wwEditor:end */
        const {
            hasLink,
            tag: linkTag,
            properties,
        } = wwLib.wwElement.useLink({
            isDisabled: computed(() => props.content.disabled),
        });
        return {
            /* wwEditor:start */
            createElement,
            /* wwEditor:end */
            hasLink,
            linkTag,
            properties,
        };
    },

    data() {
        return {
            isReallyFocused: false,
            isReallyActive: false,
            isSuccess: false,
        };
    },
    computed: {
        buttonStyle() {
            return {
                justifyContent: TEXT_ALIGN_TO_JUSTIFY[this.content['_ww-text_textAlign']] || 'center',
            };
        },
        isEditing() {
            /* wwEditor:start */
            return this.wwEditorState.editMode === wwLib.wwEditorHelper.EDIT_MODES.EDITION;
            /* wwEditor:end */
            // eslint-disable-next-line no-unreachable
            return false;
        },
        tag() {
            if (this.isEditing) return 'div';
            if (this.hasLink) {
                return this.linkTag;
            }
            if (
                this.content.buttonType === 'submit' ||
                this.content.buttonType === 'reset' ||
                this.content.buttonType === 'button'
            )
                return 'button';
            return 'div';
        },
        buttonType() {
            if (this.isEditing || this.hasLink) return '';
            if (
                this.content.buttonType === 'submit' ||
                this.content.buttonType === 'reset' ||
                this.content.buttonType === 'button'
            )
                return this.content.buttonType;
            return '';
        },
        text() {
            return this.wwElementState.props.text;
        },
        displayText() {
            if (this.isSuccess) return this.content.successLabel || 'Done!';
            if (this.content.isLoading) return this.content.loadingLabel || 'Loading...';
            return this.text;
        },
        isFocused() {
            /* wwEditor:start */
            if (this.wwEditorState.isSelected) {
                return this.wwElementState.states.includes('focus');
            }
            /* wwEditor:end */
            return this.isReallyFocused;
        },
        isActive() {
            /* wwEditor:start */
            if (this.wwEditorState.isSelected) {
                return this.wwElementState.states.includes('active');
            }
            /* wwEditor:end */
            return this.isReallyActive;
        },
    },
    watch: {
        /* wwEditor:start */
        'content.hasLoadingIcon': {
            async handler(hasLoadingIcon) {
                if (this.wwEditorState.isACopy) return;
                if (hasLoadingIcon && !this.content.loadingIcon) {
                    // 'aa27b26f-0686-4c29-98c5-8217044045b7' is the stable base component ID for ww-loader
                    // in the Hipped WeWeb workspace. Update this UUID if the workspace changes.
                    const content = await this.createElement('aa27b26f-0686-4c29-98c5-8217044045b7');
                    this.$emit('update:content:effect', { loadingIcon: content });
                }
            },
        },
        'content.hasSuccessIcon': {
            async handler(hasSuccessIcon) {
                if (this.wwEditorState.isACopy) return;
                if (hasSuccessIcon && !this.content.successIcon) {
                    const content = await this.createElement('ww-icon');
                    this.$emit('update:content:effect', { successIcon: content });
                }
            },
        },
        'content.hasRightIcon': {
            async handler(hasRightIcon) {
                if (this.wwEditorState.isACopy) {
                    return;
                }
                if (hasRightIcon && !this.content.rightIcon) {
                    const content = await this.createElement('ww-icon');
                    this.$emit('update:content:effect', { rightIcon: content });
                }
            },
        },
        'content.hasLeftIcon': {
            async handler(hasLeftIcon) {
                if (this.wwEditorState.isACopy) {
                    return;
                }
                if (hasLeftIcon && !this.content.leftIcon) {
                    const content = await this.createElement('ww-icon');
                    this.$emit('update:content:effect', { leftIcon: content });
                }
            },
        },
        /* wwEditor:end */
        'content.isLoading': {
            handler(isLoading, wasLoading) {
                if (wasLoading && !isLoading) {
                    this.isSuccess = true;
                    setTimeout(() => {
                        this.isSuccess = false;
                        this.$emit('remove-state', 'loading');
                    }, 1500);
                } else if (isLoading) {
                    this.$emit('add-state', 'loading');
                } else {
                    this.$emit('remove-state', 'loading');
                }
            },
        },
        'content.disabled': {
            immediate: true,
            handler(value) {
                if (value) {
                    this.$emit('add-state', 'disabled');
                } else {
                    this.$emit('remove-state', 'disabled');
                }
            },
        },
        isReallyFocused(isFocused, wasFocused) {
            if (isFocused && !wasFocused) {
                this.$emit('trigger-event', { name: 'focus' });
            } else if (!isFocused && wasFocused) {
                this.$emit('trigger-event', { name: 'blur' });
            }
        },
        isFocused: {
            immediate: true,
            handler(value) {
                if (value) {
                    this.$emit('add-state', 'focus');
                } else {
                    this.$emit('remove-state', 'focus');
                }
            },
        },
        isActive: {
            immediate: true,
            handler(value) {
                if (value) {
                    this.$emit('add-state', 'active');
                } else {
                    this.$emit('remove-state', 'active');
                }
            },
        },
    },
    methods: {
        /* wwEditor:start */
        selectParentFormContainer() {
            const parentUid = wwLib.selectParentByFlag(this.$el, 'form-container');
            if (!parentUid) {
                wwLib.wwNotification.open({
                    text: {
                        en: 'No parent form container found. Please, add this submit button into a form container.',
                        fr: 'Aucun formulaire parent trouvé. Veuillez intégrer ce bouton submit dans un form container.',
                    },
                    color: 'yellow',
                    duration: 6000,
                });
            }
        },
        /* wwEditor:end */
        onBlur() {
            this.isReallyFocused = false;
        },
        onActivate(event) {
            this.isReallyActive = true;
            // Emit the original event name
            const eventName = event.type;
            this.$emit('trigger-event', { name: eventName, event });
        },
        onDeactivate(event) {
            this.isReallyActive = false;
            // Emit the original event name
            const eventName = event.type;
            this.$emit('trigger-event', { name: eventName, event });
        },
        onTouchActivate() {
            this.isReallyActive = true;
        },
        onTouchDeactivate() {
            this.isReallyActive = false;
        },
        onMouseActivate() {
            this.isReallyActive = true;
        },
        onMouseDeactivate() {
            this.isReallyActive = false;
        },
        onKeyActivate() {
            this.isReallyActive = true;
        },
        onKeyDeactivate() {
            this.isReallyActive = false;
        },
        onKeyDown(event) {
            this.$emit('trigger-event', { name: 'keydown', event });
        },
        onKeyUp(event) {
            this.$emit('trigger-event', { name: 'keyup', event });
        },
    },
};
</script>

<style lang="scss" scoped>
.ww-button {
    justify-content: center;
    align-items: center;
    gap: 8px;
    &.button {
        outline: none;
        border: none;
        background: none;
        font-family: inherit;
        font-size: inherit;
    }
    &.-link {
        cursor: pointer;
    }
    &__spinner {
        display: inline-block;
        width: 14px;
        height: 14px;
        border: 2px solid currentColor;
        border-top-color: transparent;
        border-radius: 50%;
        animation: ww-button-spin 0.7s linear infinite;
        flex-shrink: 0;
    }
    &__checkmark {
        font-size: 14px;
        line-height: 1;
        flex-shrink: 0;
    }
}
@keyframes ww-button-spin {
    to { transform: rotate(360deg); }
}
</style>
