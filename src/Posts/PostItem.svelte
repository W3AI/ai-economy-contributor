<script>
  import { createEventDispatcher } from "svelte";
  import posts from "./posts-store.js";
  import Button from "../UI/Button.svelte";
  import Badge from "../UI/Badge.svelte";
  import LoadingSpinner from "../UI/LoadingSpinner.svelte";

  export let id;
  export let title;
  export let tags;
  export let imageUrl;
  export let content;
  export let repoUrl;
  export let creator;
  export let isFav;

  let isLoading = false;

  const dispatch = createEventDispatcher();

  function toggleFavorite() {
    isLoading = true;
    fetch(`https://ai-economy.firebaseio.com/posts/${id}.json`, {
      method: "PATCH",
      body: JSON.stringify({ isFavorite: !isFav }),
      headers: { "Content-Type": "application/json" }
    })
      .then(res => {
        if (!res.ok) {
          throw new Error("An error occurred, please try again!");
        }
        isLoading = false;
        posts.toggleFavorite(id);
      })
      .catch(err => {
        isLoading = false;
        console.log(err);
      });
  }
</script>

<style>
  article {
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.26);
    border-radius: 5px;
    background: white;
    margin: 1rem;
  }

  header,
  .content,
  footer {
    padding: 1rem;
  }

  .image {
    width: 100%;
    height: 14rem;
  }

  .image img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  h1 {
    font-size: 1.25rem;
    margin: 0.5rem 0;
    font-family: "Roboto Slab", sans-serif;
  }

  h1.is-favorite {
    background: #01a129;
    color: white;
    padding: 0 0.5rem;
    border-radius: 5px;
  }

  h2 {
    font-size: 1rem;
    color: #808080;
    margin: 0.5rem 0;
  }

  p {
    font-size: 1.25rem;
    margin: 0;
  }

  div {
    text-align: right;
  }

  .content {
    height: 4rem;
  }

  code {
    color: #01a129;
    background-color: #eee;
    padding: 0 0.5rem;
  }

  @media (max-width: 375px) {
    h1 {
      font-size: 1rem;
    }
    h2 {
      font-size: 0.9rem;
    }

    p {
      font-size: 0.8rem;
    }
  }
</style>

<article>
  <header>
    <h1>
       {title}
      {#if isFav}
        <Badge>❤</Badge>
      {/if}
    </h1>
    <h2>{tags}</h2>
    <p>
      <code>{repoUrl}</code>
    </p>
  </header>
  <div class="image">
    <img src={imageUrl} alt={title} />
  </div>
  <div class="content">
    <p>{content}</p>
  </div>
  <footer>
    <Button mode="outline" type="button" on:click={() => dispatch('edit', id)}>
      Edit
    </Button>
    {#if isLoading}
      <!-- <LoadingSpinner /> -->
      <span>Changing...</span>
    {:else}
      <Button
        mode="outline"
        color={isFav ? null : 'success'}
        type="button"
        on:click={toggleFavorite}>
         {isFav ? 'Unfav' : 'Fav'}
      </Button>
    {/if}
    <Button type="button" on:click={() => dispatch('showdetails', id)}>
      Details
    </Button>
  </footer>
</article>
