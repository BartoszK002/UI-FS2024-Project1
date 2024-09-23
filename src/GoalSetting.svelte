<script>
  import { createEventDispatcher } from 'svelte';
  import { 
    FaUtensils, FaHome, FaBolt, FaBus, FaFilm, FaMedkit, 
    FaGraduationCap, FaShoppingBag, FaPlane, FaEllipsisH,
    FaShieldAlt, FaPhoneAlt, FaShoppingCart, FaTshirt,
    FaCar, FaCut
  } from 'svelte-icons/fa';

  export let expenses = {};
  export let goals = {};

  const dispatch = createEventDispatcher();

  export let monthlyExpenses;
  export let currentMonth;

  let selectedCategory = Object.keys(expenses)[0];
  let goalType = 'spending';
  let goalAmount = '';
  let goalTimeframe = 'monthly';

  let showNotification = false;
  let notificationMessage = '';

  function setGoal() {
    if (selectedCategory && goalAmount) {
      console.log('Setting goal:', {
        category: selectedCategory,
        type: goalType,
        amount: parseFloat(goalAmount),
        timeframe: goalTimeframe
      });
      dispatch('updateGoal', {
        category: selectedCategory,
        type: goalType,
        amount: parseFloat(goalAmount),
        timeframe: goalTimeframe
      });
      goalAmount = '';
      showNotificationMessage(`Goal added successfully for ${selectedCategory}!`);
    }
  }

  $: console.log('Goals:', goals);

  function removeGoal(category) {
    console.log('Removing goal for category:', category);
    if (goals[category]) {
      const updatedGoals = { ...goals };
      delete updatedGoals[category]; // Remove the goal from the updatedGoals object
      goals = updatedGoals; // Assign the updated object back to goals
      console.log('Goal removed from local goals object:', goals);
      dispatch('updateGoal', {
        category: category,
        type: null,
        amount: null,
        timeframe: null
      });
      console.log('Dispatched updateGoal event for category:', category);
      showNotificationMessage(`Goal removed for ${category}!`);
    }
  }

  function showNotificationMessage(message) {
    notificationMessage = message;
    showNotification = true;
    setTimeout(() => {
      showNotification = false;
    }, 3000); // Hide notification after 3 seconds
  }

  function calculateProgress(category, goal) {
    if (!expenses[category] && !monthlyExpenses[currentMonth]?.categories[category]) {
      return 0;
    }

    let progress;
    const currentDate = new Date();
    const currentYear = currentDate.getFullYear();

    if (goal.timeframe === 'yearly') {
      const yearlyTotal = Object.entries(monthlyExpenses).reduce((sum, [monthKey, month]) => {
        const monthDate = new Date(monthKey);
        if (monthDate.getFullYear() === currentYear) {
          return sum + (month.categories[category]?.value || 0);
        }
        return sum;
      }, 0);

      if (goal.type === 'spending') {
        progress = (yearlyTotal / goal.amount) * 100;
      } else if (goal.type === 'saving') {
        const remaining = goal.amount - (expenses[category]?.max - yearlyTotal);
        progress = ((goal.amount - remaining) / goal.amount) * 100;
      }
    } else {
      const expenseValue = expenses[category]?.value || monthlyExpenses[currentMonth]?.categories[category]?.value || 0;
      const budget = expenses[category]?.max || monthlyExpenses[currentMonth]?.categories[category]?.max || 0;

      if (goal.type === 'spending') {
        progress = (expenseValue / goal.amount) * 100;
      } else if (goal.type === 'saving') {
        const remaining = goal.amount - (budget - expenseValue);
        progress = ((goal.amount - remaining) / goal.amount) * 100;
      }
    }

    return progress;
  }


  function getProgressColor(progress) {
    if (progress <= 80) return '#4CAF50';
    if (progress <= 100) return '#FFA500';
    return '#FF4136';
  }

  function getEstimatedYearlyTotal(category, goal) {
    const currentDate = new Date();
    const currentYear = currentDate.getFullYear();
    
    if (goal.timeframe === 'yearly') {
      const yearlyTotal = Object.entries(monthlyExpenses).reduce((sum, [monthKey, month]) => {
        const monthDate = new Date(monthKey);
        if (monthDate.getFullYear() === currentYear) {
          return sum + (month.categories[category]?.value || 0);
        }
        return sum;
      }, 0);
      console.log(`Yearly total for ${category} in ${currentYear}: ${yearlyTotal}`);
      return yearlyTotal;
    } else {
      const expenseValue = expenses[category]?.value || monthlyExpenses[currentMonth]?.categories[category]?.value || 0;
      const budget = expenses[category]?.max || monthlyExpenses[currentMonth]?.categories[category]?.max || 0;
      return budget - expenseValue;
    }
  }

  function getProgressText(category, goal) {
    const progress = calculateProgress(category, goal);
    const yearlyTotal = getEstimatedYearlyTotal(category, goal);
    const remaining = goal.amount - (goal.timeframe === 'yearly' ? (expenses[category]?.max - yearlyTotal) : (expenses[category]?.max - expenses[category]?.value));

    if (!expenses[category] && !monthlyExpenses[currentMonth]?.categories[category]) {
      return { text: 'Category not available this month', class: 'not-available' };
    }

    if (goal.type === 'spending') {
      const expenseValue = expenses[category]?.value || monthlyExpenses[currentMonth]?.categories[category]?.value || 0;
      const amountLeft = goal.amount - expenseValue;
      if (amountLeft < 0) {
        return { text: `Over by $${Math.abs(amountLeft).toFixed(2)}`, class: 'over-amount' };
      } else {
        return { text: `$${amountLeft.toFixed(2)} left`, class: 'remaining' };
      }
    } else if (goal.type === 'saving') {
      if (remaining <= 0) {
        return { text: 'Goal achieved!', class: 'goal-achieved' };
      } else {
        return { text: `$${remaining.toFixed(2)} left to save`, class: 'remaining' };
      }
    }
  }
  
  // Use of AI-generated code to help with the icon mapping
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
</script>

<div class="goal-setting">
  <h3>Set a New Financial Goal</h3>
  <div class="goal-form">
    <div class="form-group">
      <label for="category-select">Category:</label>
      <select id="category-select" bind:value={selectedCategory}>
        {#each Object.keys(expenses) as category}
          <option value={category}>{category}</option>
        {/each}
      </select>
    </div>
    
    <div class="form-group">
      <label for="goal-type-select">Goal Type:</label>
      <select id="goal-type-select" bind:value={goalType}>
        <option value="spending">Spending Limit</option>
        <option value="saving">Savings Target ($ Remaining in Budget)</option>
      </select>
    </div>
    
    <div class="form-group">
      <label for="goal-amount-input">Goal Amount ($):</label>
      <input id="goal-amount-input" type="number" bind:value={goalAmount} placeholder="Enter amount">
    </div>
    
    <div class="form-group">
      <label for="goal-timeframe-select">Timeframe:</label>
      <select id="goal-timeframe-select" bind:value={goalTimeframe}>
        <option value="monthly">Monthly</option>
        <option value="yearly">Yearly</option>
      </select>
    </div>
    
    <button on:click={setGoal}>Set Goal</button>
  </div>
  
  <div class="goals-list">
    <h4>Current Goals:</h4>
    {#each Object.entries(goals) as [category, goal]}
      {#if expenses[category]}
        <div class="goal-item">
          <div class="goal-header">
            <div class="icon-container">
              <svelte:component this={getIconComponent(category)} />
            </div>
            <span class="category-name">{category}</span>
          </div>
          <div class="goal-details">
            <div class="goal-info">
              <span class="goal-type">{goal.type} goal: ${goal.amount} {goal.timeframe}</span>
              {#if goal.type === 'spending'}
                <span class="spent bold">
                  ${goal.timeframe === 'yearly' ? getEstimatedYearlyTotal(category, goal).toFixed(2) : expenses[category].value.toFixed(2)} spent
                </span>
              {:else if goal.type === 'saving'}
                <span class="saved bold">
                  ${goal.timeframe === 'yearly' ? (goal.amount - getEstimatedYearlyTotal(category, goal)).toFixed(2) : (expenses[category].max - expenses[category].value).toFixed(2)} saved
                </span>
              {/if}
              {#if getProgressText(category, goal)}
                {@const progressInfo = getProgressText(category, goal)}
                <span class="progress-text {progressInfo.class}">{progressInfo.text}</span>
              {/if}
            </div>
            <div class="progress-container">
              <div class="progress-bar-container">
                <div class="progress-bar" style="height: {Math.min(calculateProgress(category, goal), 100)}%; background-color: {getProgressColor(calculateProgress(category, goal))};"></div>
              </div>
              <span class="progress-percentage">{Math.max(calculateProgress(category, goal), 0).toFixed(1)}%</span>
            </div>
          </div>
          <button class="remove-goal-button" on:click={() => removeGoal(category)}>Remove</button>
        </div>
      {/if}
    {/each}
  </div>

  {#if showNotification}
    <div class="notification">
      {notificationMessage}
    </div>
  {/if}
</div>

<style>
  .goal-setting {
    background-color: var(--surface);
    padding: 1rem 0.1rem;
    border-radius: 4px;
    height: 100%;
    box-sizing: border-box;
    display: flex;
    flex-direction: column;
  }

  h3 {
    margin-top: 0;
    margin-bottom: 0.75rem;
    color: var(--text);
  }

  .goal-form {
    display: flex;
    flex-direction: column;
    gap: 0.65rem;
    margin-bottom: 0.75rem;
  }

  .form-group {
    display: flex;
    flex-direction: column;
    gap: 0.1rem;
  }

  .form-group label {
    font-size: 0.9em;
    color: var(--text);
  }

  .goal-form select, .goal-form input {
    width: 100%;
    padding: 0.4rem;
    border: 1px solid rgba(255, 255, 255, 0.1);
    border-radius: 4px;
    background-color: var(--background);
    color: var(--text);
    box-sizing: border-box;
  }

  .goal-form button {
    padding: 0.4rem 1rem;
    background-color: var(--primary);
    color: var(--text);
    border: 1px solid rgba(255, 255, 255, 0.1);
    border-radius: 4px;
    cursor: pointer;
    align-self: flex-start;
    margin-top: 0.25rem;
    transition: background-color 0.2s, color 0.2s, border-color 0.2s;
  }

  .goal-form button:hover {
    background-color: var(--secondary);
  }

  .goals-list {
    margin-top: 0rem;
    padding: 0 0;
  }

  .goal-item {
    background-color: rgba(0, 0, 0, 0.1);
    padding: 0.5rem 0.5rem;
    border-radius: 4px;
    margin-bottom: 0.4rem;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    border: 1px solid rgba(255, 255, 255, 0.1);
  }

  .goal-header {
    display: flex;
    align-items: center;
    margin-bottom: 0.5rem;
  }

  .icon-container {
    width: 20px;
    height: 20px;
    margin-right: 0.5rem; /* Increased space between icon and text */
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .icon-container :global(svg) {
    width: 100%;
    height: 100%;
    color: var(--primary);
  }

  .category-name {
    font-weight: bold;
    font-size: 1.1em;
    color: var(--text);
  }

  .goal-details {
    display: flex;
    align-items: stretch;
    justify-content: space-between;
  }

  .goal-info {
    flex: 1;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
  }

  .goal-type, .spent, .progress-text {
    font-size: 0.85em;
    margin-bottom: 0.2rem;
  }

  .goal-type {
    font-weight: bold;
    color: var(--text);
  }

  .spent {
    color: var(--primary);
  }

  .bold {
    font-weight: bold;
  }

  .over-amount {
    color: #FF4136;
    font-weight: bold;
  }

  .remaining {
    color: goldenrod;
    font-weight: bold;
  }

  .progress-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    margin-left: 0.25rem;
  }

  .progress-bar-container {
    width: 12px;
    height: 50px;
    background-color: var(--background);
    border-radius: 6px;
    overflow: hidden;
    margin-bottom: 0.2rem;
    position: relative;
  }

  .progress-bar {
    width: 100%;
    background-color: var(--primary);
    position: absolute;
    bottom: 0;
    left: 0;
    transition: height 0.3s ease-in-out, background-color 0.3s ease-in-out;
  }

  .progress-percentage {
    font-size: 0.8em;
    font-weight: bold;
    color: var(--text);
  }

  .goal-icon {
    font-size: 1.2em;
  }

  .goal-label {
    margin-left: 4px;
  }

  .remove-goal-button {
    background-color: #ff4136;
    color: black;
    border: none;
    padding: 0.2rem 0.5rem;
    border-radius: 4px;
    cursor: pointer;
    font-size: 0.8em;
    align-self: flex-end;
    margin-top: 0.5rem;
    transition: background-color 0.2s, color 0.2s;
  }

  .remove-goal-button:hover {
    background-color: darkred;
  }

  .notification {
    position: fixed;
    bottom: 20px;
    right: 20px;
    background-color: var(--primary);
    color: var(--text-dark);
    padding: 10px 20px;
    border-radius: 4px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
    z-index: 1000;
    animation: fadeIn 0.3s, fadeOut 0.3s 2.7s;
  }

  @keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
  }

  @keyframes fadeOut {
    from { opacity: 1; }
    to { opacity: 0; }
  }
</style>