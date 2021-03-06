<template>
    <div class="dropdown"
        :class="[position, {
            'is-disabled': disabled,
            'is-hoverable': hoverable,
            'is-inline': inline,
            'is-active': isActive || inline,
            'is-mobile-modal': isMobileModal
        }]">
        <div v-if="!inline"
            role="button"
            ref="trigger"
            class="dropdown-trigger"
            @click="toggle">
            <slot name="trigger"></slot>
        </div>

        <transition :name="transition">
            <div v-if="isMobileModal"
                v-show="isActive"
                class="background">
            </div>
        </transition>
        <transition :name="transition">
            <div v-show="isActive || hoverable || inline"
                ref="dropdownMenu"
                class="dropdown-menu">
                <div class="dropdown-content">
                    <slot></slot>
                </div>
            </div>
        </transition>
    </div>
</template>

<script>
    export default {
        name: 'bDropdown',
        props: {
            value: {
                type: [String, Number, Boolean, Object, Array, Symbol, Function],
                default: null
            },
            disabled: Boolean,
            hoverable: Boolean,
            inline: Boolean,
            position: {
                type: String,
                validator(value) {
                    return [
                        'is-top-right',
                        'is-top-left',
                        'is-bottom-left'
                    ].indexOf(value) > -1
                }
            },
            transition: {
                type: String,
                default: 'fade'
            },
            mobileModal: {
                type: Boolean,
                default: true
            }
        },
        data() {
            return {
                selected: this.value,
                isActive: false,
                _isDropdown: true // Used internally by DropdownItem
            }
        },
        computed: {
            isMobileModal() {
                return this.mobileModal && !this.inline && !this.hoverable
            }
        },
        watch: {
            /**
             * When v-model is changed set the new selected item.
             */
            value(value) {
                this.selectItem(value)
            },

            /**
             * Emit event when isActive value is changed.
             */
            isActive(value) {
                this.$emit('active-change', value)
            }
        },
        methods: {
            /**
             * Click listener from DropdownItem.
             *   1. Set new selected item.
             *   2. Emit input event to update the user v-model.
             *   3. Close the dropdown.
             */
            selectItem(value) {
                this.selected = value
                this.$emit('input', value)
                this.$emit('change', value)
                this.isActive = false
            },

            /**
             * White-listed items to not close when clicked.
             */
            isInWhiteList(el) {
                if (el === this.$refs.dropdownMenu) return true
                if (el === this.$refs.trigger) return true
                // All chidren from dropdown
                if (this.$refs.dropdownMenu !== undefined) {
                    const children = this.$refs.dropdownMenu.querySelectorAll('*')
                    for (const child of children) {
                        if (el === child) {
                            return true
                        }
                    }
                }
                // All children from trigger
                if (this.$refs.trigger !== undefined) {
                    const children = this.$refs.trigger.querySelectorAll('*')
                    for (const child of children) {
                        if (el === child) {
                            return true
                        }
                    }
                }

                return false
            },

            /**
             * Close dropdown if clicked outside.
             */
            clickedOutside(event) {
                if (this.inline) return

                if (!this.isInWhiteList(event.target)) this.isActive = false
            },

            /**
             * Toggle dropdown if it's not disabled.
             */
            toggle() {
                if (this.disabled || this.hoverable) return

                this.isActive = !this.isActive
            }
        },
        created() {
            if (typeof window !== 'undefined') {
                document.addEventListener('click', this.clickedOutside)
            }
        },
        beforeDestroy() {
            if (typeof window !== 'undefined') {
                document.removeEventListener('click', this.clickedOutside)
            }
        }
    }
</script>
