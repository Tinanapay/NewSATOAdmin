<script>
import { goto } from '$app/navigation';
import { getAuth, signOut } from "firebase/auth";
import { onMount } from "svelte";

let activeTab = "library";
let auth;

onMount(async () => {
  const mod = await import('$lib/firebase'); // your firebase config
  auth = getAuth(mod.app); // initialize auth AFTER firebase app exists
});

// Logout function
function logout() {
  if (!auth) return console.error("Auth not ready");
  signOut(auth)
    .then(() => window.location.href = "/")
    .catch(err => console.error(err));
}

// Navigation
function goTo(tab) {
  activeTab = tab;
  if (tab === "library") goto("/library");
  if (tab === "catalog") goto("/catalog");
}

// Update note functionality
let updateText = "";
let updates = [];

 // Load from localStorage on mount
  onMount(() => {
    const saved = localStorage.getItem("updates");
    if (saved) updates = JSON.parse(saved);
  });

function submitUpdate() {
  if (updateText.trim() === "") return;
  updates = [...updates, updateText];
  updateText = "";
    localStorage.setItem("updates", JSON.stringify(updates));
}
</script>


<div class = "landingpage-container">
<h1> S.A.T.O Admin Panel (*^▽^*)</h1>
<p>ミ || Navigate to where u want to go ehe ||ヽ(*￣▽￣*)ノミ</p>
   <p> (❁´◡`❁) Configure the Library and Catalog sections. (❁´◡`❁)</p>
</div>

<div class="button-group">
  <button class="gotoLib" class:active={activeTab === "library"} on:click={() => goTo("library")}>
    ⇒ Library .｡.:*☆
  </button>

  <button class="gotoCat" class:active={activeTab === "catalog"} on:click={() => goTo("catalog")}>
    ⇒ Catalog .｡.:*☆
  </button>
</div>

<div class = "note_update">

<p>Note: Please note every update to be made, as to keep track. ☆*: .｡. o(≧▽≦)o .｡.:*☆</p>

<input type="text" class="input-update"placeholder="Updates here..." bind:value={updateText}
 on:keydown={(e) => e.key === "Enter" && submitUpdate()} />
<button class="submit-update" on:click={submitUpdate}>Submit</button>

<ul class="updates-list">
  {#each updates.slice().reverse() as u}
   <ul>✍️(◔◡◔)  {u}</ul>
    
  {/each}
</ul>

<button class="logout-button" on:click={logout}>LOG OUT </button>

</div>


<style>

  :global(body) {
    background-color: #19333c;
    margin: 0;
    padding: 0;
    color: #CF8C44;
   width: 100%;
  height: 100% ;
  }


.landingpage-container {
  display: flex;
  flex-direction: column;
  align-items: center; /* centers horizontally */
  justify-content: flex-start; /* starts from top, not vertical center */
  width: 100%;
  padding: 2rem;
  box-sizing: border-box;
  border: solid #3E92B5 3px ;
  border-radius: 12px;
  background-color:  #065a63;
}

.note_update {
  background-color: #065a63;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  max-width: 900px;
  margin: 40px auto;
  text-align: center;
  color: #CF8C44;
}
.input-update {
  padding: 10px;
  border-radius: 6px;
  border: 1px solid #3E92B5;
  width: 60%;
  max-width: 400px;
  margin-right: 10px;
}
.submit-update {
  padding: 10px 20px;
  border-radius: 6px;
  background-color: #CF8C44;
  color: #19333c;
  border: solid #43a0c7 3px ;
  cursor: pointer;
  font-weight: bold;
} 
.button-group {
  display: flex;
  gap: 10px;
  margin-top: 20px;
  justify-content: center;
}

.button-group button {
  padding: 1rem 2rem;
  border-radius: 12px;
  width: clamp(200px, 30%, 600px);      /* scale with container */
  font-size: clamp(1.2rem, 2vw, 2rem);
  cursor: pointer;
  font-weight: bold;
  text-align: center;
  background-color: #000;
  color: #CF8C44;
  border: 2px solid #3E92B5;
}

.gotoLib {
  background-color: #CF8C44;
  color: #19333c;
  border: none;
}
.gotoCat {
  background-color: #CF8C44;
  color: #19333c;
  border: none;
}
.button-group button {
  transition: all 0.2s ease;
}

.button-group button:hover {
  transform: scale(1.05);
}
.logout-button {
  margin-top: 40px;
  padding: 12px 24px;
  border-radius: 8px;
  background-color: #CF8C44;
  color: #19333c;
  border: none;
  cursor: pointer;
  font-weight: bold;
}

.updates-list {
  list-style-type: none;
  padding: 0;
  margin-top: 20px;
  text-align: center;
}
</style>
