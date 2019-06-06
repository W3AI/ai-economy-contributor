<script>
  import posts from "./posts-store.js";
  import { createEventDispatcher } from "svelte";

  import TextInput from "../UI/TextInput.svelte";
  import Button from "../UI/Button.svelte";
  import Modal from "../UI/Modal.svelte";
  import { isEmpty, isValidEmail } from "../helpers/validation.js";

  export let id = null;

  let title = "eg: My contribution";
  let tags = "service, git, commit, change";
  // ToDo: Add url/links comp with a horizontal $ sign => hook/enzime facilitating cont. trade/progress
  let repoUrl = "https://github.com/W3AI/ai-society-server";
  let creator = "agile.team.366@w3ai.org";
  let content = "git commit -m '<message>'";
  let imageUrl =
    "https://tcp1pnet.files.wordpress.com/2019/05/2000-contributions.png";

  if (id) {
    const unsubscribe = posts.subscribe(items => {
      const selectedPost = items.find(i => i.id === id);
      title = selectedPost.title;
      tags = selectedPost.tags;
      repoUrl = selectedPost.repoUrl;
      creator = selectedPost.contactEmail;
      content = selectedPost.content;
      imageUrl = selectedPost.imageUrl;
    });

    unsubscribe();
  }

  const dispatch = createEventDispatcher();

  $: titleValid = !isEmpty(title);
  $: tagsValid = !isEmpty(tags);
  $: repoUrlValid = !isEmpty(repoUrl);
  $: contentValid = !isEmpty(content);
  $: imageUrlValid = !isEmpty(imageUrl);
  $: creatorValid = isValidEmail(creator);
  $: formIsValid =
    titleValid &&
    tagsValid &&
    repoUrlValid &&
    contentValid &&
    imageUrlValid &&
    creatorValid;

  function submitForm() {
    const postData = {
      title: title,
      tags: tags,
      content: content,
      imageUrl: imageUrl,
      contactEmail: creator,
      repoUrl: repoUrl
    };

    // posts.push(newPost);    // DOES NOT WORK IN SVELTE
    if (id) {
      fetch(`https://ai-economy.firebaseio.com/posts/${id}.json`, {
        method: "PATCH",
        body: JSON.stringify(postData),
        headers: { "Content-Type": "application/json" }
      })
        .then(res => {
          if (!res.ok) {
            throw new Error("An error occurred, please try again!");
          }
          posts.updatePost(id, postData); // update in local store after db
        })
        .catch(err => {
          console.log();
        });
    } else {
      fetch("https://ai-economy.firebaseio.com/posts.json", {
        method: "POST",
        body: JSON.stringify({ ...postData, isFavorite: false }),
        headers: { "Content-Type": "application/json" }
      })
        .then(res => {
          if (!res.ok) {
            throw new Error("An error occurred, please try again!");
          }
          return res.json();
        })
        .then(data => {
          // save in local posts-store - in browser
          posts.addPost({
            ...postData,
            isFavorite: false,
            id: data.name
          });
        })
        .catch(err => {
          console.log(err);
        });

      // Save to skills server
      fetch("http://localhost:8080/feed/post", {
        method: "POST",
        body: JSON.stringify({ ...postData, isFavorite: false }),
        headers: { "Content-Type": "application/json" }
      }).then(res => {
        if (!res.ok) {
          throw new Error("Skills server error, please try again!");
        }
        return res.json();
      })
      .catch(err => {
          console.log(err);
        });
    }
    dispatch("save");
  }

  function deletePost() {
    fetch(`https://ai-economy.firebaseio.com/posts/${id}.json`, {
      method: "DELETE"
    })
      .then(res => {
        if (!res.ok) {
          throw new Error("An error occurred, please try again!");
        }
        posts.removePost(id);
      })
      .catch(err => console.log(err));
    dispatch("save");
  }

  function cancel() {
    dispatch("cancel");
  }
</script>

<style>
  form {
    width: 100%;
  }
</style>

<Modal title="Edit" on:cancel>
  <form on:submit|preventDefault={submitForm}>
    <TextInput
      id="title"
      label="Message"
      valid={titleValid}
      validityMessage="Please enter a valid title."
      value={title}
      on:input={event => (title = event.target.value)} />
    <TextInput
      id="tags"
      label="Tags"
      valid={tagsValid}
      validityMessage="Please enter a valid tags."
      value={tags}
      on:input={event => (tags = event.target.value)} />
    <TextInput
      id="repoUrl"
      label="Repository URL"
      valid={repoUrlValid}
      validityMessage="Please enter a valid repository URL."
      value={repoUrl}
      on:input={event => (repoUrl = event.target.value)} />
    <TextInput
      id="imageUrl"
      label="Image URL"
      valid={imageUrlValid}
      validityMessage="Please enter a valid image URL."
      value={imageUrl}
      on:input={event => (imageUrl = event.target.value)} />
    <TextInput
      id="creator"
      label="Creator email"
      type="creator"
      valid={creatorValid}
      validityMessage="Please enter a valid email."
      value={creator}
      on:input={event => (creator = event.target.value)} />
    <TextInput
      id="content"
      label="Content"
      controlType="textarea"
      valid={contentValid}
      validityMessage="Please enter a valid content."
      value={content}
      on:input={event => (content = event.target.value)} />
  </form>
  <div slot="footer">
    <Button type="button" mode="outline" on:click={cancel}>Cancel</Button>
    <Button type="button" on:click={submitForm} disabled={!formIsValid}>
      Save
    </Button>
    {#if id}
      <Button type="button" on:click={deletePost}>Delete</Button>
    {/if}
  </div>
</Modal>
