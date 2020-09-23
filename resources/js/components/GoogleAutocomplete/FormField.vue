<template>
    <default-field :field="this.field">
        <template slot="field">
            <div class="form-group">
                <vue-google-autocomplete
                        ref="address"
                        :id="this.field.name"
                        class="custom-input w-full"
                        :class="errorClasses"
                        :placeholder="placeholder"
                        :country="this.field.countries"
                        :types="this.field.type"
                        v-model="value"
                        :disabled="isReadonly"
                        v-on:keypress.enter.prevent=""
                        v-on:placechanged="getAddressData">
                </vue-google-autocomplete>
            </div>

            <p v-if="value != ''" class="my-2"><span class="lowercase">{{ translate.current_address }}</span>: {{ value }}</p>

            <p v-if="hasError" class="help-text error-text mt-2 text-danger">
                {{ firstError }}
            </p>
        </template>
    </default-field>
</template>

<script>
import { FormField, HandlesValidationErrors } from 'laravel-nova'
import VueGoogleAutocomplete from 'vue-google-autocomplete'

export default {
    components: { VueGoogleAutocomplete },

    mixins: [FormField, HandlesValidationErrors],

    props: ['resourceName', 'resourceId', 'field'],

    data: function () {
        return {
            address: ''
        }
    },

    computed: {
        translate() {
            return Nova.config.google_autocomplete_translations
        },

        placeholder() {
            if (this.value != '') {
                return this.translate.update_address
            }

            return this.field.name
        }
    },

    methods: {

        /**
         * Get address
         */
        getAddressData: function (addressData, placeResultData) {
            // Save current data address as a string
            this.handleChange(placeResultData.formatted_address)

            // Emmit events to by catch up for the other AddressMetadata fields
            this.field.addressObject.forEach(element => {
                let value = '';
                // Get value from addressData
                if (addressData.hasOwnProperty(element)) {
                    value = addressData[element];
                }
                // Get value from placeResultData
                let res = _.find(placeResultData.address_components, function (addressComponent) {
                  return _.find(addressComponent.types, function (componentType) {
                    return componentType == element;
                  }) !== undefined;
                });
                if (res !== undefined) {
                  value = res.long_name;
                }
                // Propagate value to other address fields
                Nova.$emit('address-metadata-update-' + element, {
                    value: value
                })

            });
        },

        /*
         * Set the initial, internal value for the field.
         */
        setInitialValue() {
          this.value = this.field.value || ''
        },

        /**
         * Fill the given FormData object with the field's internal value.
         */
        fill(formData) {
          formData.append(this.field.attribute, this.value || '')
        },

        /**
         * Update the field's internal value.
         */
        handleChange(value) {
          this.value = value
        }
    }
}
</script>
