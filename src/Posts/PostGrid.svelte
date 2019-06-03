<script>
  import { createEventDispatcher } from "svelte";
  import { scale } from "svelte/transition";
  import { flip } from "svelte/animate";
  import PostItem from "./PostItem.svelte";
  import PostFilter from "./PostFilter.svelte";
  import Button from "../UI/Button.svelte";

  export let posts;

  const dispatch = createEventDispatcher();

  let favsOnly = false;

  $: filteredPosts = favsOnly ? posts.filter(m => m.isFavorite) : posts;

  function setFilter(event) {
    favsOnly = event.detail === 1;
  }
</script>

<style>
  #posts {
    width: 100%;
    display: grid;
    grid-template-columns: 1fr;
    grid-gap: 1rem;
  }

  #post-controls {
    margin: 1rem;
    display: flex;
    justify-content: space-between;
  }

  #no-posts {
    margin: 1rem;
  }

  @media (max-width: 330px) {
    #post-controls {
      margin: .7rem;
    }
  }

  @media (min-width: 768px) {
    #posts {
      grid-template-columns: repeat(2, 1fr);
    }
  }
</style>

<section id="post-controls">
  <PostFilter on:select={setFilter} />

  <Button on:click={() => dispatch('add')}>+Friends</Button>
  <Button on:click={() => dispatch('add')}>+Service</Button>

</section>
{#if filteredPosts.length === 0}
  <p id="no-posts">No posts found, you can start adding some.</p>
{/if}
<section id="posts">
  {#each filteredPosts as post (post.id)}
    <div transition:scale animate:flip={{ duration: 300 }}>
      <PostItem
        id={post.id}
        title={post.title}
        subtitle={post.subtitle}
        description={post.description}
        imageUrl={post.imageUrl}
        email={post.contactEmail}
        address={post.address}
        isFav={post.isFavorite}
        on:showdetails
        on:edit />
    </div>
  {/each}
</section>
