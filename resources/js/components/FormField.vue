<template>
    <DefaultField
        :id="field.attribute"
        :field="field"
        :errors="errors"
        :show-help-text="showHelpText"
        :full-width-content="true"
    >
        <template #field>
            <input
                :id="id"
                :class="errorClasses"
                :placeholder="field.name"
                @change="handleChange"
                ref="input"
                type="hidden"
            />
        </template>
    </DefaultField>
</template>

<script>
// Fix cant resolve nova mixins package
import {FormField, HandlesValidationErrors} from '../../../vendor/laravel/nova/resources/js/mixins/packages.js'
import uuidv4 from '../uuidv4'

export default {
    mixins: [FormField, HandlesValidationErrors],
    props: ['resourceName', 'resourceId', 'field'],
    data: () => ({
        draftId: uuidv4()
    }),
    computed: {
        id() {
            return this.getId()
        }
    },
    methods: {
        /*
         * Set the initial, internal value for the field.
         */
        setInitialValue() {
            this.value = this.field.value || ''
        },
        getId() {
            return `${this.field.component}-${this.field.attribute}`
        },
        getSettings() {
            const settings = {}

            if (this.field.settings?.height) {
                settings.height = `${this.field.settings.height}px`
                delete this.field.settings.height
            }

            return {...settings, ...this.field.settings}
        },

        handleChange(event) {
            this.value = event.target.value
        },
        /**
         * Fill the given FormData object with the field's internal value.
         */
        fill(formData) {
            formData.append(this.field.attribute, this.value || '')
            formData.append(this.field.attribute + 'DraftId', this.draftId)
        },
        registerActions() {
            const { addAction, doAction } = Laraberg.wordpress.hooks
            addAction('blockEditor.beforeInit', 'nova-gutenberg', () => {
                doAction('novaGutenberg.beforeInit', this.getSettings(), this.resourceName, this.field)
            })

            addAction('blockEditor.afterInit', 'nova-gutenberg', () => {
                doAction('novaGutenberg.afterInit', this.getSettings(), this.resourceName, this.field)
            })
        }
    },
    mounted() {
        this.$refs.input.value = this.value
        this.registerActions()
        Laraberg.init(this.getId(), this.getSettings())

        /**
         * Temporary fix for Typography buttons submitting the resource
         */
        document.getElementById(this.field?.attribute).addEventListener('click', (e) => {
            const selector = '[data-wp-component="ToggleGroupControlOption"]'
            if (
                e.target.matches(selector)
                || e.target.parentNode.matches(selector)
            ) {
                e.preventDefault()
            }
        })

        /**
         * Fix for Typography inspector
         */
        document.getElementById(this.field?.attribute).addEventListener('click', (e) => {
            const selector = '[data-wp-component="ToggleGroupControlOptionBase"]'
            if (
                e.target.matches(selector)
                || e.target.parentNode.matches(selector)
            ) {
                    // If target is a button and dont have type of button add type button attribute
                    if(e.target.nodeName == "BUTTON" && e.target.type !== "button"){
                        e.target.setAttribute("type","button")
                        e.preventDefault()
                    }

                    // If target is a div inside button click a parent node
					if(e.target.nodeName === "DIV"){
						e.target.parentNode.click()
					}
                    
                    // If target is a button click self 
                    else if(e.target.nodeName === "BUTTON"){
                        e.target.click()
                    }
            }
        })

    },
    beforeDestroy() {
        Laraberg.removeEditor(this.$refs.input)
    }
}
</script>
