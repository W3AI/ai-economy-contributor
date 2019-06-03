<script>
    import Modal from './Modal.svelte';
    import Button from './Button.svelte';

    export let message;
</script>

<Modal title="An error occured!" on:cancel>
    <p>{message}</p>
    <hr>
    <p>Logging error in issue log...</p>
    <label>Comment</label>
    <textarea rows="3">Add issue details here</textarea>
    <Button>Help!</Button>
    <p>ToDo: Add Issue Indexing and Ranking engine!</p>
</Modal>