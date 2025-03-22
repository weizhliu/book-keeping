<script setup lang="ts">
import { ref, computed } from "vue";

const props = defineProps<{
  transactions: Array<{
    id: string;
    type: "Income" | "Expense";
    category: string;
    amount: number;
    date: string;
    note: string;
  }>;
}>();

const emit = defineEmits(["edit", "delete"]);

const searchTerm = ref("");
const filterType = ref("All");

const filteredTransactions = computed(() => {
  return props.transactions
    .filter((tx) => {
      if (filterType.value === "All") return true;
      return tx.type === filterType.value;
    })
    .filter((tx) => {
      if (!searchTerm.value) return true;
      const search = searchTerm.value.toLowerCase();
      return (
        tx.category.toLowerCase().includes(search) ||
        tx.note.toLowerCase().includes(search) ||
        tx.amount.toString().includes(search) ||
        tx.date.includes(search)
      );
    })
    .sort((a, b) => new Date(b.date).getTime() - new Date(a.date).getTime());
});

const formatCurrency = (amount: number) => {
  return new Intl.NumberFormat("en-US", {
    style: "currency",
    currency: "USD",
  }).format(amount);
};

function editTransaction(transaction: any) {
  emit("edit", transaction);
}

function deleteTransaction(id: any) {
  if (confirm("Are you sure you want to delete this transaction?")) {
    emit("delete", id);
  }
}
</script>

<template>
  <div class="space-y-6">
    <!-- Search and Filter -->
    <div class="space-y-3 border-b border-dotted border-amber-300 pb-4">
      <div class="relative">
        <input
          type="text"
          v-model="searchTerm"
          placeholder="Search transactions..."
          class="block w-full pl-10 pr-3 py-2 bg-amber-50 border border-amber-200 rounded-sm text-sm focus:outline-none focus:ring-2 focus:ring-amber-600"
        />
        <div
          class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            class="h-4 w-4 text-amber-700"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"
            />
          </svg>
        </div>
      </div>

      <div class="flex space-x-2">
        <button
          @click="filterType = 'All'"
          :class="[
            filterType === 'All'
              ? 'bg-amber-100 text-amber-900 border-amber-400 font-bold'
              : 'bg-amber-50 border-amber-200 text-amber-700 hover:text-amber-800',
            'px-3 py-1 rounded-sm text-sm font-medium flex-1 border transition-colors duration-200',
          ]"
        >
          <span class="relative">
            All
            <span
              v-if="filterType === 'All'"
              class="absolute inset-x-0 bottom-0 bg-amber-500 h-[6px] -z-10 opacity-40"
            ></span>
          </span>
        </button>
        <button
          @click="filterType = 'Income'"
          :class="[
            filterType === 'Income'
              ? 'bg-amber-100 text-amber-900 border-amber-400 font-bold'
              : 'bg-amber-50 border-amber-200 text-amber-700 hover:text-amber-800',
            'px-3 py-1 rounded-sm text-sm font-medium flex-1 border transition-colors duration-200',
          ]"
        >
          <span class="relative">
            Income
            <span
              v-if="filterType === 'Income'"
              class="absolute inset-x-0 bottom-0 bg-amber-500 h-[6px] -z-10 opacity-40"
            ></span>
          </span>
        </button>
        <button
          @click="filterType = 'Expense'"
          :class="[
            filterType === 'Expense'
              ? 'bg-amber-100 text-amber-900 border-amber-400 font-bold'
              : 'bg-amber-50 border-amber-200 text-amber-700 hover:text-amber-800',
            'px-3 py-1 rounded-sm text-sm font-medium flex-1 border transition-colors duration-200',
          ]"
        >
          <span class="relative">
            Expenses
            <span
              v-if="filterType === 'Expense'"
              class="absolute inset-x-0 bottom-0 bg-amber-500 h-[6px] -z-10 opacity-40"
            ></span>
          </span>
        </button>
      </div>
    </div>

    <!-- Transaction List -->
    <div>
      <h3 class="text-lg font-semibold mb-4 text-amber-800">
        All Transactions
      </h3>

      <div
        v-if="filteredTransactions.length === 0"
        class="text-amber-700 text-center py-6 border-b border-dotted border-amber-200"
      >
        No transactions found
      </div>

      <div v-else class="space-y-0">
        <div
          v-for="tx in filteredTransactions"
          :key="tx.id"
          class="py-4 border-b border-dotted border-amber-200"
        >
          <div class="flex justify-between items-center mb-2">
            <div class="flex items-center">
              <span
                :class="[
                  tx.type === 'Income'
                    ? 'bg-amber-100 text-amber-800 border-amber-300'
                    : 'bg-red-100 text-red-700 border-red-300',
                  'px-2 py-0.5 text-xs rounded-sm mr-2 border border-dotted',
                ]"
              >
                {{ tx.type }}
              </span>
              <span class="font-medium text-amber-900">{{ tx.category }}</span>
            </div>
            <span class="text-xs text-amber-700">{{
              new Date(tx.date).toLocaleDateString()
            }}</span>
          </div>

          <div class="flex justify-between items-center">
            <p class="text-sm text-amber-800 truncate max-w-[200px]">
              {{ tx.note || "No note" }}
            </p>
            <span
              :class="tx.type === 'Income' ? 'text-amber-800' : 'text-red-700'"
              class="font-semibold"
            >
              {{ tx.type === "Income" ? "+" : "-"
              }}{{ formatCurrency(tx.amount) }}
            </span>
          </div>

          <div class="flex justify-end space-x-3 mt-2 pt-2">
            <button
              @click="editTransaction(tx)"
              class="text-xs text-amber-700 hover:text-amber-800 font-medium underline transition-colors duration-200"
            >
              Edit
            </button>
            <button
              @click="deleteTransaction(tx.id)"
              class="text-xs text-red-600 hover:text-red-700 font-medium underline transition-colors duration-200"
            >
              Delete
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
