<script>
  import posts from "./posts-store.js";
  import { createEventDispatcher } from "svelte";

  import TextInput from "../UI/TextInput.svelte";
  import Button from "../UI/Button.svelte";
  import Modal from "../UI/Modal.svelte";
  import { isEmpty, isValidEmail } from "../helpers/validation.js";

  export let id = null;

  let title = "eg: Servant Lead";
  let subtitle = "Team Coordinator";
  let contribution = "12 Servants Ave, New Haven";
  let email = "servant.team.60@w3ai.net";
  let description = "Help teams plan projects and schedule services.";
  let imageUrl =
    "https://upload.wikimedia.org/wikipedia/commons/thumb/a/af/McDonald%27s_open_24_hours_banners%2C_Orchard_Road%2C_Singapore_-_20060313.jpg/800px-McDonald%27s_open_24_hours_banners%2C_Orchard_Road%2C_Singapore_-_20060313.jpg";

  if (id) {
    const unsubscribe = posts.subscribe(items => {
      const selectedPost = items.find(i => i.id === id);
      title = selectedPost.title;
      subtitle = selectedPost.subtitle;
      contribution = selectedPost.contribution;
      email = selectedPost.contactEmail;
      description = selectedPost.description;
      imageUrl = selectedPost.imageUrl;
    });

    unsubscribe();
  }

  const dispatch = createEventDispatcher();

  $: titleValid = !isEmpty(title);
  $: subtitleValid = !isEmpty(subtitle);
  $: contributionValid = !isEmpty(contribution);
  $: descriptionValid = !isEmpty(description);
  $: imageUrlValid = !isEmpty(imageUrl);
  $: emailValid = isValidEmail(email);
  $: formIsValid =
    titleValid &&
    subtitleValid &&
    contributionValid &&
    descriptionValid &&
    imageUrlValid &&
    emailValid;

  function submitForm() {
    const postData = {
      title: title,
      subtitle: subtitle,
      description: description,
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

<Modal title="Edit Service" on:cancel>
  <form on:submit|preventDefault={submitForm}>
    <TextInput
      id="title"
      label="Title"
      valid={titleValid}
      validityMessage="Please enter a valid title."
      value={title}
      on:input={event => (title = event.target.value)} />
    <TextInput
      id="subtitle"
      label="Subtitle"
      valid={subtitleValid}
      validityMessage="Please enter a valid subtitle."
      value={subtitle}
      on:input={event => (subtitle = event.target.value)} />
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
      id="description"
      label="Description"
      controlType="textarea"
      valid={descriptionValid}
      validityMessage="Please enter a valid description."
      value={description}
      on:input={event => (description = event.target.value)} />
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
