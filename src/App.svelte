<script>
  import posts from "./Posts/posts-store.js";
  import Header from "./UI/Header.svelte";
  import PostGrid from "./Posts/PostGrid.svelte";
  import TextInput from "./UI/TextInput.svelte";
  import Button from "./UI/Button.svelte";
  import EditPost from "./Posts/EditPost.svelte";
  import PostDetail from "./Posts/PostDetail.svelte";
  import LoadingSpinner from "./UI/LoadingSpinner.svelte";
  import Error from "./UI/Error.svelte";

  let editMode;
  let editedId;
  let page = "overview";
  let pageData = {};
  let isLoading = true;
  let error;

  fetch("https://ai-economy.firebaseio.com/posts.json")
    .then(res => {
      if (!res.ok) {
        throw new error("Fetching posts failed, please try again later!");
      }
      return res.json();
    })
    .then(data => {
      const loadedPosts = [];
      for (const key in data) {
        loadedPosts.push({
          ...data[key],
          id: key
        });
      }
      isLoading = false;
      posts.setPosts(loadedPosts.reverse());
    })
    .catch(err => {
      error = err;
      isLoading = false;
      console.log(err);
    });

  function savedPost(event) {
    editMode = null;
    editedId = null;
  }

  function cancelEdit() {
    editMode = null;
    editedId = null;
  }

  function showDetails(event) {
    page = "details";
    pageData.id = event.detail;
  }

  function closeDetails() {
    page = "overview";
    pageData = {};
  }

  function startEdit(event) {
    editMode = "edit";
    editedId = event.detail;
  }

  function clearError() {
    error = null;
  }
</script>

<style>
  main {
    margin-top: 5rem;
  }
</style>

{#if error}
  <Error message={error.message} on:cancel={clearError} />
{/if}

<Header />

<main>
  {#if page === 'overview'}
    {#if editMode === 'edit'}
      <EditPost id={editedId} on:save={savedPost} on:cancel={cancelEdit} />
    {/if}
    {#if isLoading}
      <LoadingSpinner />
    {:else}
      <PostGrid
        posts={$posts}
        on:showdetails={showDetails}
        on:edit={startEdit}
        on:add={() => {
          editMode = 'edit';
        }} />
    {/if}
  {:else}
    <PostDetail id={pageData.id} on:close={closeDetails} />
  {/if}
</main>
