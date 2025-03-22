<script setup lang="ts">
import { ref, onMounted, watch } from "vue";
import Dashboard from "./components/Dashboard.vue";
import Calendar from "./components/Calendar.vue";
import TransactionForm from "./components/TransactionForm.vue";
import TransactionList from "./components/TransactionList.vue";

// Define transaction type
interface Transaction {
  id: string;
  type: "Income" | "Expense";
  category: string;
  amount: number;
  date: string;
  note: string;
}

// State management
const transactions = ref<Transaction[]>([]);
const isFormVisible = ref(false);
const selectedDate = ref("");
const editTransaction = ref<Transaction | undefined>(undefined);
const activeTab = ref("dashboard"); // dashboard, calendar, list
const calendarRef = ref<any>(null);

// Load transactions from local storage on mount
onMounted(() => {
  const savedTransactions = localStorage.getItem("transactions");
  if (savedTransactions) {
    transactions.value = JSON.parse(savedTransactions);
  }

  // Add calendar events
  if (calendarRef.value) {
    calendarRef.value.addEvents(transactions.value);
  }
});

// Save transactions to local storage whenever they change
watch(
  transactions,
  (newTransactions) => {
    localStorage.setItem("transactions", JSON.stringify(newTransactions));

    // Update calendar events
    if (calendarRef.value) {
      calendarRef.value.addEvents(transactions.value);
    }
  },
  { deep: true },
);

// Update calendar when showing the calendar tab
watch(activeTab, (newTab) => {
  if (newTab === "calendar" && calendarRef.value) {
    // Use a small delay to ensure the tab is fully rendered
    setTimeout(() => {
      calendarRef.value.addEvents([...transactions.value]);
    }, 50);
  }
});

// Calendar event handlers
function handleCalendarDateClick(arg: any) {
  selectedDate.value = arg.dateStr;
  isFormVisible.value = true;
}

function handleCalendarEventClick(arg: any) {
  const transactionId = arg.event.id;
  const transaction = transactions.value.find((tx) => tx.id === transactionId);
  if (transaction) {
    editTransaction.value = { ...transaction };
    isFormVisible.value = true;
  }
}

function handleCalendarSelect(arg: any) {
  selectedDate.value = arg.startStr;
  isFormVisible.value = true;
}

// Transaction CRUD operations
function saveTransaction(transaction: Transaction) {
  const index = transactions.value.findIndex((tx) => tx.id === transaction.id);

  if (index !== -1) {
    // Update existing transaction
    transactions.value[index] = transaction;
  } else {
    // Add new transaction
    transactions.value.push(transaction);
  }

  closeForm();
}

function deleteTransaction(id: string) {
  transactions.value = transactions.value.filter((tx) => tx.id !== id);
}

function editTransactionItem(transaction: Transaction) {
  editTransaction.value = { ...transaction };
  isFormVisible.value = true;
}

function closeForm() {
  isFormVisible.value = false;
  editTransaction.value = undefined;
  selectedDate.value = "";
}
</script>

<template>
  <div class="min-h-screen bg-amber-50 text-amber-900">
    <main class="max-w-7xl mx-auto px-3 pt-3 pb-24">
      <!-- Main Content with scrollable sections -->
      <div class="mt-2 overflow-hidden h-[calc(100vh-7rem)]">
        <!-- Main Content Container -->
        <div v-if="activeTab === 'dashboard'" class="h-full">
          <Dashboard
            :transactions="transactions"
            class="mb-2 bg-white p-3 rounded-sm border border-amber-200 shadow-sm h-full overflow-y-auto custom-scrollbar"
          />
        </div>

        <div v-if="activeTab === 'calendar'" class="h-full">
          <Calendar
            ref="calendarRef"
            @date-click="handleCalendarDateClick"
            @event-click="handleCalendarEventClick"
            @select="handleCalendarSelect"
            class="mb-2 bg-white p-3 rounded-sm border border-amber-200 shadow-sm h-full overflow-y-auto custom-scrollbar"
          />
        </div>

        <div v-if="activeTab === 'list'" class="h-full">
          <TransactionList
            :transactions="transactions"
            @edit="editTransactionItem"
            @delete="deleteTransaction"
            class="mb-2 bg-white p-3 rounded-sm border border-amber-200 shadow-sm h-full overflow-y-auto custom-scrollbar"
          />
        </div>
      </div>

      <!-- Transaction Form Modal -->
      <div
        v-if="isFormVisible"
        class="fixed inset-0 backdrop-blur-sm bg-amber-900/30 flex items-center justify-center z-50"
        @click="closeForm"
      >
        <div class="max-w-md w-full mx-4" @click.stop>
          <TransactionForm
            :date="selectedDate"
            :edit-transaction="editTransaction"
            @save="saveTransaction"
            @cancel="closeForm"
          />
        </div>
      </div>

      <!-- Bottom Nav Bar -->
      <div
        class="fixed bottom-0 left-0 right-0 bg-amber-50 border-t border-amber-200 shadow-lg z-10 safe-area-inset-bottom"
      >
        <div class="flex justify-around py-1">
          <button
            @click="activeTab = 'dashboard'"
            class="flex flex-col items-center py-2 px-3 flex-1 transition-all duration-200"
            :class="
              activeTab === 'dashboard'
                ? 'text-amber-900 font-bold scale-110'
                : 'text-amber-700 hover:text-amber-800 hover:scale-105'
            "
          >
            <!-- Hand-drawn Dashboard icon -->
            <svg
              class="h-6 w-6 transform transition-transform duration-300"
              :class="{ 'scale-110 rotate-2': activeTab === 'dashboard' }"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              style="filter: url(#hand-drawn)"
            >
              <!-- Main dashboard frame -->
              <path
                d="M5,5 C4.5,5 4,5.5 4,6 L4,18 C4,18.5 4.5,19 5,19 L19,19 C19.5,19 20,18.5 20,18 L20,6 C20,5.5 19.5,5 19,5 Z"
                stroke-width="1.5"
                fill="none"
                class="transition-all duration-300"
                :style="{
                  'stroke-opacity': activeTab === 'dashboard' ? '1' : '0.8',
                }"
              />

              <!-- Grid layout for dashboard -->
              <path
                d="M4,9 L20,9"
                stroke-width="1"
                class="transition-all duration-300"
                :style="{
                  'stroke-opacity': activeTab === 'dashboard' ? '0.8' : '0.5',
                }"
              />
              <path
                d="M4,14 L20,14"
                stroke-width="1"
                class="transition-all duration-300"
                :style="{
                  'stroke-opacity': activeTab === 'dashboard' ? '0.8' : '0.5',
                }"
              />
              <path
                d="M12,9 L12,19"
                stroke-width="1"
                class="transition-all duration-300"
                :style="{
                  'stroke-opacity': activeTab === 'dashboard' ? '0.8' : '0.5',
                }"
              />

              <!-- Chart Elements -->
              <!-- Bar chart -->
              <path
                d="M6,17 L6,15"
                stroke-width="2"
                stroke-linecap="round"
                :class="{ 'animate-grow': activeTab === 'dashboard' }"
              />
              <path
                d="M8,17 L8,12"
                stroke-width="2"
                stroke-linecap="round"
                :class="{ 'animate-grow': activeTab === 'dashboard' }"
              />
              <path
                d="M10,17 L10,13.5"
                stroke-width="2"
                stroke-linecap="round"
                :class="{ 'animate-grow': activeTab === 'dashboard' }"
              />

              <!-- Line graph -->
              <path
                d="M14,13 C14.5,12 15,12.5 15.5,11.5 C16,10.5 16.5,12 17,11 C17.5,10 18,12"
                stroke-width="1.5"
                stroke-linecap="round"
                :class="{ 'animate-draw': activeTab === 'dashboard' }"
              />

              <!-- Small title blocks -->
              <rect
                x="6"
                y="6.5"
                width="4"
                height="1"
                rx="0.5"
                :fill="activeTab === 'dashboard' ? 'currentColor' : 'none'"
                stroke="currentColor"
                stroke-width="0.5"
              />
              <rect
                x="14"
                y="6.5"
                width="4"
                height="1"
                rx="0.5"
                :fill="activeTab === 'dashboard' ? 'currentColor' : 'none'"
                stroke="currentColor"
                stroke-width="0.5"
              />
            </svg>
            <span
              class="text-[0.65rem] mt-1 transition-all duration-200"
              :class="activeTab === 'dashboard' ? 'relative font-medium' : ''"
            >
              Dashboard
              <span
                v-if="activeTab === 'dashboard'"
                class="absolute inset-x-0 bottom-0 bg-amber-500 h-[3px] -z-10 opacity-30"
                style="transform: rotate(-1deg)"
              ></span>
            </span>
          </button>

          <button
            @click="activeTab = 'calendar'"
            class="flex flex-col items-center py-2 px-3 flex-1 transition-all duration-200"
            :class="
              activeTab === 'calendar'
                ? 'text-amber-900 font-bold scale-110'
                : 'text-amber-700 hover:text-amber-800 hover:scale-105'
            "
          >
            <!-- Hand-drawn Calendar icon -->
            <svg
              class="h-6 w-6 transform transition-transform duration-300"
              :class="{ 'scale-110 rotate-[2deg]': activeTab === 'calendar' }"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              style="filter: url(#hand-drawn)"
            >
              <!-- Calendar outer frame -->
              <path
                d="M5,5 C4.5,5 4,5.5 4,6 L4,18 C4,18.5 4.5,19 5,19 L19,19 C19.5,19 20,18.5 20,18 L20,6 C20,5.5 19.5,5 19,5 Z"
                stroke-width="1.5"
                fill="none"
                class="transition-all duration-300"
                :style="{
                  'stroke-opacity': activeTab === 'calendar' ? '1' : '0.8',
                }"
              />

              <!-- Calendar header bar -->
              <rect
                x="4"
                y="8"
                width="16"
                height="0.5"
                stroke="currentColor"
                stroke-width="0.5"
              />

              <!-- Calendar day labels (S M T W T F S) -->
              <text
                x="5.2"
                y="7"
                font-size="2"
                fill="currentColor"
                style="font-weight: 500"
              >
                S
              </text>
              <text
                x="7.7"
                y="7"
                font-size="2"
                fill="currentColor"
                style="font-weight: 500"
              >
                M
              </text>
              <text
                x="10.2"
                y="7"
                font-size="2"
                fill="currentColor"
                style="font-weight: 500"
              >
                T
              </text>
              <text
                x="12.7"
                y="7"
                font-size="2"
                fill="currentColor"
                style="font-weight: 500"
              >
                W
              </text>
              <text
                x="15.2"
                y="7"
                font-size="2"
                fill="currentColor"
                style="font-weight: 500"
              >
                T
              </text>
              <text
                x="17.7"
                y="7"
                font-size="2"
                fill="currentColor"
                style="font-weight: 500"
              >
                F
              </text>
              <text
                x="20.2"
                y="7"
                font-size="2"
                fill="currentColor"
                style="font-weight: 500"
              >
                S
              </text>

              <!-- Calendar grid -->
              <path
                d="M7,8 L7,19"
                stroke-width="0.5"
                :style="{
                  'stroke-opacity': activeTab === 'calendar' ? '0.5' : '0.3',
                }"
              />
              <path
                d="M11,8 L11,19"
                stroke-width="0.5"
                :style="{
                  'stroke-opacity': activeTab === 'calendar' ? '0.5' : '0.3',
                }"
              />
              <path
                d="M15,8 L15,19"
                stroke-width="0.5"
                :style="{
                  'stroke-opacity': activeTab === 'calendar' ? '0.5' : '0.3',
                }"
              />
              <path
                d="M19,8 L19,19"
                stroke-width="0.5"
                :style="{
                  'stroke-opacity': activeTab === 'calendar' ? '0.5' : '0.3',
                }"
              />
              <path
                d="M4,11 L20,11"
                stroke-width="0.5"
                :style="{
                  'stroke-opacity': activeTab === 'calendar' ? '0.5' : '0.3',
                }"
              />
              <path
                d="M4,15 L20,15"
                stroke-width="0.5"
                :style="{
                  'stroke-opacity': activeTab === 'calendar' ? '0.5' : '0.3',
                }"
              />

              <!-- Highlighted current day -->
              <rect
                x="8"
                y="12"
                width="2"
                height="2"
                rx="0.5"
                :fill="activeTab === 'calendar' ? 'currentColor' : 'none'"
                stroke="currentColor"
                stroke-width="0.75"
                :style="{
                  'stroke-opacity': activeTab === 'calendar' ? '1' : '0.5',
                }"
                :class="{ 'animate-wiggle': activeTab === 'calendar' }"
              />
            </svg>
            <span
              class="text-[0.65rem] mt-1 transition-all duration-200"
              :class="activeTab === 'calendar' ? 'relative font-medium' : ''"
            >
              Calendar
              <span
                v-if="activeTab === 'calendar'"
                class="absolute inset-x-0 bottom-0 bg-amber-500 h-[3px] -z-10 opacity-30"
                style="transform: rotate(-1deg)"
              ></span>
            </span>
          </button>

          <button
            @click="activeTab = 'list'"
            class="flex flex-col items-center py-2 px-3 flex-1 transition-all duration-200"
            :class="
              activeTab === 'list'
                ? 'text-amber-900 font-bold scale-110'
                : 'text-amber-700 hover:text-amber-800 hover:scale-105'
            "
          >
            <!-- Hand-drawn Transactions list icon -->
            <svg
              class="h-6 w-6 transform transition-transform duration-300"
              :class="{ 'scale-110 rotate-[-2deg]': activeTab === 'list' }"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              style="filter: url(#hand-drawn)"
            >
              <!-- Main receipt/transaction list frame -->
              <path
                d="M5,5 C4.5,5 4,5.5 4,6 L4,18 C4,18.5 4.5,19 5,19 L19,19 C19.5,19 20,18.5 20,18 L20,6 C20,5.5 19.5,5 19,5 Z"
                stroke-width="1.5"
                fill="none"
                class="transition-all duration-300"
                :style="{
                  'stroke-opacity': activeTab === 'list' ? '1' : '0.8',
                }"
              />

              <!-- Receipt header -->
              <rect
                x="6"
                y="6.5"
                width="12"
                height="1.5"
                rx="0.5"
                :fill="activeTab === 'list' ? 'currentColor' : 'none'"
                stroke="currentColor"
                stroke-width="0.5"
              />

              <!-- Transaction items with amounts -->
              <line
                x1="6"
                y1="10"
                x2="12"
                y2="10"
                stroke-width="1.5"
                stroke-linecap="round"
                :class="{ 'animate-transactionSlide': activeTab === 'list' }"
              />
              <line
                x1="14"
                y1="10"
                x2="18"
                y2="10"
                stroke-width="1.5"
                stroke-linecap="round"
                :class="{ 'animate-transactionSlide': activeTab === 'list' }"
              />

              <line
                x1="6"
                y1="13"
                x2="10"
                y2="13"
                stroke-width="1.5"
                stroke-linecap="round"
                :class="{
                  'animate-transactionSlide': activeTab === 'list',
                  'animation-delay-100': true,
                }"
              />
              <line
                x1="14"
                y1="13"
                x2="16"
                y2="13"
                stroke-width="1.5"
                stroke-linecap="round"
                :class="{
                  'animate-transactionSlide': activeTab === 'list',
                  'animation-delay-100': true,
                }"
              />

              <line
                x1="6"
                y1="16"
                x2="11"
                y2="16"
                stroke-width="1.5"
                stroke-linecap="round"
                :class="{
                  'animate-transactionSlide': activeTab === 'list',
                  'animation-delay-200': true,
                }"
              />
              <line
                x1="14"
                y1="16"
                x2="17"
                y2="16"
                stroke-width="1.5"
                stroke-linecap="round"
                :class="{
                  'animate-transactionSlide': activeTab === 'list',
                  'animation-delay-200': true,
                }"
              />
            </svg>
            <span
              class="text-[0.65rem] mt-1 transition-all duration-200"
              :class="activeTab === 'list' ? 'relative font-medium' : ''"
            >
              Transactions
              <span
                v-if="activeTab === 'list'"
                class="absolute inset-x-0 bottom-0 bg-amber-500 h-[3px] -z-10 opacity-30"
                style="transform: rotate(1deg)"
              ></span>
            </span>
          </button>

          <button
            @click="isFormVisible = true"
            class="flex flex-col items-center py-2 px-3 flex-1 transition-all duration-200 text-amber-700 hover:text-amber-800 hover:scale-105 active:scale-95"
          >
            <!-- Hand-drawn New transaction icon -->
            <svg
              class="h-6 w-6 transform transition-all duration-300 hover:rotate-[15deg]"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              style="filter: url(#hand-drawn)"
            >
              <!-- Transaction slip/receipt -->
              <path
                d="M7,4 C6.5,4 6,4.5 6,5 L6,19 C6,19.5 6.5,20 7,20 L17,20 C17.5,20 18,19.5 18,19 L18,5 C18,4.5 17.5,4 17,4 Z"
                stroke-width="1.5"
                fill="none"
                class="transition-all duration-300 hover:stroke-amber-900"
              />

              <!-- Dollar sign -->
              <path
                d="M12,8 C11,8 10,8.5 10,9.5 C10,10.5 11,11 12,11 C13,11 14,11.5 14,12.5 C14,13.5 13,14 12,14 C11,14 10,13.5 10,12.5"
                stroke-width="1.5"
                stroke-linecap="round"
                class="transition-all duration-300 hover:stroke-amber-900"
              />
              <line
                x1="12"
                y1="7"
                x2="12"
                y2="15"
                stroke-width="1.5"
                stroke-linecap="round"
                class="transition-all duration-300 hover:stroke-amber-900"
              />

              <!-- Plus symbol -->
              <circle
                cx="18"
                cy="6"
                r="3"
                stroke-width="1.25"
                class="transition-all duration-300"
                :fill="'rgba(255, 213, 128, 0.3)'"
              />
              <path
                d="M18,4.5 L18,7.5"
                stroke-width="1.25"
                stroke-linecap="round"
                class="transition-all duration-200 hover:stroke-width-1.5"
              />
              <path
                d="M16.5,6 L19.5,6"
                stroke-width="1.25"
                stroke-linecap="round"
                class="transition-all duration-200 hover:stroke-width-1.5"
              />
            </svg>
            <span class="text-[0.65rem] mt-1 font-medium">New</span>
          </button>
        </div>
      </div>

      <!-- SVG Filters for hand-drawn effect -->
      <svg width="0" height="0" style="position: absolute">
        <filter id="hand-drawn">
          <feTurbulence
            type="fractalNoise"
            baseFrequency="0.01"
            numOctaves="3"
            result="noise"
          />
          <feDisplacementMap
            in="SourceGraphic"
            in2="noise"
            scale="1"
            xChannelSelector="R"
            yChannelSelector="G"
          />
        </filter>
      </svg>
    </main>
  </div>
</template>

<style>
/* Calendar styles via CDN will be loaded in index.html instead */

body {
  margin: 0;
  font-family:
    -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue",
    Arial, sans-serif;
}

@keyframes dash {
  to {
    stroke-dasharray: 0;
  }
}

@keyframes wiggle {
  0% {
    transform: translateX(0);
  }
  25% {
    transform: translateX(1px);
  }
  75% {
    transform: translateX(-1px);
  }
  100% {
    transform: translateX(0);
  }
}

@keyframes grow {
  0% {
    transform: scaleY(0);
  }
  100% {
    transform: scaleY(1);
  }
}

@keyframes draw {
  0% {
    stroke-dashoffset: 100;
  }
  100% {
    stroke-dashoffset: 0;
  }
}

@keyframes transactionSlide {
  0% {
    transform: translateX(-20px);
    opacity: 0;
  }
  100% {
    transform: translateX(0);
    opacity: 1;
  }
}

.animate-wiggle {
  animation: wiggle 0.5s ease-in-out;
}

.animate-grow {
  transform-origin: bottom;
  animation: grow 0.7s ease-out forwards;
}

.animate-draw {
  stroke-dasharray: 100;
  stroke-dashoffset: 100;
  animation: draw 1.2s ease-in-out forwards;
}

.animate-transactionSlide {
  opacity: 0;
  animation: transactionSlide 0.5s ease-out forwards;
}

.animation-delay-100 {
  animation-delay: 0.1s;
}

.animation-delay-200 {
  animation-delay: 0.2s;
}

/* Custom scrollbar styles */
.custom-scrollbar::-webkit-scrollbar {
  width: 6px;
}

.custom-scrollbar::-webkit-scrollbar-track {
  background: rgba(251, 233, 211, 0.5);
  border-radius: 10px;
}

.custom-scrollbar::-webkit-scrollbar-thumb {
  background: rgba(217, 119, 6, 0.3);
  border-radius: 10px;
  transition: background 0.2s ease;
}

.custom-scrollbar::-webkit-scrollbar-thumb:hover {
  background: rgba(217, 119, 6, 0.5);
}

/* Firefox scrollbar */
.custom-scrollbar {
  scrollbar-width: thin;
  scrollbar-color: rgba(217, 119, 6, 0.3) rgba(251, 233, 211, 0.5);
}
</style>
