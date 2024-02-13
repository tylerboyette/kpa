<template>
  <div class="fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full" id="my-modal">
    <div class="relative top-20 mx-auto p-5 border w-96 shadow-lg rounded-md bg-white">
      <div class="flex items-center justify-between">
        <h2 class="text-lg font-bold text-center flex-1">{{ guest ? 'Edit Guest' : 'Add New Guest' }}</h2>
        <button @click="handleCancel" class="p-2 hover:bg-gray-200">
          <span class="text-2xl">&times;</span>
        </button>
      </div>
      <form @submit.prevent="handleSave" class="bg-white rounded px-8 pt-6 pb-8 mb-4">
        <div class="mb-4">
          <label class="block text-gray-700 text-sm font-bold mb-2" for="email">
            Email
          </label>
          <input class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" id="email" type="email" v-model="guestData.email" required placeholder="Email" />
        </div>
        <div class="mb-6">
          <label class="block text-gray-700 text-sm font-bold mb-2" for="tickets">
            Tickets
          </label>
          <input class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 mb-3 leading-tight focus:outline-none focus:shadow-outline" id="tickets" type="number" v-model.number="guestData.tickets" required min="1" placeholder="Tickets" />  
        </div>
        <div class="flex items-center justify-between">
          <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline" type="submit">
            Save
          </button>
          <button class="inline-block align-baseline font-bold text-sm text-blue-500 hover:text-blue-800" @click="handleCancel">
            Cancel
          </button>
        </div>
      </form>
    </div>
  </div>
</template> 

<script lang="ts" setup>
import { ref, watch, defineProps, defineEmits } from 'vue';
import { GuestType } from '../services/guest-repository';

const props = defineProps({
  guest: Object as () => GuestType | null
});

const emits = defineEmits(['saved', 'cancelled']);

const guestData = ref<GuestType>(props.guest || { email: '', tickets: 1 });

watch(() => props.guest, (newGuest) => {
  if (newGuest) {
    guestData.value = { ...newGuest };
  }
}, { deep: true });

const handleSave = () => {
  emits('saved', guestData.value);
};

const handleCancel = () => {
  emits('cancelled');
};
</script>
