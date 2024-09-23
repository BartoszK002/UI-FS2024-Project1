<script>
  import { createEventDispatcher } from 'svelte';

  const dispatch = createEventDispatcher();
  let newCategory = '';

  function handleSubmit() {
    if (newCategory.trim()) {
      dispatch('addCategory', newCategory.trim());
      newCategory = '';
    }
  }
</script>

<form on:submit|preventDefault={handleSubmit}>
  <input bind:value={newCategory} placeholder="New category name" />
  <button type="submit">Add Category</button>
</form>

<style>
  form {
    display: flex;
    margin-bottom: 1rem;
  }

  input {
    flex-grow: 1;
    padding: 0.5rem;
    border: 1px solid #444;
    border-radius: 4px 0 0 4px;
    background-color: #333;
    color: #e0e0e0;
  }

  button {
    padding: 0.4rem 0.8rem;
    background-color: var(--primary);
    color: white;
    border: none;
    border-radius: 0 4px 4px 0;
    cursor: pointer;
    transition: background-color 0.3s;
    font-size: 0.9em;
  }

  button:hover {
    background-color: #45a049;
  }
</style>