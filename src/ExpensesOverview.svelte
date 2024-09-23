<script>
  import { onMount, afterUpdate, tick } from 'svelte';
  import Chart from 'chart.js/auto';
  import { 
    FaUtensils, FaHome, FaBolt, FaBus, FaFilm, FaMedkit, 
    FaGraduationCap, FaShoppingBag, FaPlane, FaEllipsisH,
    FaShieldAlt, FaPhoneAlt, FaShoppingCart, FaTshirt,
    FaCar, FaCut, FaCalendarAlt
  } from 'svelte-icons/fa';

  export let expenses = {};
  export let period = 'month';
  export let currentMonth;

  let barCanvas;
  let pieCanvas;
  let monthlyBarCanvas;
  let barChart;
  let pieChart;
  let monthlyBarChart;

  let colorMap = new Map();
  let hueStep = 360 / 12; // Divide the color wheel into 12 segments
  let hueOffset = Math.random() * 360; // Random starting point on the color wheel
  let data;
  let chartTitle;
  let topExpenses;
  let monthlyData;
  let topMonths = [];
  let averageDailySpend = 0;
  let categoryGrowthRates = [];
  let yearlyAverageDailySpend = 0;


   /**
   * Generates a unique color for each category.
   * @param {string} category - The expense category
   * @returns {string} HSL color string
   */
  function generateColor(category) {
  if (colorMap.has(category)) {
    return colorMap.get(category);
  }
  
  // AI-generated: This function uses a clever algorithm to create visually distinct colors
  let hue = (hueOffset + colorMap.size * hueStep) % 360;
  let saturation = 50 + Math.random() * 20; // Reduced from 70-90% to 50-70%
  let lightness = 65 + Math.random() * 10; // Increased from 45-55% to 65-75%
  
  let newColor = `hsl(${hue}, ${saturation}%, ${lightness}%)`;
  colorMap.set(category, newColor);
  return newColor;
}

 /**
   * Gets an array of colors for the given categories.
   * @param {Array<string>} categories - Array of expense categories
   * @returns {Array<string>} Array of color strings
   */
  function getColors(categories) {
    return categories.map(category => generateColor(category));
  }

  $: {
  console.log('Expenses data changed:', expenses);
  console.log('Current month:', currentMonth);
  console.log('Period:', period);
  data = processData(expenses, period, currentMonth);
  chartTitle = getChartTitle(period, currentMonth);
  topExpenses = getTopExpenses(expenses, period, currentMonth);
  monthlyData = getMonthlyData(expenses, currentMonth);
  if (period === 'year') {
    topMonths = getTopMonths(expenses, currentMonth);
    yearlyAverageDailySpend = calculateYearlyAverageDailySpend(expenses, currentMonth);
  } else {
    topMonths = [];
  }
  if (period === 'month') {
    calculateAverageDailySpend();
    calculateCategoryGrowthRates();
  }
  tick().then(() => {
    updateCharts();
  });
}

 /**
   * Generates the title for the charts based on the period and current month.
   * @param {string} period - Either 'month' or 'year'
   * @param {string} currentMonth - Current month in 'YYYY-MM' format
   * @returns {string} Chart title
   */
function getChartTitle(period, currentMonth) {
  if (period === 'month') {
    const [year, month] = currentMonth.split('-');
    const date = new Date(year, parseInt(month) - 1, 1);
    return date.toLocaleString('default', { month: 'long', year: 'numeric' });
  } else {
    return new Date(currentMonth).getFullYear().toString();
  }
}

 /**
   * Processes the expenses data for chart display.
   * @param {Object} expenses - Expenses data object
   * @param {string} period - Either 'month' or 'year'
   * @param {string} currentMonth - Current month in 'YYYY-MM' format
   * @returns {Object} Processed data for charts
   */
function processData(expenses, period, currentMonth) {
  console.log('Processing data for charts:', expenses);
  console.log('Current month:', currentMonth);
  console.log('Period:', period);

  if (period === 'month') {
    const monthData = expenses[currentMonth].categories;
    console.log('Month data:', monthData);
    const labels = Object.keys(monthData);
    console.log('Labels:', labels);
    return {
      labels: labels,
      datasets: [{
        label: `Expenses for ${getChartTitle(period, currentMonth)}`,
        data: Object.values(monthData).map(item => item.value),
        backgroundColor: getColors(labels),
      }]
    };
  } else if (period === 'year') {
    const year = new Date(currentMonth).getFullYear();
    const yearData = Object.entries(expenses)
      .filter(([month]) => new Date(month).getFullYear() === year)
      .reduce((acc, [_, data]) => {
        Object.entries(data.categories).forEach(([category, categoryData]) => {
          if (!acc[category]) acc[category] = 0;
          acc[category] += categoryData.value;
        });
        return acc;
      }, {});
    const labels = Object.keys(yearData);
    return {
      labels: labels,
      datasets: [{
        label: `Yearly Expenses for ${year}`,
        data: Object.values(yearData),
        backgroundColor: getColors(labels),
      }]
    };
  }
}

function getMonthlyData(expenses, currentMonth) {
  const year = new Date(currentMonth).getFullYear();
  const monthlyTotals = Array(12).fill(0);
  
  Object.entries(expenses).forEach(([month, data]) => {
    const date = new Date(month);
    if (date.getFullYear() === year) {
      const monthIndex = date.getMonth();
      const total = Object.values(data.categories).reduce((sum, category) => sum + category.value, 0);
      monthlyTotals[monthIndex] = total;
    }
  });

  return {
    labels: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'],
    datasets: [{
      label: `Monthly Totals for ${year}`,
      data: monthlyTotals,
      backgroundColor: 'rgba(75, 192, 192, 0.6)',
      borderColor: 'rgba(75, 192, 192, 1)',
      borderWidth: 1
    }],
    monthlyTotals // Return the raw monthly totals
  };
}

  /**
   * Gets the top 5 expenses for the given period and current month.
   * @param {Object} expenses - Expenses data object
   * @param {string} period - Either 'month' or 'year'
   * @param {string} currentMonth - Current month in 'YYYY-MM' format
   * @returns {Array<Object>} Array of top expenses
   */
  function getTopExpenses(expenses, period, currentMonth) {
    console.log('Processing top expenses:', expenses);
  let expenseData;
  let totalExpenses;

  if (period === 'month') {
    expenseData = expenses[currentMonth].categories;
    totalExpenses = Object.values(expenseData).reduce((sum, category) => sum + category.value, 0);
  } else {
    const year = new Date(currentMonth).getFullYear();
    expenseData = Object.entries(expenses)
      .filter(([month]) => new Date(month).getFullYear() === year)
      .reduce((acc, [_, data]) => {
        Object.entries(data.categories).forEach(([category, categoryData]) => {
          if (!acc[category]) acc[category] = { value: 0, max: 0, icon: categoryData.icon };
          acc[category].value += categoryData.value;
          acc[category].max += categoryData.max;
        });
        return acc;
      }, {});
    totalExpenses = Object.values(expenseData).reduce((sum, category) => sum + category.value, 0);
  }

  return Object.entries(expenseData)
    .map(([category, data]) => ({
      category,
      amount: data.value,
      percentage: (data.value / totalExpenses) * 100,
      icon: data.icon
    }))
    .sort((a, b) => b.amount - a.amount)
    .slice(0, 5);
}

  /**
   * Gets the top 5 months for the given expenses and current month.
   * @param {Object} expenses - Expenses data object
   * @param {string} currentMonth - Current month in 'YYYY-MM' format
   * @returns {Array<Object>} Array of top months
   */
  function getTopMonths(expenses, currentMonth) {
    const year = new Date(currentMonth).getFullYear();
    const monthlyTotals = Object.entries(expenses)
      .filter(([month]) => new Date(month).getFullYear() === year)
      .map(([month, data]) => {
        const total = Object.values(data.categories).reduce((sum, category) => sum + category.value, 0);
        const daysInMonth = new Date(new Date(month).getFullYear(), new Date(month).getMonth() + 1, 0).getDate();
        const averageDailySpend = total / daysInMonth;
        return {
          month,
          total,
          averageDailySpend
        };
      })
      .sort((a, b) => b.total - a.total)
      .slice(0, 5);

    return monthlyTotals.map(({ month, total, averageDailySpend }) => ({
      month: new Date(month).toLocaleString('default', { month: 'long' }),
      amount: total,
      averageDailySpend
    }));
  }

  /**
   * Calculates the average daily spend for the given expenses and current month.
   * @param {Object} expenses - Expenses data object
   * @param {string} currentMonth - Current month in 'YYYY-MM' format
   * @returns {number} Average daily spend
   */
  function calculateYearlyAverageDailySpend(expenses, currentMonth) {
    const year = new Date(currentMonth).getFullYear();
    const yearlyTotal = Object.entries(expenses)
      .filter(([month]) => new Date(month).getFullYear() === year)
      .reduce((sum, [_, data]) => sum + Object.values(data.categories).reduce((catSum, category) => catSum + category.value, 0), 0);
    
    const daysInYear = (year % 4 === 0 && year % 100 !== 0) || year % 400 === 0 ? 366 : 365;
    return yearlyTotal / daysInYear;
  }

    /**
   * Updates all charts with the latest data.
   */
  // Use of AI: To help get the label rotation working properly when adding more categories
  function updateCharts() {
  if (barChart && pieChart) {
    barChart.data = data;
    pieChart.data = data;
    
    // Update chart options to show all labels
    barChart.options = {
      responsive: true,
      maintainAspectRatio: false,
      scales: {
        x: {
          grid: {
            display: false
          },
          ticks: {
            autoSkip: false, // This ensures all labels are shown
            maxRotation: 45, // Rotate labels if they don't fit
            minRotation: 45,
            font: {
              size: 10 // Reduce font size to fit more labels
            }
          }
        },
        y: {
          beginAtZero: true,
          ticks: {
            callback: function(value) {
              return '$' + value;
            }
          },
          grid: {
            color: 'rgba(255, 255, 255, 0.1)' // Lighter grid lines
          }
        }
      },
      plugins: {
        legend: {
          display: false,
        },
        tooltip: {
          callbacks: {
            label: function(context) {
              let label = context.dataset.label || '';
              if (label) {
                label += ': ';
              }
              if (context.parsed.y !== null) {
                label += new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(context.parsed.y);
              }
              return label;
            }
          }
        }
      }
    };

    barChart.update();
    pieChart.update();
  }

  if (period === 'year') {
    if (!monthlyBarChart && monthlyBarCanvas) {
      monthlyBarChart = new Chart(monthlyBarCanvas, {
        type: 'bar',
        data: monthlyData,
        options: {
          responsive: true,
          maintainAspectRatio: false,
          layout: {
            padding: {
              left: 10,
              right: 10,
              top: 10,
              bottom: 20 // Increased bottom padding
            }
          },
          scales: {
            x: {
              grid: {
                display: false
              }
            },
            y: {
              beginAtZero: true,
              ticks: {
                callback: function(value) {
                  return '$' + value;
                }
              },
              grid: {
                color: 'rgba(255, 255, 255, 0.1)' // Lighter grid lines
              }
            }
          },
          plugins: {
            legend: {
              display: false,
            },
            tooltip: {
              callbacks: {
                label: function(context) {
                  let label = context.dataset.label || '';
                  if (label) {
                    label += ': ';
                  }
                  if (context.parsed.y !== null) {
                    label += new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(context.parsed.y);
                  }
                  return label;
                }
              }
            }
          }
        }
      });
    } else if (monthlyBarChart) {
      monthlyBarChart.data = monthlyData;
      monthlyBarChart.update();
    }
  } else {
    if (monthlyBarChart) {
      monthlyBarChart.destroy();
      monthlyBarChart = null;
    }
  }
}

  /**
   * Initializes the charts on mount.
   */
  onMount(() => {
    barChart = new Chart(barCanvas, {
      type: 'bar',
    data: data,
    options: {
      responsive: true,
      maintainAspectRatio: false,
      scales: {
        x: {
          grid: {
            display: false
          },
          ticks: {
            autoSkip: false,
            maxRotation: 45,
            minRotation: 45,
            font: {
              size: 10
            }
          }
        },
        y: {
          beginAtZero: true,
          ticks: {
            callback: function(value) {
              return '$' + value;
            }
          }
        }
      },
      plugins: {
        legend: {
          display: false,
        },
        tooltip: {
          callbacks: {
            label: function(context) {
              let label = context.label || '';
              if (label) {
                label += ': ';
              }
              if (context.parsed.y !== null) {
                label += new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(context.parsed.y);
              }
              return label;
            }
          }
        }
      }
    }
  });

  pieChart = new Chart(pieCanvas, {
    type: 'pie',
    data: data,
    options: {
      responsive: true,
      maintainAspectRatio: false,
      layout: {
        padding: {
          left: 10,
          right: 10,
          top: 10,
          bottom: 10
        }
      },
      plugins: {
        legend: {
          position: 'bottom',
          align: 'center',
          labels: {
            boxWidth: 15,
            padding: 15,
            font: {
              size: 14
            }
          }
        },
        tooltip: {
          callbacks: {
            label: function(context) {
              let label = context.label || '';
              let value = context.parsed;
              let sum = context.dataset.data.reduce((a, b) => a + b, 0);
              let percentage = Math.round((value / sum) * 100);
              return `${label}: ${new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(value)} (${percentage}%)`;
            }
          }
        }
      }
    }
  });

  updateCharts();

  return () => {
    if (barChart) barChart.destroy();
    if (pieChart) pieChart.destroy();
    if (monthlyBarChart) monthlyBarChart.destroy();
  };
});

  afterUpdate(() => {
    updateCharts();
  });

  /**
   * Gets the icon component for the given category.
   * @param {string} category - Expense category
   * @returns {Component} Icon component
   */
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

  /**
   * Calculates the average daily spend for the given expenses and current month.
   */
  function calculateAverageDailySpend() {
    const currentMonthExpenses = expenses[currentMonth]?.categories || {};
    const totalSpent = Object.values(currentMonthExpenses).reduce((sum, category) => sum + category.value, 0);
    const daysInMonth = new Date(currentMonth.slice(0, 4), currentMonth.slice(5, 7), 0).getDate();
    averageDailySpend = totalSpent / daysInMonth;
  }

  /**
   * Calculates the growth rates for each category compared to the previous month.
   */
  function calculateCategoryGrowthRates() {
    const currentMonthExpenses = expenses[currentMonth]?.categories || {};
    const previousMonth = getPreviousMonth(currentMonth);
  const previousMonthExpenses = expenses[previousMonth]?.categories || {};

  categoryGrowthRates = Object.keys(currentMonthExpenses).map(category => {
    const currentValue = currentMonthExpenses[category].value;
    const previousValue = previousMonthExpenses[category]?.value || 0;
    const difference = currentValue - previousValue;
    const growthRate = previousValue === 0 ? 100 : ((difference / previousValue) * 100);
    const amountSpentSign = difference >= 0 ? `+$${Math.abs(difference).toFixed(2)}` : `-$${Math.abs(difference).toFixed(2)}`;
    return { category, growthRate, amountSpent: amountSpentSign };
  });

    categoryGrowthRates.sort((a, b) => Math.abs(b.growthRate) - Math.abs(a.growthRate));
  }

  /**
   * Gets the previous month based on the given current month.
   * @param {string} currentMonth - Current month in 'YYYY-MM' format
   * @returns {string} Previous month in 'YYYY-MM' format
   */
  function getPreviousMonth(currentMonth) {
    const [year, month] = currentMonth.split('-');
    const previousDate = new Date(year, month - 2, 1);
    return `${previousDate.getFullYear()}-${String(previousDate.getMonth() + 1).padStart(2, '0')}`;
  }
</script>


<div class="charts-container">
  <div class="chart-wrapper pie-chart-wrapper">
    <canvas bind:this={pieCanvas}></canvas>
  </div>
  <div class="chart-wrapper bar-chart-wrapper">
    <canvas bind:this={barCanvas}></canvas>
  </div>
  {#if period === 'year'}
    <div class="chart-wrapper monthly-bar-chart-wrapper">
      <h4>Monthly Totals for {new Date(currentMonth).getFullYear()}</h4>
      <canvas bind:this={monthlyBarCanvas}></canvas>
    </div>
  {/if}
</div>

<div class="top-expenses">
  <h3>Top 5 Biggest Expenses for {chartTitle}</h3>
  <ul>
    {#each topExpenses as expense}
      <li>
        <div class="expense-icon">
          <svelte:component this={getIconComponent(expense.category)} />
        </div>
        <span class="category" title={expense.category}>{expense.category}</span>
        <span class="amount">${expense.amount.toFixed(2)}</span>
        <span class="percentage">({expense.percentage.toFixed(1)}%)</span>
      </li>
    {/each}
  </ul>
</div>

{#if period === 'year'}
  <div class="monthly-stats">
    <div class="stat-item">
      <h4>Yearly Average Daily Spend</h4>
      <p>${yearlyAverageDailySpend.toFixed(2)}</p>
    </div>
  </div>
  <div class="top-expenses">
    <h3>Top 5 Most Expensive Months for {new Date(currentMonth).getFullYear()}</h3>
    <ul>
      {#each topMonths as month}
        <li>
          <div class="expense-icon">
            <svelte:component this={FaCalendarAlt} />
          </div>
          <span class="category" title={month.month}>{month.month}</span>
          <span class="amount">${month.amount.toFixed(2)}</span>
          <span class="daily-average">(Avg. Daily: ${month.averageDailySpend.toFixed(2)})</span>
        </li>
      {/each}
    </ul>
  </div>
  <!-- <YearlyStats {monthlyData}={monthlyData.monthlyTotals} /> -->
{/if}

{#if period === 'month'}
  <div class="monthly-stats">
    <div class="stat-item">
      <h4>Average Daily Spend</h4>
      <p>${averageDailySpend.toFixed(2)}</p>
    </div>
    
    <div class="stat-item">
      <h4>Top Category Growth Rates (vs Last Month)</h4>
      <ul>
        {#each categoryGrowthRates.slice(0, 5) as { category, growthRate, amountSpent }}
          <li>
            <span class="category">{category}</span>
            <span class={growthRate > 0 ? 'positive' : 'negative'}>
              {growthRate > 0 ? '+' : ''}{growthRate.toFixed(2)}% &nbsp;{amountSpent}
            </span>
          </li>
        {/each}
      </ul>
    </div>
  </div>
{/if}

<style>
  .charts-container {
    display: flex;
    flex-direction: column;
    gap: 12px;
  }

  .chart-wrapper {
    width: 100%;
    background-color: #2c2c2c;
    border-radius: 8px;
    padding: 1px;
    box-sizing: border-box;
  }

  .pie-chart-wrapper canvas {
    height: 500px;
    width: 100%;
    position: relative;
  }

  .bar-chart-wrapper {
    height: 400px;
  }

  .monthly-bar-chart-wrapper {
    height: 400px;
    margin-bottom: 4px; /* Increased from 20px to 24px */
    padding-bottom: 40px;
  }

  .monthly-bar-chart-wrapper h4 {
    text-align: center;
    margin-bottom: 10px;
  }

  .top-expenses,
  .monthly-stats {
    margin-top: 24px;
    margin-bottom: 24px;
    width: 100%;
    box-sizing: border-box;
  }

  .top-expenses {
    background-color: #333;
    padding: 16px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    border: 1px solid rgba(255, 255, 255, 0.1);
  }

  .monthly-stats {
    display: flex;
    flex-direction: column;
    gap: 1rem;
    background-color: transparent;
    padding: 0;
  }

  .stat-item {
    background-color: #333;
    padding: 16px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    border: 1px solid rgba(255, 255, 255, 0.1);
    width: 100%;
    box-sizing: border-box;
  }

  .top-expenses h3 {
    margin-top: 0;
    margin-bottom: 12px;
    color: #e0e0e0;
  }

  .top-expenses ul {
    list-style-type: none;
    padding: 0;
    margin: 0;
  }

  .top-expenses li {
    display: flex;
    align-items: center;
    margin-bottom: 8px;
    font-size: 0.9em;
    white-space: nowrap;
    overflow: hidden;
  }

  .expense-icon {
    flex-shrink: 0;
    width: 20px;
    height: 20px;
    margin-right: 10px;
    color:  var(--primary);
  }

  .expense-icon :global(svg) {
    width: 100%;
    height: 100%;
  }

  .top-expenses .category {
    font-weight: bold;
    margin-right: 10px;
    max-width: 120px;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .top-expenses .amount {
    color:  var(--primary);
    margin-right: 10px;
  }

  .top-expenses .percentage {
    color: #FFA500;
  }

  .monthly-stats {
    display: flex;
    flex-direction: column;
    gap: 1rem;
    margin-top: 1rem;
    margin-bottom: 1rem;  /* Add some bottom margin */
    margin-left: 8x;  /* Add left margin to match the right */
    margin-right: 8px; /* Keep the right margin */
    width: calc(100% - 2px); /* Adjust width to account for both margins */
  }

  .stat-item h4 {
    margin-top: 0;
    margin-bottom: 0.5rem;
    color: var(--primary);
    font-size: 1.1em;
  }

  .stat-item p {
    font-size: 1.4em;
    font-weight: bold;
    margin: 0;
    color: var(--text);
  }

  .stat-item ul {
    list-style-type: none;
    padding: 0;
    margin: 0;
  }

  .stat-item li {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 0.5rem;
    font-size: 1em;
  }

  .stat-item .category {
    font-weight: bold;
    max-width: 60%;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .positive {
    color: #4CAF50;
  }

  .negative {
    color: #FF4136;
  }

  .top-expenses .daily-average {
    color: #FFA500;
    font-size: 0.9em;
    margin-left: 10px;
  }
</style>