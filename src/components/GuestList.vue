<template>
  <div class="container mx-auto my-6 p-4 bg-white shadow rounded-lg">
    <h2 class="text-2xl font-bold mb-4">Guest List for Nyan Cat Performance</h2>
    <table class="min-w-full leading-normal">
      <thead>
        <tr>
          <th class="px-5 py-3 border-b-2 border-gray-200 bg-gray-100 text-left text-xs font-semibold text-gray-600 uppercase tracking-wider">
            Email
          </th>
          <th class="px-5 py-3 border-b-2 border-gray-200 bg-gray-100 text-left text-xs font-semibold text-gray-600 uppercase tracking-wider">
            Tickets
          </th>
          <th class="px-5 py-3 border-b-2 border-gray-200 bg-gray-100 text-left text-xs font-semibold text-gray-600 uppercase tracking-wider">
            Actions
          </th>
        </tr>
      </thead>  
      <tbody>
        <tr v-if="loadingGuests">
          <td colspan="3" class="text-center p-8">
            <Spinner />
          </td>
        </tr>
        <tr v-else-if="guests.length === 0">
          <td colspan="3" class="text-center p-8">
            No guests have been added yet.
          </td>
        </tr>
        <tr v-for="guest in guests" :key="guest.email" class="hover:bg-gray-100">
          <td class="px-5 py-5 border-b border-gray-200 bg-white text-sm">
            {{ guest.email }}
          </td>
          <td class="px-5 py-5 border-b border-gray-200 bg-white text-sm">
            {{ guest.tickets }}
          </td>
          <td class="px-5 py-5 border-b border-gray-200 bg-white text-sm">
            <button @click="editGuest(guest)" class="text-indigo-600 hover:text-indigo-900">
              Edit
            </button>
            <button @click="confirmDeleteGuest(guest)" class="text-red-600 hover:text-red-900 ml-4">
              Delete
            </button>
          </td>
        </tr>
      </tbody>
    </table>
    <button 
      @click="addGuest"
      :disabled="!canAddGuests"
      class="mt-4 bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded disabled:bg-blue-300"
    >
      Add Guest
    </button>
  </div>
  <GuestForm v-if="editingGuest" :guest="selectedGuest" @saved="saveGuest" @cancelled="editingGuest = false" />
  <div v-if="savingGuest || deletingGuest" class="fixed inset-0 bg-gray-600 bg-opacity-50 flex justify-center items-center">
    <Spinner />
  </div>
</template>

<script lang="ts" setup>
import { ref, onMounted, computed } from 'vue';
import Spinner from './Spinner.vue';
import GuestForm from './GuestForm.vue';
import GuestRepository, { GuestType } from '../services/guest-repository';

const guestRepository = new GuestRepository();
const guests = ref<GuestType[]>([]);
const selectedGuest = ref<GuestType | null>(null);
const editingGuest = ref(false);
const loadingGuests = ref(true);
const savingGuest = ref(false);
const deletingGuest = ref(false);

const maxGuests = 20; // Limitation based on the count of guests

const guestCount = computed(() => guests.value.length); // Compute the current guest count
const canAddGuests = computed(() => guestCount.value < maxGuests); // Check if adding more guests is possible

onMounted(async () => {
  try {
    guests.value = await guestRepository.load();
  } catch (error) {
    console.error("Failed to load guests:", error);
  } finally {
    loadingGuests.value = false;
  }
});

const addGuest = () => {
  if (canAddGuests.value) {
    selectedGuest.value = { email: '', tickets: 1 };
    editingGuest.value = true;
  } else {
    alert('Maximum number of guests reached. Cannot add more guests.');
  }
};

const editGuest = (guest: GuestType) => {
  selectedGuest.value = { ...guest };
  editingGuest.value = true;
};

const confirmDeleteGuest = async (guest: GuestType) => {
  deletingGuest.value = true;
  try {
    guests.value = guests.value.filter(g => g.email !== guest.email);
    await guestRepository.save(guests.value);
  } catch (error) {
    console.error("Failed to delete guest:", error);
  } finally {
    deletingGuest.value = false;
  }
};

const saveGuest = async (guest: GuestType) => {
  savingGuest.value = true;
  try {
    const existingGuestIndex = guests.value.findIndex(g => g.email === guest.email);
    if (existingGuestIndex !== -1) {
      guests.value[existingGuestIndex] = guest;
    } else {
      if (guestCount.value >= maxGuests) {
        alert('Adding this guest would exceed the maximum number of guests allowed.');
        return;
      }
      guests.value.push(guest);
    }
    await guestRepository.save(guests.value);
  } catch (error) {
    console.error("Failed to save guest:", error);
  } finally {
    savingGuest.value = false;
    editingGuest.value = false;
  }
};
</script>
