<script>
  import { createEventDispatcher } from 'svelte';

  export let expenses = {};
  export let currentMonth;
  export let totalBudget;

  const dispatch = createEventDispatcher();

  $: total = Math.round(Object.values(expenses).reduce((sum, expense) => sum + expense.value, 0));
  $: remaining = totalBudget - total;
  $: monthName = formatMonth(currentMonth);
  $: budgetDifference = totalBudget - total;

  function updateBudget(event) {
    const newBudget = Math.round(parseFloat(event.target.value) || 0);
    dispatch('updateBudget', newBudget);
  }

  function formatMonth(monthString) {
  if (monthString !== currentMonth) {
    console.log('Received currentMonth:', monthString);
    const [year, month] = monthString.split('-');
    const date = new Date(year, parseInt(month) - 1, 1);
    console.log('Formatted date:', date);
    currentMonth = monthString;
  }
  return new Date(currentMonth.split('-')[0], parseInt(currentMonth.split('-')[1]) - 1, 1).toLocaleString('default', { month: 'long', year: 'numeric' });
}
</script>

<div class="total-expenses">
  <h3>Total Expenses for {monthName}</h3>
  <div class="budget-input">
    <label for="totalBudget">Monthly Budget:</label>
    <div class="input-wrapper">
      <span class="currency">$</span>
      <input type="number" id="totalBudget" value={totalBudget} on:change={updateBudget} min="0" step="1">
    </div>
  </div>
  <p class="total">Spent: ${total} / ${totalBudget}</p>
  {#if budgetDifference >= 0}
    <p class="remaining">Remaining: ${remaining}</p>
  {:else}
    <p class="budget-difference over-budget">
      Over budget by: ${Math.abs(budgetDifference)}
    </p>
  {/if}
</div>

<style>
  .total-expenses {
    background-color: var(--surface);
    padding: 1.5rem;
    border-radius: 4px;
    margin-bottom: 1rem;
    text-align: center;
    border: 1px solid rgba(255, 255, 255, 0.2); /* Add a soft color border outline */
  }

  h3 {
    margin: 0 0 1rem 0;
    color: var(--text);
    font-size: 1.5rem;
  }

  .budget-input {
    display: flex;
    justify-content: center;
    align-items: center;
    margin-bottom: 1rem;
    font-size: 1.3rem;
  }

  .budget-input label {
    margin-right: 0.5rem;
    color: var(--text);
  }

  .input-wrapper {
    display: flex;
    align-items: center;
    background-color: var(--background);
    border: 1px solid rgba(255, 255, 255, 0.1); /* Softer border color */
    border-radius: 4px;
    padding: 0.25rem 0.5rem;
  }

  .currency {
    color: var(--text);
    margin-right: 0.25rem;
  }

  .budget-input input {
    width: 150px;
    padding: 0.5rem;
    font-size: 1.3rem;
    border: none;
    background-color: transparent;
    color: var(--text);
  }

  .total, .remaining, .budget-difference {
    font-size: 1.5rem;
    font-weight: bold;
    margin: 0.5rem 0;
  }

  .total {
    color: var(--primary);
  }

  .remaining {
    color: goldenrod; /* Changed from var(--secondary) to goldenrod */
  }

  .over-budget {
    color: #FF4136;
  }
</style>