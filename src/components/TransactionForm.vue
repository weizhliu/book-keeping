<script setup lang="ts">
import { ref, defineEmits } from "vue";

const props = defineProps<{
  date?: string;
  editTransaction?: {
    id: string;
    type: "Income" | "Expense";
    category: string;
    amount: number;
    date: string;
    note: string;
  };
}>();

const emit = defineEmits(["save", "cancel"]);

const id = ref(props.editTransaction?.id || crypto.randomUUID());
const type = ref(props.editTransaction?.type || "Expense");
const category = ref(props.editTransaction?.category || "");
const amount = ref(props.editTransaction?.amount || 0);
const date = ref(
  props.editTransaction?.date ||
    props.date ||
    new Date().toISOString().split("T")[0],
);
const note = ref(props.editTransaction?.note || "");

const categoryOptions = {
  Income: ["Salary", "Investment", "Gift", "Other"],
  Expense: [
    "Food",
    "Transportation",
    "Entertainment",
    "Utilities",
    "Shopping",
    "Housing",
    "Health",
    "Education",
    "Other",
  ],
};

function saveTransaction() {
  if (amount.value <= 0 || !category.value) {
    return;
  }

  emit("save", {
    id: id.value,
    type: type.value,
    category: category.value,
    amount: amount.value,
    date: date.value,
    note: note.value,
  });
}

function cancel() {
  emit("cancel");
}
</script>

<template>
  <div
    class="bg-amber-50/95 p-5 rounded-sm shadow-lg border border-amber-200 backdrop-filter"
  >
    <div class="flex items-center justify-between mb-5">
      <h2 class="text-lg font-semibold text-amber-900">
        {{ editTransaction ? "Edit" : "New" }} Transaction
      </h2>
      <button
        type="button"
        @click="cancel"
        class="text-amber-600 hover:text-amber-800"
      >
        <svg
          xmlns="http://www.w3.org/2000/svg"
          class="h-5 w-5"
          viewBox="0 0 20 20"
          fill="currentColor"
        >
          <path
            fill-rule="evenodd"
            d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z"
            clip-rule="evenodd"
          />
        </svg>
      </button>
    </div>

    <form @submit.prevent="saveTransaction" class="space-y-4">
      <!-- Transaction Type Tabs -->
      <div class="flex rounded-sm overflow-hidden border border-amber-300">
        <button
          type="button"
          @click="type = 'Expense'"
          :class="[
            type === 'Expense'
              ? 'bg-amber-100 text-amber-900 font-bold'
              : 'bg-amber-50 text-amber-700 hover:text-amber-800',
            'flex-1 py-2 text-sm font-medium focus:outline-none transition-colors duration-200',
          ]"
        >
          <span class="relative">
            Expense
            <span
              v-if="type === 'Expense'"
              class="absolute inset-x-0 bottom-0 bg-amber-500 h-[6px] -z-10 opacity-40"
            ></span>
          </span>
        </button>
        <button
          type="button"
          @click="type = 'Income'"
          :class="[
            type === 'Income'
              ? 'bg-amber-100 text-amber-900 font-bold'
              : 'bg-amber-50 text-amber-700 hover:text-amber-800',
            'flex-1 py-2 text-sm font-medium focus:outline-none transition-colors duration-200',
          ]"
        >
          <span class="relative">
            Income
            <span
              v-if="type === 'Income'"
              class="absolute inset-x-0 bottom-0 bg-amber-500 h-[6px] -z-10 opacity-40"
            ></span>
          </span>
        </button>
      </div>

      <div>
        <label
          for="category"
          class="block text-sm font-medium text-amber-900 mb-1"
          >Category</label
        >
        <select
          id="category"
          v-model="category"
          class="block w-full px-3 py-2 bg-amber-50 border border-amber-200 rounded-sm text-sm focus:outline-none focus:ring-2 focus:ring-amber-600"
          required
        >
          <option value="" disabled>Select a category</option>
          <option v-for="cat in categoryOptions[type]" :key="cat" :value="cat">
            {{ cat }}
          </option>
        </select>
      </div>

      <div>
        <label
          for="amount"
          class="block text-sm font-medium text-amber-900 mb-1"
          >Amount</label
        >
        <div class="relative">
          <div
            class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none"
          >
            <span class="text-amber-700">$</span>
          </div>
          <input
            type="number"
            id="amount"
            v-model="amount"
            class="block w-full pl-8 pr-3 py-2 bg-amber-50 border border-amber-200 rounded-sm text-sm focus:outline-none focus:ring-2 focus:ring-amber-600"
            placeholder="0.00"
            step="0.01"
            min="0"
            required
          />
        </div>
      </div>

      <div>
        <label for="date" class="block text-sm font-medium text-amber-900 mb-1"
          >Date</label
        >
        <input
          type="date"
          id="date"
          v-model="date"
          class="block w-full px-3 py-2 bg-amber-50 border border-amber-200 rounded-sm text-sm focus:outline-none focus:ring-2 focus:ring-amber-600"
          required
        />
      </div>

      <div>
        <label for="note" class="block text-sm font-medium text-amber-900 mb-1"
          >Note</label
        >
        <textarea
          id="note"
          v-model="note"
          rows="2"
          class="block w-full px-3 py-2 bg-amber-50 border border-amber-200 rounded-sm text-sm focus:outline-none focus:ring-2 focus:ring-amber-600"
          placeholder="Add a note (optional)"
        ></textarea>
      </div>

      <button
        type="submit"
        class="w-full py-3 bg-amber-800 hover:bg-amber-900 text-white font-medium rounded-sm text-sm focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-amber-700 mt-2"
      >
        {{ editTransaction ? "Update" : "Save" }} Transaction
      </button>
    </form>
  </div>
</template>
