<script>
import TodoItem from './TodoItem.svelte';
import { db } from './firebase';
import { collectionData } from 'rxfire/firestore';
import { startWith } from 'rxjs/operators';

export let uid;

let text = 'some task';

const query = db.collection('todos').where('uid', '==', uid).orderBy('created');

const todos = collectionData(query, 'id').pipe(startWith([]));

//nice
function add() {
    db.collection('todos').add({
        complete: false,
        created: Date.now(),
        text: text,
        uid: uid
    });
    text = '';
}

function updateStatus(event) {
    console.log(event);
    const { id, newStatus } = event.detail;
    db.collection('todos').doc(id).update({ complete: newStatus });
}

function removeItem(event) {
    const { id } = event.detail;
    db.collection('todos').doc(id).delete(); 
}


</script>

<section>
    <ul>
        {#each $todos as todo}
            <TodoItem {...todo} on:remove={removeItem} on:toggle={updateStatus} />
        {/each}
    </ul>

    <input type="text" bind:value={text}>

    <hr>

    <p>Your task: <strong>{ text }</strong></p>

    <button on:click={add}>Add task</button>
</section>

<style>
ul {
    background: #eee;
    list-style: none;
}
</style>