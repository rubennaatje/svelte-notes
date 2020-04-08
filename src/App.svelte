<script>
  import { slide } from "svelte/transition";
  import { elasticInOut } from "svelte/easing";
  import { onMount } from "svelte";
  let todos = [];
  let apiTodos = [];
  let input = "";

  async function addTodo() {
    if (input) {
      const todo = {
        id: Date.now(),
        title: input,
        content: "",
        done: false
      };
      await fetch(`http://localhost:3000/notes`, {
        method: "POST",
        body: JSON.stringify(todo),
        headers: {
          "Content-Type": "application/json"
        }
      })
        .then(r => r.json())
        .then(data => {
          fetchTodos();
          console.log(apiTodos);
        });
    }
    input = "";
  }

  onMount(() => fetchTodos());

  async function fetchTodos() {
    await fetch(`http://localhost:3000/notes`)
      .then(r => r.json())
      .then(data => {
        apiTodos = data.notes;
        console.log(apiTodos);
      });
  }

  async function updateToDo(id) {
    // Find the correct to-do item.
    let todoSend = apiTodos.find(todo => todo.id === id);
    todoSend.done = !todoSend.done;
    if (todoSend.done == undefined) {
      todoSend.done = true;
    }
    console.log(id);
    console.log(todoSend);
    fetch("http://localhost:3000/notes", {
      method: "PUT",
      body: JSON.stringify(todoSend),
      headers: {
        "Content-Type": "application/json"
      }
    })
      .then(res => res.json())
      .then(data => {
        fetchTodos();
      })
      .catch(console.log);
  }

  async function removeToDo(id) {
    fetch("http://localhost:3000/notes/" + id, { method: "DELETE" })
      .then(res => res.json())
      .then(data => {
        fetchTodos();
      })
      .catch(console.log);
  }
</script>

<svelte:head>
  <link
    rel="stylesheet"
    type="text/css"
    href="https://cdn.jsdelivr.net/npm/bulma@0.8.0/css/bulma.min.css" />
  <script src="https://use.fontawesome.com/releases/v5.3.1/js/all.js">

  </script>
</svelte:head>

<main class="container is-fluid">
  <div class="columns is-centered is-vcentered is-mobile">
    <div class="column is-narrow" style="width: 70%">
      <h1 class="has-text-centered title">Svelte TODO</h1>
      <form
        class="field has-addons"
        style="justify-content: center"
        on:submit|preventDefault={addTodo}>
        <div class="control">
          <input
            bind:value={input}
            class="input"
            type="text"
            placeholder="TODO" />
        </div>
        <div class="control">
          <button class="button is-primary">
            <span class="icon is-small">
              <i class="fas fa-plus" />
            </span>
          </button>
        </div>
      </form>
      <ul class:list={apiTodos.length > 0}>
        {#each apiTodos as todo (todo.id)}
          <li
            class="list-item"
            transition:slide={{ duration: 300, easing: elasticInOut }}>
            <div class="is-flex" style="align-items: center">
              <span
                class="is-pulled-left {todo.done ? '' : 'has-text-grey-lighter'}">
                {todo.title}
              </span>
              <div style="flex: 1" />
              <button
                class="button is-text is-pulled-right is-small"
                on:click={() => updateToDo(todo.id)}>
                <span class="icon">
                  <i class="fas fa-check" />
                </span>
              </button>
              <button
                class="button is-text is-pulled-right is-small"
                on:click={() => removeToDo(todo.id)}>
                <span class="icon">
                  <i class="fas fa-trash" />
                </span>
              </button>
            </div>
          </li>
        {:else}
          <li
            class="has-text-centered"
            transition:slide={{ delay: 600, duration: 300, easing: elasticInOut }}>
            Nothing here!
          </li>
        {/each}
      </ul>
    </div>
  </div>
</main>
