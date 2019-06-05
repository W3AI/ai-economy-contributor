<script>
  import posts from "./posts-store.js";
  import { createEventDispatcher } from "svelte";

  import TextInput from "../UI/TextInput.svelte";
  import Button from "../UI/Button.svelte";
  import Modal from "../UI/Modal.svelte";
  import { isEmpty, isValidEmail } from "../helpers/validation.js";

  export let id = null;

  let title = "eg: Commit Contribution";
  let tags = "git commit command";
  let contribution = "git commit -m '<contribution>'";
  let email = "agile.team.366@w3ai.org";
  let content = "Commit contributions to the project.";
  let imageUrl =
    "https://tcp1pnet.files.wordpress.com/2019/05/2000-contributions.png";

  if (id) {
    const unsubscribe = posts.subscribe(items => {
      const selectedPost = items.find(i => i.id === id);
      title = selectedPost.title;
      tags = selectedPost.tags;
      contribution = selectedPost.contribution;
      email = selectedPost.contactEmail;
      content = selectedPost.content;
      imageUrl = selectedPost.imageUrl;
    });

    unsubscribe();
  }

  const dispatch = createEventDispatcher();

  $: titleValid = !isEmpty(title);
  $: tagsValid = !isEmpty(tags);
  $: contributionValid = !isEmpty(contribution);
  $: contentValid = !isEmpty(content);
  $: imageUrlValid = !isEmpty(imageUrl);
  $: emailValid = isValidEmail(email);
  $: formIsValid =
    titleValid &&
    tagsValid &&
    contributionValid &&
    contentValid &&
    imageUrlValid &&
    emailValid;

  function submitForm() {
    const postData = {
      title: title,
      tags: tags,
      content: content,
      imageUrl: imageUrl,
      contactEmail: email,
      contribution: contribution
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
          posts.addPost({
            ...postData,
            isFavorite: false,
            id: data.name
          });
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
      id="contribution"
      label="Contribution"
      valid={contributionValid}
      validityMessage="Please enter a valid contribution."
      value={contribution}
      on:input={event => (contribution = event.target.value)} />
    <TextInput
      id="imageUrl"
      label="Image URL"
      valid={imageUrlValid}
      validityMessage="Please enter a valid image url."
      value={imageUrl}
      on:input={event => (imageUrl = event.target.value)} />
    <TextInput
      id="email"
      label="E-Mail"
      type="email"
      valid={emailValid}
      validityMessage="Please enter a valid email contribution."
      value={email}
      on:input={event => (email = event.target.value)} />
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
