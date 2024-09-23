<script>
  import { createEventDispatcher } from 'svelte';
  import { 
    FaUtensils, FaHome, FaBolt, FaBus, FaFilm, FaMedkit, 
    FaGraduationCap, FaShoppingBag, FaPlane, FaEllipsisH,
    FaShieldAlt, FaPhoneAlt, FaShoppingCart, FaTshirt,
    FaCar, FaCut
  } from 'svelte-icons/fa';

  export let category;
  export let icon;
  export let value;
  export let max;
  export let isCustom = false;
  export let sliderElement;

  const dispatch = createEventDispatcher();

  $: percentage = (value / max) * 100;
  $: colorClass = percentage >= 100 ? 'exceeded' : 
                  percentage >= 80 ? 'warning' : 'normal';

$: remaining = max - value;
$: isOverBudget = remaining < 0;
$: overBudgetAmount = Math.abs(remaining);

$: if (sliderElement && max !== undefined) {
  sliderElement.max = max;
  if (value !== null && value <= max) {
    sliderElement.value = value;
  }
}

function updateExpense(newValue) {
  if (newValue.trim() === '') {
    value = null;
  } else {
    // Allow decimal input but round to 2 decimal places
    value = Math.round(parseFloat(newValue) * 100) / 100;
  }
  dispatch('update', { category, value, max });
}

function updateMaxBudget(newMax) {
  if (newMax.trim() === '') {
    max = null;
  } else {
    max = parseFloat(newMax) || 0;
    
    // Update the slider's max value
    if (sliderElement) {
      sliderElement.max = max;
      
      // Update the slider's value to match the current spent amount
      if (value !== null && value <= max) {
        sliderElement.value = value;
      }
    }
  }
  dispatch('update', { category, value, max });
}

  function handleSliderChange(newValue) {
    value = parseFloat(newValue) || 0;
    dispatch('update', { category, value, max });
  }

  function handleRemove() {
    dispatch('remove');
  }

  function getIconComponent(category) {
    const iconMap = {
      "Housing": FaHome,
      "Groceries": FaShoppingCart,
      "Utilities": FaBolt,
      "Transportation": FaBus,
      "Dining Out": FaUtensils,
      "Healthcare": FaMedkit,
      "Insurance": FaShieldAlt,
      "Entertainment": FaFilm,
      "Shopping": FaShoppingBag,
      "Education": FaGraduationCap,
      "Travel": FaPlane,
      "Personal Care": FaCut,
      "Other": FaEllipsisH
    };
    return iconMap[category] || FaEllipsisH;
  }

  console.log('Received icon for', category, ':', icon);
</script>

<div class="expense-item">
  <div class="category-info">
    <div class="category-icon">
      <svelte:component this={getIconComponent(category)} />
    </div>
    <div class="label-container">
      <label for={category}>{category}</label>
    </div>
  </div>
  <div class="expense-details">
    <div class="input-row">
      <div class="input-group">
        <span>Spent:</span>
        <div class="input-wrapper">
          <span class="currency">$</span>
          <input 
            type="number" 
            id={category} 
            bind:value 
            on:input={(e) => updateExpense(e.target.value)}
            min="0" 
            max={max}
            step="1"
            placeholder="$ Spent"
          >
        </div>
      </div>
      <div class="budget-group">
        <span>Budget:</span>
        <div class="input-wrapper">
          <span class="currency">$</span>
          <input 
            type="number" 
            bind:value={max} 
            on:input={(e) => updateMaxBudget(e.target.value)}
            min="0" 
            step="1"
            placeholder="Budget Amt. $"
          >
        </div>
      </div>
    </div>
    <div class="slider-row">
      <input 
      type="range" 
      bind:this={sliderElement}
      bind:value 
      on:input={(e) => updateExpense(e.target.value)}
      min="0" 
      max={max} 
      step="1"
    >
      <div class="remaining">
        {#if isOverBudget}
          <span class="over-budget-label">Over-budget: </span>
          <span class="amount over-budget">${overBudgetAmount.toFixed(2)}</span>
        {:else}
          <span>Remaining: </span>
          <span class="amount">${remaining.toFixed(2)}</span>
        {/if}
      </div>
    </div>
    <div class="spent-budget-label">
      <span class="spent-budget" class:over-budget={isOverBudget}>${value?.toFixed(2) || '0.00'} / ${max?.toFixed(2) || '0.00'}</span>
      {#if isCustom}
        <button class="remove-button" on:click={handleRemove}>Remove</button>
      {/if}
    </div>
  </div>
</div>

<style>

.over-budget-label {
  color: #ff0000 !important;
}

.over-budget {
  color: var(--error);
}

.spent-budget-label {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-top: 0.13rem;
  width: 100%;
}

.spent-budget {
  font-weight: bold;
  font-size: 0.9em;
  color: var(--primary);
}

.spent-budget.over-budget {
  color: #ff0000 !important;
}
  .expense-item {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    margin-bottom: 1rem;
    padding: 0.75rem;
    background-color: var(--surface);
    border-radius: 4px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    border: 1px solid rgba(255, 255, 255, 0.1); /* Softer border color */
  }

  .category-info {
    display: flex;
    align-items: center;
    width: 100%;
    margin-bottom: 0.75rem;
  }

  .category-icon {
    width: 24px;
    height: 24px;
    flex-shrink: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    margin-right: 0.5rem; /* Consistent spacing */
  }

  .category-icon :global(svg) {
    width: 100%;
    height: 100%;
    color: var(--primary);
  }

  .label-container {
    flex-grow: 1;
    display: flex;
    align-items: center;
  }

  label {
    font-weight: bold;
    font-size: 1em;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    display: block;
    padding: 2px 0;
  }

  .expense-details {
    width: 100%;
    padding-right: 0;
    flex-grow: 1;
    display: flex;
    flex-direction: column;
    min-width: 0;
  }

  .input-row {
    display: flex;
    justify-content: space-between;
    margin-bottom: 0.5rem;
    width: 100%;
  }

  .input-group, .budget-group {
    display: flex;
    align-items: center;
    flex: 1;
  }

  .input-group {
    flex: 1;
    margin-right: 0.5rem;
  }

  .budget-group {
    flex: 1;
  }

  .input-group span, .budget-group span {
    width: 50px;
    margin-right: 0.25rem;
    font-size: 0.9em;
  }

  .input-wrapper {
    display: flex;
    align-items: center;
    background-color: var(--background);
    border: 1px solid rgba(255, 255, 255, 0.1); /* Softer border color */
    border-radius: 4px;
    padding: 0 2px;
    flex: 1;
  }

  .currency {
    color: var(--text);
    margin-right: 0px;
    font-size: 0.9em;
  }

  input[type="number"] {
    width: 100%;
    min-width: 60px;
    max-width: 100px;
    padding: 4px 2px 4px 0;
    border: none;
    background-color: transparent;
    color: var(--text);
    font-size: 0.9em;
  }

  .budget-group {
    margin-right: 0.5rem;
    justify-content: flex-end;
  }

  .slider-row {
    display: flex;
    align-items: center;
    margin-bottom: 0.13rem; /* Further reduced from 0.15rem (about 15% reduction) */
  }

  input[type="range"] {
    flex-grow: 1;
    margin-right: 0.5rem;
    -webkit-appearance: none;
    background: rgba(255, 255, 255, 0.1); /* Softer color for the range track */
    border-radius: 5px;
    height: 5px;
  }

  input[type="range"]::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    width: 15px;
    height: 15px;
    border-radius: 50%;
    background: var(--primary);
    cursor: pointer;
  }

  input[type="range"]::-moz-range-thumb {
    width: 15px;
    height: 15px;
    border-radius: 50%;
    background: var(--primary);
    cursor: pointer;
  }

  .remaining {
    white-space: nowrap;
    font-size: 0.8em;
  }

  .remaining .amount {
    font-weight: bold;
    color: goldenrod; /* Changed from var(--secondary) to goldenrod */
  }

  .spent-budget-label {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-top: 0.13rem;
    width: 100%;
  }

  .spent-budget {
    font-weight: bold;
    font-size: 0.9em;
    color: var(--primary);
  }

  .remove-button {
    background-color: #ff4136;
    color: var(--text-dark);
    border: none;
    padding: 5px 10px;
    border-radius: 4px;
    cursor: pointer;
    font-size: 0.8em;
    margin-left: 10px; /* Add some space between the budget text and the button */
  }

  .remove-button:hover {
    background-color: #d63a30; /* Darker shade for hover effect */
  }
</style>