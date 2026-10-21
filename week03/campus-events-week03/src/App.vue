<script setup lang="ts">
import { computed, ref, watch } from "vue";
import EventList from "./components/EventList.vue";
import { initialEvents } from "./data/events";
import type { CampusEvent } from "./types";

const STORAGE_KEY = "myBookings";

// A reactive copy of the hard-coded events
const events = ref<CampusEvent[]>(initialEvents.map(e => ({ ...e })));
// TODO 4 (bonus, part1): keep my bookings after a page reload
//   - watch myBookings and save it to localStorage (JSON.stringify)
//   - read it back when the app starts, and add those seats to `booked`
function loadBookings(): number[] {
  try {
    const saved = JSON.parse(localStorage.getItem(STORAGE_KEY) ?? "[]");
    return Array.isArray(saved) ? saved : [];
  } catch {
    return [];
  }
}

// TODO 1: a ref `myBookings` holding the ids of the events I booked (one entry per seat)
const myBookings = ref<number[]>(loadBookings());
for (const id of myBookings.value) {
  const ev = events.value.find(e => e.id === id);
  if (ev) ev.booked++;
}


// TODO 2: a computed `bookingCount`
const bookingCount = computed(() => myBookings.value.length);

// TODO 3: onBook(id): find the event; if it still has a seat,
//         increase its `booked` and add the id to myBookings
function onBook(id: number) {
  const ev = events.value.find(e => e.id === id);
  if (!ev || ev.booked >= ev.capacity) return; // not found, or full
  ev.booked++;
  myBookings.value.push(id);
}
watch(
  myBookings,
  list => localStorage.setItem(STORAGE_KEY, JSON.stringify(list)),
  { deep: true }
);


</script>

<template>
  <main>
    <header>
      <h1>Campus Events</h1>
      <!-- TODO 5: show the real count ("1 seat" / "N seats") -->
      <p class="mine">You have booked 0 seats.</p>
      <p class="mine">You have booked {{ bookingCount }} {{ bookingCount === 1 ? "seat" : "seats" }}.</p>
    </header>
    <EventList :events="events" @book="onBook" />
  </main>
</template>

<style scoped>
header { margin-bottom: 16px; }
.mine { color: #6b7280; margin: 0; }
</style>
