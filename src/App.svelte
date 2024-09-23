<script>
  import { onMount } from 'svelte';
  import { 
    FaUtensils, FaHome, FaBolt, FaBus, FaFilm, FaMedkit, 
    FaGraduationCap, FaShoppingBag, FaPlane, FaEllipsisH,
    FaShieldAlt, FaShoppingCart, FaTshirt,
    FaCar, FaCut, FaChartLine, FaChartPie, FaCalendarAlt
  } from 'svelte-icons/fa';
  import ExpenseItem from './ExpenseItem.svelte';
  import TotalExpenses from './TotalExpenses.svelte';
  import AddExpenseCategory from './AddExpenseCategory.svelte';
  import ExpensesOverview from './ExpensesOverview.svelte';
  import GoalSetting from './GoalSetting.svelte';
  import { writable } from 'svelte/store';

  let name = "Bartosz Kawalkowski";
  let currentDate = new Date();
  let currentTime = currentDate.toLocaleTimeString();
  let daysSinceJoining = 30; // This should be calculated based on the user's join date
  let daysActive = 25; // This should be tracked and stored, in local storage or a database
  let sessionStartTime = new Date();
  let sessionDuration = '0:00:00';

  // Create a store for monthly budgets
  const monthlyBudgets = writable({});

  const customCategories = writable([]);
  

  // Function to generate dummy data for a month
  function generateMonthData() {
    const data = {
      "Housing": { icon: FaHome, value: Math.floor(Math.random() * 1500), max: 1500 },
      "Groceries": { icon: FaShoppingCart, value: Math.floor(Math.random() * 500), max: 500 },
      "Utilities": { icon: FaBolt, value: Math.floor(Math.random() * 200), max: 200 },
      "Transportation": { icon: FaBus, value: Math.floor(Math.random() * 200), max: 200 },
      "Dining Out": { icon: FaUtensils, value: Math.floor(Math.random() * 300), max: 300 },
      "Healthcare": { icon: FaMedkit, value: Math.floor(Math.random() * 300), max: 300 },
      "Insurance": { icon: FaShieldAlt, value: Math.floor(Math.random() * 200), max: 200 },
      "Entertainment": { icon: FaFilm, value: Math.floor(Math.random() * 200), max: 200 },
      "Shopping": { icon: FaShoppingBag, value: Math.floor(Math.random() * 300), max: 300 },
      "Education": { icon: FaGraduationCap, value: Math.floor(Math.random() * 200), max: 200 },
      "Travel": { icon: FaPlane, value: Math.floor(Math.random() * 500), max: 500 },
      "Personal Care": { icon: FaCut, value: Math.floor(Math.random() * 100), max: 100 },
      "Other": { icon: FaEllipsisH, value: Math.floor(Math.random() * 200), max: 200 }
    };
    const totalBudget = Object.values(data).reduce((sum, category) => sum + category.max, 0);
    return { categories: data, totalBudget };
  }

  export let monthlyExpenses = {};
  let currentMonth;
  let currentMonthIndex;
  let monthList = [];
  let currentMonthData;
  let expenses;
  let totalBudget;

  let displayMonth;
  let displayMonthUpdateCount = 0;

  function updateCurrentMonthData() {
  const newMonthData = monthlyExpenses[currentMonth];
  if (newMonthData !== currentMonthData) {
    currentMonthData = newMonthData;
    expenses = { ...currentMonthData.categories, ...updateCustomCategories(currentMonth) };
    totalBudget = currentMonthData.totalBudget;

    console.log('Updated current month data:', currentMonth);
    console.log('Current expenses:', expenses);
    console.log('Current total budget:', totalBudget);
  }
}

function initializeMonths() {
  let startDate = new Date();
  startDate.setMonth(startDate.getMonth()); // Keep the current month
  startDate.setDate(1); // Set to the first day of the month
  
  for (let i = 0; i < 24; i++) { // Generate 24 months of data
    let date = new Date(startDate);
    date.setMonth(startDate.getMonth() - i);
    let monthKey = date.toISOString().split('T')[0].slice(0, 7);
    monthList.push(monthKey);
    if (!monthlyExpenses[monthKey]) {
      monthlyExpenses[monthKey] = generateMonthData();
    }
  }

  currentMonthIndex = 0; // Set to 0 for the current month
  currentMonth = monthList[currentMonthIndex];
  currentMonthData = monthlyExpenses[currentMonth];
  expenses = currentMonthData.categories;
  totalBudget = currentMonthData.totalBudget;

  console.log('Initialized current month:', currentMonth, 'Index:', currentMonthIndex);

  // Update displayMonth
  updateDisplayMonth();
}


function updateDisplayMonth() {
  if (currentMonth) {
    const [year, month] = currentMonth.split('-');
    const newDisplayMonth = new Date(year, parseInt(month) - 1, 1);
    
    if (!displayMonth || newDisplayMonth.getTime() !== displayMonth.getTime()) {
      displayMonth = newDisplayMonth;
      displayMonthUpdateCount++;
      console.log(`Display month updated (${displayMonthUpdateCount}):`, 
                  displayMonth.toLocaleString('default', { month: 'long', year: 'numeric' }),
                  'Current Month:', currentMonth);
    }
  }
}

  // Call initializeMonths immediately
  initializeMonths();

  function changeMonth(delta) {
  console.log('changeMonth called with delta:', delta);
  console.log('Current index before change:', currentMonthIndex);
  
  let newIndex = currentMonthIndex + delta;
  console.log('New index before bounds check:', newIndex);
  
  if (newIndex < 0) newIndex = 0;
  if (newIndex >= monthList.length) newIndex = monthList.length - 1;
  
  console.log('New index after bounds check:', newIndex);
  
  if (newIndex !== currentMonthIndex) {
    currentMonthIndex = newIndex;
    currentMonth = monthList[currentMonthIndex];
    
    updateCurrentMonthData();
    
    // Force a rerender of the component
    expenses = {...expenses};
    
    // Apply sorting after changing the month
    sortExpenses(sortBy, sortDirection);
    
    // Update goals for the new month
    Object.keys(financialGoals).forEach(category => {
      if (expenses[category]) {
        updateFinancialGoal({
          detail: {
            ...financialGoals[category],
            category
          }
        });
      }
    });
    
    console.log('Changed to month:', currentMonth, 'Index:', currentMonthIndex);
    
    // Update displayMonth
    updateDisplayMonth();
  } else {
    console.log('Month did not change');
  }
  console.log('Current month after change:', currentMonth);
}

  let sortBy = 'spent';
  let sortDirection = 'desc';

  function toggleSort(criteria) {
    if (sortBy === criteria) {
      sortDirection = sortDirection === 'asc' ? 'desc' : 'asc';
    } else {
      sortBy = criteria;
      sortDirection = 'desc';
    }
    sortExpenses(criteria, sortDirection);
  }

  // Use of AI-generated code to help sort the monthly expenses
  function sortExpenses(criteria, direction) {
    const sortedEntries = Object.entries(expenses).sort((a, b) => {
      let aValue, bValue;
      
      switch (criteria) {
        case 'name':
          aValue = a[0].toLowerCase();
          bValue = b[0].toLowerCase();
          return direction === 'asc' ? aValue.localeCompare(bValue) : bValue.localeCompare(aValue);
        case 'spent':
          aValue = a[1].value;
          bValue = b[1].value;
          break;
        case 'budget':
          aValue = a[1].max;
          bValue = b[1].max;
          break;
        case 'remaining':
          aValue = a[1].max - a[1].value;
          bValue = b[1].max - b[1].value;
          break;
        default:
          aValue = a[0];
          bValue = b[0];
      }
      
      if (criteria !== 'name') {
        return direction === 'asc' ? aValue - bValue : bValue - aValue;
      }
    });

    expenses = Object.fromEntries(sortedEntries);
  }

  let note = "";
  let overviewPeriod = 'month';

  function handleExpenseUpdate(event) {
    const { category, value, max } = event.detail;
  expenses = {
    ...expenses,
    [category]: { ...expenses[category], value: value ?? 0, max: max ?? 0 }
  };
  monthlyExpenses[currentMonth].categories = expenses;
  updateCategoryBudget(category, max ?? 0);
    
    // Update custom categories if necessary
    customCategories.update(categories => 
      categories.map(c => c.name === category ? { ...c, value, max } : c)
    );

    // Update the goal progress if there's a goal for this category
    if (financialGoals[category]) {
      updateFinancialGoal({
        detail: {
          ...financialGoals[category],
          category
        }
      });
    }
    saveToLocalStorage();
  }

  function updateTotalBudget(newBudget) {
    totalBudget = Math.round(newBudget); // Round to whole number
    monthlyExpenses[currentMonth].totalBudget = totalBudget;
    saveToLocalStorage();
  }

  function updateCategoryBudget(category, newMax) {
    expenses[category].max = Math.round(newMax); // Round to whole number
    const newTotalBudget = Object.values(expenses).reduce((sum, cat) => sum + cat.max, 0);
    totalBudget = newTotalBudget; // No need to round, sum of whole numbers is a whole number
    monthlyExpenses[currentMonth].categories = expenses;
    monthlyExpenses[currentMonth].totalBudget = totalBudget;
    saveToLocalStorage();
  }

  // Function to save data to local storage
  function saveToLocalStorage() {
    localStorage.setItem('financeTrackerData', JSON.stringify({
      monthlyExpenses,
      currentMonth,
      currentMonthIndex,
      monthList,
      note,
      financialGoals,
      customCategories: $customCategories
    }));
  }

  // Function to load data from local storage
  function loadFromLocalStorage() {
    const savedData = localStorage.getItem('financeTrackerData');
    if (savedData) {
      const parsedData = JSON.parse(savedData);
      monthlyExpenses = parsedData.monthlyExpenses;
      currentMonth = parsedData.currentMonth;
      currentMonthIndex = parsedData.currentMonthIndex;
      monthList = parsedData.monthList;
      note = parsedData.note;
      financialGoals = parsedData.financialGoals;
      customCategories.set(parsedData.customCategories || []);
      
      // Update current month data
      currentMonthData = monthlyExpenses[currentMonth];
      expenses = currentMonthData.categories;
      totalBudget = currentMonthData.totalBudget;
      
      console.log('Loaded data from localStorage');
    } else {
      initializeMonths();
    }
    expenses = {...expenses};
  }

  let showNotification = false;
  let notificationMessage = '';
  let monthName = "";

  function showSaveNotification(message) {
  console.log("Showing notification:", message);
  const [year, month] = currentMonth.split('-');
  const monthName = new Date(year, parseInt(month) - 1, 1).toLocaleString('default', { month: 'long' });
  notificationMessage = `${message} for ${monthName} ${year}`;
  showNotification = true;
  setTimeout(() => {
    showNotification = false;
  }, 3000); // Hide notification after 3 seconds
}

function showCategoryNotification(message) {
    console.log("Showing category notification:", message);
    notificationMessage = message;
    showNotification = true;
    setTimeout(() => {
      showNotification = false;
    }, 3000); // Hide notification after 3 seconds
  }

  // Modify the saveExpenses function
  function saveExpenses() {
  console.log("Expenses saved for", currentMonth, ":", Object.fromEntries(
    Object.entries(expenses).map(([k, v]) => [k, v.value])
  ));
  console.log("Note:", note);
  
  // Update the monthlyExpenses object
  monthlyExpenses[currentMonth] = {
    categories: expenses,
    totalBudget: totalBudget
  };

  // Save to local storage
  saveToLocalStorage();

  // Show notification
  showSaveNotification('Expenses saved successfully!');
}

  // Load data from local storage on component mount
  onMount(() => {
  loadFromLocalStorage();
  console.log('Data passed to ExpensesOverview:', monthlyExpenses);
  console.log('Current month:', currentMonth);
  console.log('Current month data:', monthlyExpenses[currentMonth]);
  updateCurrentMonthData(); // Ensure data is up-to-date after loading
  updateDisplayMonth();
});

function addCustomCategory(event) {
  const category = event.detail;
  customCategories.update(categories => {
    const updatedCategories = [...categories, {
      name: category,
      icon: FaEllipsisH,
      value: 0,
      max: 500,
      addedMonth: currentMonth
    }];
    return updatedCategories;
  });

  // Add the new category only to the current month's expenses
  expenses = {
    ...expenses,
    [category]: { icon: FaEllipsisH, value: 0, max: 500 }
  };
  monthlyExpenses[currentMonth].categories = expenses;
  saveToLocalStorage();

  showCategoryNotification(`Custom category "${category}" added successfully!`);

  // Force a rerender of the ExpensesOverview component
  expenses = {...expenses};
}

function removeCustomCategory(category) {
  customCategories.update(categories => {
    const updatedCategories = categories.filter(c => c.name !== category);
    return updatedCategories;
  });
  
  // Remove the category from all months' expenses
  Object.keys(monthlyExpenses).forEach(month => {
    if (monthlyExpenses[month].categories[category]) {
      const { [category]: removed, ...updatedExpenses } = monthlyExpenses[month].categories;
      monthlyExpenses[month].categories = updatedExpenses;
    }
  });
  
  // Update current month's expenses
  expenses = monthlyExpenses[currentMonth].categories;
  
  saveToLocalStorage();

  showCategoryNotification(`Custom category "${category}" removed successfully!`);
}

function updateCustomCategories(currentMonth) {
  return $customCategories
    .filter(category => category.addedMonth === currentMonth)
    .reduce((acc, category) => {
      if (!acc[category.name]) {
        acc[category.name] = { icon: category.icon, value: category.value, max: category.max };
      }
      return acc;
    }, {});
}

  function updateSessionDuration() {
    const now = new Date();
    const duration = now - sessionStartTime;
    const hours = Math.floor(duration / 3600000);
    const minutes = Math.floor((duration % 3600000) / 60000);
    const seconds = Math.floor((duration % 60000) / 1000);
    sessionDuration = `${hours}:${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
  }

  onMount(() => {
    const timer = setInterval(() => {
      currentTime = new Date().toLocaleTimeString();
      updateSessionDuration();
    }, 1000);

    return () => {
      clearInterval(timer);
    };
  });

  // Call sortExpenses initially to set the default sorting
  onMount(() => {
    sortExpenses('spent', 'desc');
  });

  // Add a new object to store financial goals
  let financialGoals = {};

  function initializeGoals() {
    const defaultGoals = {
      "Housing": { type: "spending", amount: 1000, timeframe: "monthly" },
      "Groceries": { type: "spending", amount: 400, timeframe: "monthly" },
      "Entertainment": { type: "spending", amount: 200, timeframe: "monthly" }
    };

    Object.entries(defaultGoals).forEach(([category, goal]) => {
      if (expenses[category]) {
        financialGoals[category] = goal;
      }
    });
  }

  function updateFinancialGoal(event) {
    const { category, type, amount, timeframe } = event.detail;
    if (type === null && amount === null && timeframe === null) {
      delete financialGoals[category];
    } else {
      financialGoals[category] = { type, amount, timeframe };
    }
    console.log('Updated financial goals:', financialGoals);
    saveToLocalStorage();
  }

  // Initialize goals after expenses are set up
  onMount(() => {
    initializeGoals();
  });

  // Create a store for theme
  const theme = writable('dark');

  const themes = {
    dark: {
      background: '#1e1e1e',
      surface: '#2c2c2c',
      primary: '#4caf50',
      secondary: '#3498db',
      text: '#e0e0e0',
      textDark: '#121212',
    },
    blue: {
      background: '#1a2a3a',
      surface: '#2c3e50',
      primary: '#3498db',
      secondary: '#2ecc71',
      text: '#ecf0f1',
      textDark: '#121212',
    },
    green: {
      background: '#1a3a2a',
      surface: '#2c503e',
      primary: '#2ecc71',
      secondary: '#3498db',
      text: '#ecf0f1',
      textDark: '#121212',
    },
  };

  function setTheme(newTheme) {
    theme.set(newTheme);
  }

  $: currentTheme = themes[$theme];

  $: {
  const [year, month] = currentMonth.split('-');
  const date = new Date(year, parseInt(month) - 1, 1);
  monthName = date.toLocaleString('default', { month: 'long', year: 'numeric' });
}
</script>

<main style="--background: {currentTheme.background}; --surface: {currentTheme.surface}; --primary: {currentTheme.primary}; --secondary: {currentTheme.secondary}; --text: {currentTheme.text}; --text-dark: {currentTheme.textDark};">
  <header>
    <h1>Finance Tracker</h1>
  </header>

  <section class="user-info">
    <div class="theme-container">
      <label class="theme-label" for="theme-buttons">Select Theme:</label>
      <div id="theme-buttons" class="theme-buttons">
        <button on:click={() => setTheme('dark')}>Dark</button>
        <button on:click={() => setTheme('blue')}>Blue</button>
        <button on:click={() => setTheme('green')}>Green</button>
      </div>
    </div>
    <h2>Welcome, {name}!</h2>
    <p>Current Date: {currentDate.toLocaleDateString()} | Time: {currentTime}</p>
    <p>{daysSinceJoining} days since joining | {daysActive} days active</p>
    <p>Current session duration: {sessionDuration}</p>
  </section>

  <div class="content-wrapper">
    <section class="expense-tracker">
      <h2>Monthly Expense Tracker</h2>
      <div class="month-navigation">
        <button on:click={() => {
          console.log('Previous Month button clicked');
          changeMonth(1);
        }} disabled={currentMonthIndex === monthList.length - 1}>Previous Month</button>
        <h3>{displayMonth.toLocaleString('default', { month: 'long', year: 'numeric' })}</h3>
        <button on:click={() => {
          console.log('Next Month button clicked');
          changeMonth(-1);
        }} disabled={currentMonthIndex === 0}>Next Month</button>
      </div>
      <TotalExpenses 
      expenses={expenses} 
      currentMonth={currentMonth} 
      totalBudget={totalBudget} 
      on:updateBudget={(e) => updateTotalBudget(e.detail)} 
    />
      <div class="sorting-options">
        <span>Sort by:</span>
        <button on:click={() => toggleSort('name')} class:active={sortBy === 'name'}>
          Name {sortBy === 'name' ? (sortDirection === 'asc' ? '↑' : '↓') : ''}
        </button>
        <button on:click={() => toggleSort('spent')} class:active={sortBy === 'spent'}>
          Spent {sortBy === 'spent' ? (sortDirection === 'asc' ? '↑' : '↓') : ''}
        </button>
        <button on:click={() => toggleSort('budget')} class:active={sortBy === 'budget'}>
          Budget {sortBy === 'budget' ? (sortDirection === 'asc' ? '↑' : '↓') : ''}
        </button>
        <button on:click={() => toggleSort('remaining')} class:active={sortBy === 'remaining'}>
          Remaining {sortBy === 'remaining' ? (sortDirection === 'asc' ? '↑' : '↓') : ''}
        </button>
      </div>
      <AddExpenseCategory on:addCategory={addCustomCategory} />
      <button class="save-expenses-button" on:click={saveExpenses}>Save Expenses</button>
      {@html '<!-- ' + JSON.stringify(expenses) + ' -->'}

      {#each Object.entries(expenses) as [category, expenseData]}
        <ExpenseItem 
          {category}
          icon={expenseData.icon}
          bind:value={expenseData.value}
          bind:max={expenseData.max}
          on:update={handleExpenseUpdate}
          on:remove={() => removeCustomCategory(category)}
          isCustom={$customCategories.some(c => c.name === category)}
        />
      {/each}
      <div class="note-section">
        <label for="note">Notes:</label>
        <textarea id="note" bind:value={note} placeholder="Any additional notes..."></textarea>
      </div>
    </section>

    <section class="expenses-overview">
      <div class="overview-header">
        <div class="left-section">
          <h2>Expenses Overview</h2>
          {#if overviewPeriod === 'month'}
            <p class="current-month">{monthName}</p>
          {/if}
        </div>
        <div class="right-section">
          <select bind:value={overviewPeriod}>
            <option value="month">Monthly</option>
            <option value="year">Yearly</option>
          </select>
        </div>
      </div>
      <ExpensesOverview 
        expenses={monthlyExpenses} 
        period={overviewPeriod} 
        currentMonth={currentMonth}
        totalBudget={totalBudget}
      />
    </section>

    <section class="financial-goals">
      <h2>Financial Goals</h2>
      <div class="goals-container">
        <GoalSetting 
        expenses={expenses} 
        goals={financialGoals} 
        monthlyExpenses={monthlyExpenses} 
        currentMonth={currentMonth}
        on:updateGoal={updateFinancialGoal}
      />
      </div>
    </section>
  </div>

  {#if showNotification}
    <div class="notification">
      {notificationMessage}
    </div>
  {/if}
</main>

<style>
  :global(body) {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background-color: var(--background);
    color: var(--text);
    line-height: 1.6;
    margin: 0;
    padding: 0;
  }

  main {
    max-width: 2100px;
    margin: 0 auto;
    padding: 10px 20px;
  }

  header {
    display: flex;
    align-items: center;
    justify-content: center;
    margin-bottom: 0.5rem;
  }

  h1 {
    color: var(--primary);
    margin: 0;
    padding: 5px 0;
  }

  .user-info {
    background-color: var(--surface);
    border-radius: 8px;
    padding: 15px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.2);
    margin-bottom: 1rem;
    position: relative;
  }

  .theme-container {
    position: absolute;
    top: 0;
    right: 0;
    display: flex;
    align-items: center;
    padding: 5px 10px;
    border-radius: 5px;
  }

  .theme-label {
    font-weight: normal;
    margin-right: 10px;
  }


  .theme-buttons button {
    background-color: var(--surface);
    color: var(--text);
    border: 1px solid rgba(255, 255, 255, 0.1); /* Softer border color */
    padding: 5px 10px;
    border-radius: 4px;
    cursor: pointer;
    font-size: 0.8em;
    transition: background-color 0.2s, color 0.2s, border-color 0.2s;
  }

  .theme-buttons button:hover {
    background-color: var(--primary);
    color: var(--text-dark);
    border-color: var(--primary);
  }

  .user-info h2 {
    margin-top: 0;
    margin-bottom: 0.25rem;
    color: var(--primary);
  }

  .user-info p {
    margin: 0.15rem 0;
    color: var(--text);
  }

  .content-wrapper {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    gap: 1rem;
  }

  .expense-tracker,
  .expenses-overview,
  .financial-goals {
    flex: 1;
    margin-bottom: 2rem;
    background-color: var(--surface);
    border-radius: 8px;
    padding: 1.5rem;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.2);
    max-width: 100%;
    color: var(--text);
  }

  .expense-tracker {
    flex: 3;
  }

  .expenses-overview {
    flex: 3;
  }

  .financial-goals {
    flex: 2;
  }

  .month-navigation {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 1rem;
  }

  .month-navigation button {
    background-color: var(--primary);
    color: var(--text);
    border: 1px solid rgba(255, 255, 255, 0.1); /* Softer border color */
    padding: 0.4rem 0.8rem;
    border-radius: 4px;
    cursor: pointer;
    transition: background-color 0.2s, color 0.2s, border-color 0.2s;
    font-size: 0.9em;
  }

  .month-navigation button:hover {
    background-color: var(--secondary);
  }

  .month-navigation button:disabled {
    background-color: var(--surface);
    color: rgba(255, 255, 255, 0.3);
    border-color: rgba(255, 255, 255, 0.1);
    cursor: not-allowed;
  }

  .sorting-options {
    display: flex;
    align-items: center;
    margin-bottom: 1rem;
    flex-wrap: wrap;
  }

  .sorting-options span {
    margin-right: 0.5rem;
    margin-bottom: 0.5rem;
    color: var(--text);
  }

  .sorting-options button {
    background-color: var(--surface);
    color: var(--text);
    border: 1px solid rgba(255, 255, 255, 0.1); /* Softer border color */
    padding: 0.4rem 0.8rem;
    border-radius: 4px;
    cursor: pointer;
    transition: background-color 0.2s, color 0.2s, border-color 0.2s;
    margin-right: 0.5rem;
    margin-bottom: 0.5rem;
    font-size: 0.9em;
  }

  .sorting-options button:hover {
    background-color: var(--primary);
    color: var(--text);
    border-color: var(--primary);
  }

  .sorting-options button.active {
    background-color: var(--primary);
    color: var(--text);
    border-color: var(--primary);
  }

  .note-section {
    margin-top: 1rem;
  }

  .note-section label {
    display: block;
    margin-bottom: 0.5rem;
    color: var(--text);
  }

  .note-section textarea {
    width: 100%;
    padding: 0.5rem;
    border: 1px solid var(--surface);
    border-radius: 4px;
    background-color: var(--background);
    color: var(--text);
    resize: vertical;
  }

  button {
    background-color: var(--primary);
    color: var(--text);
    border: none;
    padding: 0.4rem 0.8rem;
    border-radius: 4px;
    cursor: pointer;
    transition: background-color 0.2s;
    margin-top: 0.5rem;
    font-size: 0.9em;
  }

  button:hover {
    background-color: var(--secondary);
  }

  .overview-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
}

.overview-header h2 {
  margin: 0 0 0.1rem 0;
}

.left-section {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}

.right-section {
  align-self: flex-start;
}

  .overview-header select {
    background-color: var(--surface);
    color: var(--text);
    border: none;
    padding: 0.5rem 1rem;
    border-radius: 4px;
    margin-top: 0.5rem;
    cursor: pointer;
    border: 1px solid rgba(255, 255, 255, 0.1); /* Softer border color */
  }

  .goals-container {
    /* max-height: 400px; */
    /* overflow-y: auto; */
  }

  @media (max-width: 1200px) {
    .expense-tracker,
    .expenses-overview,
    .financial-goals {
      flex: 1 0 100%;
    }
  }

  @media (max-width: 768px) {
    .sorting-options {
      flex-wrap: wrap;
    }

    .sorting-options button {
      margin-bottom: 0.5rem;
    }
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

  .save-expenses-button {
    background-color: var(--primary);
    color: var(--text);
    border: none;
    padding: 0.5rem 1rem;
    border-radius: 4px;
    cursor: pointer;
    font-size: 1em;
    margin-top: 0.5rem;
    margin-bottom: 1rem;
    width: 100%;
    transition: background-color 0.2s, color 0.2s;
  }

  .save-expenses-button:hover {
    background-color: var(--secondary);
  }

  .current-month {
  margin: 0 0 0.5rem 0;
  color: var(--text);
  font-size: 1.1rem;
  font-weight: 500;
}
</style>
