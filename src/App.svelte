<script>
  import { onMount, tick } from 'svelte';
  import { fly } from 'svelte/transition';
  import { v4 as uuid } from 'uuid';
  import TodoList from './lib/TodoList.svelte';
  import spin from './lib/transitions/spin';

  let todoList;
  let showList = true;

  let todos = null;
  let error = null;
  let isLoading = false;
  let isAdding = false;
  let scrollOnAdd = 'top';
  let disableAdding;
  let disabledItems = [];

  onMount(() => {
    loadTodos();
  });

  async function loadTodos() {
    isLoading = true;
    await fetch('https://jsonplaceholder.typicode.com/todos?_limit=10').then(async (response) => {
      if (response.ok) {
        todos = await response.json();
      } else {
        error = 'An error has occurred';
      }
    });
    isLoading = false;
  }

  async function handleAddTodo(event) {
    event.preventDefault();
    isAdding = true;
    await fetch('https://jsonplaceholder.typicode.com/todos', {
      method: 'POST',
      body: JSON.stringify({
        title: event.detail.title,
        completed: false
      }),
      headers: {
        'Content-type': 'application/json; charset=UTF-8'
      }
    }).then(async (response) => {
      if (response.ok) {
        const todo = await response.json();
        todos = [{ ...todo, id: uuid() }, ...todos];
        todoList.clearInput();
      } else {
        alert('An error has occurred.');
      }
    });
    isAdding = false;
    await tick();
    todoList.focusInput();
  }

  async function handleRemoveTodo(event) {
    const id = event.detail.id;
    if (disabledItems.includes(id)) return;
    disabledItems = [...disabledItems, id];
    await fetch(`https://jsonplaceholder.typicode.com/todos/${id}`, {
      method: 'DELETE'
    }).then((response) => {
      if (response.ok) {
        todos = todos.filter((t) => t.id !== event.detail.id);
      } else {
        alert('An error has occurred.');
      }
    });
    disabledItems = disabledItems.filter((itemId) => itemId !== id);
  }

  async function handleToggleTodo(event) {
    const id = event.detail.id;
    const value = event.detail.value;
    if (disabledItems.includes(id)) return;
    disabledItems = [...disabledItems, id];
    await fetch(`https://jsonplaceholder.typicode.com/todos/${id}`, {
      method: 'PATCH',
      body: JSON.stringify({
        completed: value
      }),
      headers: {
        'Content-type': 'application/json; charset=UTF-8'
      }
    }).then(async (response) => {
      if (response.ok) {
        const updatedTodo = await response.json();
        todos = todos.map((todo) => {
          if (todo.id === id) {
            return updatedTodo;
          }
          return { ...todo };
        });
      } else {
        alert('An error has occurred.');
      }
    });

    disabledItems = disabledItems.filter((itemId) => itemId !== id);
  }
</script>

<label>
  <input type="checkbox" bind:checked={showList} />
  Show/Hide list
</label>
{#if showList}
  <div transition:spin={{ spin: 1, duration: 4000 }} style:max-width="400px">
    <TodoList
      {todos}
      {error}
      {isLoading}
      {disabledItems}
      disableAdding={isAdding}
      scrollOnAdd="top"
      bind:this={todoList}
      on:addtodo={handleAddTodo}
      on:removetodo={handleRemoveTodo}
      on:toggletodo={handleToggleTodo}
    />
  </div>
  {#if todos}
    <p>
      Number of todos:
      {#key todos.length}
        <span style:display="inline-block" in:fly|local={{ y: -10 }}>{todos.length}</span>
      {/key}
    </p>
  {/if}
{/if}

<style>
</style>
