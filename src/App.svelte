<script>
  let userObject = null;
  const userbase = window.userbase;
  const appId = "13874ccd-cb9c-478c-8997-3bff3fcbde32";
  let authPromise = userbase
    .init({ appId })
    .then(({ user }) => (userObject = user));

  let username;
  let password;

  const signIn = () =>
    (authPromise = userbase
      .signIn({ username, password })
      .then((user) => (userObject = user)));
  const signUp = () =>
    (authPromise = userbase
      .signUp({ username, password })
      .then((user) => (userObject = user)));

  const signOut = () =>
    (authPromise = userbase.signOut).then(() => (userObject = null));

  let todoPromise;
  let todos = [],
    newTodo = "";
  const databaseName = "todos";
  function changeHandler(items) {
    todos = items;
  }
  $: if (userObject)
    todoPromise = userbase.openDatabase({ databaseName, changeHandler });

  function addTodo() {
    todoPromise = userbase.insertItem({ databaseName, item: newTodo });
    newTodo = "";
  }

  function deleteTodo(itemId) {
    todoPromise = userbase.deleteItem({ databaseName, itemId });
  }

  function updateTodo(index) {
    const { item, itemId } = todos[index];
    todoPromise = userbase.updateItem({ databaseName, itemId, item });
  }
</script>

<div class="container">
  <div class="row">
    <main class="form-signin text-center">
      {#await authPromise}
        <h1>Loading...</h1>{:then _}
        {#if !userObject}
          <form class="d-md-block">
            <img
              class="mb-4"
              src="https://kayacuneyt.com/wp-content/uploads/2021/03/CuneytKayaRealSon.png"
              alt=""
              width="72"
              height="72"
            />
            <h1 class="h3 mb-3 fw-normal">Please sign in</h1>

            <div class="form-floating my-2">
              <input
                type="text"
                class="form-control"
                id="username"
                placeholder="yourusername"
                bind:value={username}
              />
              <label for="floatingInput">Username</label>
            </div>
            <div class="form-floating my-2">
              <input
                type="password"
                class="form-control"
                id="password"
                placeholder="Password"
                bind:value={password}
              />
              <label for="floatingPassword">Password</label>
            </div>

            <div class="checkbox mb-3">
              <label>
                <input type="checkbox" value="remember-me" /> Remember me
              </label>
            </div>
            <button
              on:click={signIn}
              type="button"
              class="w-100 btn btn-lg btn-primary my-2">Sign in</button
            >

            <button
              on:click={signUp}
              type="button"
              class="w-100 btn btn-lg btn-primary my-2">Sign up</button
            >

            <p class="mt-5 mb-3 text-muted">© 2017–2021</p>
          </form>
        {:else}
          <div class="container">
            <div class="row">
              <h1>Hi {userObject.username}, welcome on board!</h1>
              <br />
              <button
                on:click={signOut}
                type="button"
                class="btn btn-lg btn-primary my-2"
              >
                Sign Out
              </button>
              {#await todoPromise}
                <h2>Todos are loading</h2>
              {:then _}
                <div class="row">
                  <div class="form-floating my-2">
                    <input
                      type="text"
                      id="new-todo"
                      placeholder="Add your new task"
                      bind:value={newTodo}
                    />

                    <button
                      on:click={addTodo}
                      type="button"
                      class="btn btn-lg btn-primary my-2"
                    >
                      Add Todo
                    </button>
                  </div>
                </div>

                <div class="row">
                  {#each todos as { item, itemId }, index}
                    <ul class="list-group">
                      <li class="list-group-item">
                        <input
                          class="my-1"
                          type="text"
                          id=""
                          bind:value={todos[index].item}
                        />
                        <button
                          type="button"
                          on:click={() => deleteTodo(itemId)}
                          on:blur={() => updateTodo(index)}
                          class="btn btn-danger my-2">Delete</button
                        >
                      </li>
                    </ul>
                  {/each}
                </div>
              {:catch error}
                Error ! {error}
              {/await}
            </div>
          </div>
        {/if}
      {:catch error}<h1>Error!</h1>
        {error.message}{/await}
    </main>
  </div>
</div>

<style>
  .form-signin {
    width: 100%;
    max-width: 330px;
    padding: 15px;
    margin: auto;
    height: 90vh;
    justify-content: center;
    display: flex;
    align-items: center;
  }
</style>
