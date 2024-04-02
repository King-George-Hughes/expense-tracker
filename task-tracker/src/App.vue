<template>
  <Header />

  <div class="container">
    <Balance :total="+total" />
    <IncomeExpense :income="+income" :expense="+expense" />
    <TransactionList
      :transactions="transactions"
      @transactionDeleted="handleTransactionDeleted"
    />
    <AddTransaction @transactionSubmitted="handleTransactionSubmitted" />
  </div>
</template>

<script setup>
import {
  Header,
  Balance,
  IncomeExpense,
  TransactionList,
  AddTransaction,
} from "./components";

import { ref, computed, onMounted } from "vue";
import { useToast } from "vue-toastification";

const toast = useToast();

const transactions = ref([]);

onMounted(() => {
  const savedTransactions = JSON.parse(localStorage.getItem("transactions"));

  if (savedTransactions) {
    transactions.value = savedTransactions;
  }
});

// Get Total
const total = computed(() => {
  return transactions.value.reduce((acc, transaction) => {
    return acc + transaction.amount;
  }, 0);
});

// Get Income
const income = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.amount > 0)
    .reduce((acc, val) => acc + val.amount, 0)
    .toFixed(2);
});

// Get Expense
const expense = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.amount < 0)
    .reduce((acc, val) => acc + val.amount, 0)
    .toFixed(2);
});

// Handle Transaction
const handleTransactionSubmitted = (transactionData) => {
  transactions.value.push({
    id: generateUniqueId(),
    ...transactionData,
  });

  saveToLocalStorage();

  toast.success("Transaction Successfully added");
  // console.log(transactionData);
};

// Generate Unique Id
const generateUniqueId = () => Math.floor(Math.random() * 100000);

// Delete Transaction
const handleTransactionDeleted = (id) => {
  transactions.value = transactions.value.filter(
    (transaction) => +transaction.id !== +id
  );

  saveToLocalStorage();

  toast.success("Transaction Successfully deleted");
};

// Save to Local Storage
const saveToLocalStorage = () => {
  localStorage.setItem("transactions", JSON.stringify(transactions.value));
};
</script>
