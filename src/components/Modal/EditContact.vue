<script setup lang="ts">
import { reactive, computed, watch } from 'vue';
import { clone } from '@/helpers/utils';
import { useContactsStore } from '@/stores/contacts';
import { validateForm } from '@/helpers/validation';

const DEFAULT_FORM_STATE = {
  name: '',
  address: ''
};

const props = defineProps<{
  open: boolean;
  initialState?: any;
}>();

const emit = defineEmits<{
  (e: 'close');
}>();

const contactsStore = useContactsStore();

const definition = {
  type: 'object',
  title: 'Contact',
  additionalProperties: false,
  required: ['name', 'address'],
  properties: {
    name: {
      type: 'string',
      title: 'Name',
      minLength: 1
    },
    address: {
      type: 'string',
      title: 'Address',
      examples: ['Address or ENS']
    }
  }
};

const form: {
  name: string;
  address: string;
} = reactive(clone(DEFAULT_FORM_STATE));

const addressDuplicated = computed(() => {
  if (form.address === props.initialState?.address) return false;
  return !!contactsStore.contacts.find(c => c.address === form.address);
});

const formErrors = computed(() => {
  const errors = validateForm(definition, form);
  if (addressDuplicated.value) errors.address = 'Address duplicated';
  return errors;
});

function handleSubmit() {
  contactsStore.saveContact(clone(form));
  emit('close');
}

watch(
  () => props.open,
  () => {
    if (props.initialState) {
      form.name = props.initialState.name;
      form.address = props.initialState.address;
    } else {
      form.name = DEFAULT_FORM_STATE.name;
      form.address = DEFAULT_FORM_STATE.address;
    }
  }
);
</script>

<template>
  <UiModal :open="open" @close="$emit('close')">
    <template #header>
      <h3 v-text="initialState?.address ? 'Edit contact' : 'Add contact'" />
    </template>
    <div class="s-box p-4">
      <SIObject :model-value="form" :error="formErrors" :definition="definition" />
    </div>
    <template #footer>
      <UiButton class="w-full" :disabled="Object.keys(formErrors).length > 0" @click="handleSubmit">
        Confirm
      </UiButton>
    </template>
  </UiModal>
</template>
