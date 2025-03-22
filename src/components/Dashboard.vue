<script setup lang="ts">
import { computed } from "vue";

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

const totalIncome = computed(() => {
  return props.transactions
    .filter((tx) => tx.type === "Income")
    .reduce((sum, tx) => sum + tx.amount, 0);
});

const totalExpense = computed(() => {
  return props.transactions
    .filter((tx) => tx.type === "Expense")
    .reduce((sum, tx) => sum + tx.amount, 0);
});

const balance = computed(() => {
  return totalIncome.value - totalExpense.value;
});

const expensesByCategory = computed(() => {
  const categories: { [key: string]: number } = {};

  props.transactions
    .filter((tx) => tx.type === "Expense")
    .forEach((tx) => {
      if (!categories[tx.category]) {
        categories[tx.category] = 0;
      }
      categories[tx.category] += tx.amount;
    });

  return Object.entries(categories)
    .map(([category, amount]) => ({ category, amount }))
    .sort((a, b) => (b.amount as number) - (a.amount as number));
});

// Removed unused computed property

const recentTransactions = computed(() => {
  return [...props.transactions]
    .sort((a, b) => new Date(b.date).getTime() - new Date(a.date).getTime())
    .slice(0, 5);
});

const formatCurrency = (amount: number) => {
  return new Intl.NumberFormat("en-US", {
    style: "currency",
    currency: "USD",
  }).format(amount);
};
</script>

<template>
  <div class="space-y-8">
    <!-- Balance Section -->
    <div class="text-center">
      <h3 class="text-md font-medium text-amber-800 mb-1">Current Balance</h3>
      <p
        class="text-3xl font-bold"
        :class="balance >= 0 ? 'text-amber-800' : 'text-red-700'"
      >
        {{ formatCurrency(balance) }}
      </p>

      <!-- Income/Expense Summary -->
      <div
        class="flex justify-between mt-4 pt-4 border-t border-dotted border-amber-300"
      >
        <div class="text-center">
          <p class="text-sm font-medium text-amber-700">Income</p>
          <p class="text-xl font-semibold text-amber-800">
            {{ formatCurrency(totalIncome) }}
          </p>
        </div>
        <div class="text-center">
          <p class="text-sm font-medium text-amber-700">Expenses</p>
          <p class="text-xl font-semibold text-red-700">
            {{ formatCurrency(totalExpense) }}
          </p>
        </div>
      </div>
    </div>

    <!-- Category Breakdowns -->
    <div class="border-t border-dotted border-amber-300 pt-4">
      <h3 class="text-lg font-semibold mb-4 text-amber-800">
        Expense Categories
      </h3>

      <div
        v-if="expensesByCategory.length === 0"
        class="text-amber-700 text-center py-4"
      >
        No expense data to display
      </div>

      <div v-else class="space-y-4">
        <div
          v-for="item in expensesByCategory"
          :key="item.category"
          class="space-y-1 pb-3 border-b border-dotted border-amber-200"
        >
          <div class="flex justify-between items-center">
            <span class="text-sm font-medium text-amber-900">{{
              item.category
            }}</span>
            <span class="text-sm font-semibold text-amber-900">{{
              formatCurrency(item.amount as number)
            }}</span>
          </div>
          <div class="h-2 bg-amber-100 rounded-sm overflow-hidden">
            <div
              class="h-full bg-amber-600 rounded-sm"
              :style="{ width: `${(item.amount / totalExpense) * 100}%` }"
            ></div>
          </div>
        </div>
      </div>
    </div>

    <!-- Recent Transactions -->
    <div class="border-t border-dotted border-amber-300 pt-4">
      <h3 class="text-lg font-semibold mb-4 text-amber-800">
        Recent Transactions
      </h3>

      <div
        v-if="recentTransactions.length === 0"
        class="text-amber-700 text-center py-4"
      >
        No transactions yet
      </div>

      <div v-else class="space-y-0">
        <div
          v-for="tx in recentTransactions"
          :key="tx.id"
          class="flex justify-between items-center py-3 border-b border-dotted border-amber-200"
        >
          <div>
            <div class="font-medium text-amber-900">{{ tx.category }}</div>
            <div class="text-xs text-amber-700">
              {{ new Date(tx.date).toLocaleDateString() }}
            </div>
          </div>
          <div
            :class="tx.type === 'Expense' ? 'text-red-700' : 'text-amber-800'"
            class="font-semibold"
          >
            {{ tx.type === "Expense" ? "-" : "+"
            }}{{ formatCurrency(tx.amount) }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
