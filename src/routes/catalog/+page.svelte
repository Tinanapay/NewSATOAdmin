<script>
import {
  collection,
  query,
  orderBy,
  onSnapshot,
  getDocs,
  updateDoc,
  deleteDoc,
  doc,
  setDoc
} from "firebase/firestore";

let componentId = "";

import { signOut, onAuthStateChanged } from "firebase/auth";
import { getStorage, ref, uploadBytes, getDownloadURL } from "firebase/storage";

import { onMount } from "svelte";
import DOMPurify from "dompurify";
import validator from "validator";


let activeTab = "catalog";
let db = null;
let app = null;
let auth = null;
let entries = [];
let componentName = "";
let discoveredBy = "";
let discoveryYear = "";
let category = "";
let saving = false;
let loading = true;


/*no auto id generator
async function saveEntry(entry) {
  if (!entry.id) {
    alert("ID is required meow");
    return;
  }

  await setDoc(
    doc(db, "components", entry.id),
    {
      name: entry.name,
      category: entry.category,
      symbol: entry.symbol,
      year: entry.year,
      img: entry.sym_img
    }
  );

}
*/

// Redirect helper
function goTo(path) {
  if (typeof window !== "undefined") window.location.href = path;
}

// Initialize Firebase and fetch entries
onMount(async () => {
  try {
    const mod = await import("$lib/firebase");
    db = mod.db;
    app = mod.app;
    auth = mod.auth;

    if (!db) { loading = false; return; }

    const q = query(collection(db, "catalog"), orderBy("name"));

    // Fetch initial data
    const snap = await getDocs(q);
    entries = snap.docs.map(d => {
      const data = d.data() || {};
      return {
        id: d.id,
        name: data.name ?? "",
        discovered: data.discovered ?? "",
        year: data.year ?? "",
        category: data.category ?? "",
        sym_img: data.sym_img ?? "",
        symbol: data.symbol ?? "",
        editing: false,
        uploading: false
      };
    });
    loading = false;

    // Real-time listener
    const unsub = onSnapshot(q, snap => {
      entries = snap.docs.map(d => {
        const data = d.data() || {};
        return {
          id: d.id,
          name: data.name ?? "",
          discovered: data.discovered ?? "",
          year: data.year ?? "",
          category: data.category ?? "",
          sym_img: data.sym_img ?? "",
          symbol: data.symbol ?? "",
          editing: false,
          uploading: false
        };
      });
    });

    // Admin auth check
    const unsubAuth = onAuthStateChanged(auth, async user => {
      if (!user) { window.location.href = "/"; return; }

      const email = (user.email || "").trim().toLowerCase();
      const snap = await getDocs(collection(db, "Admins"));
      const allowed = snap.docs.some(d => Object.values(d.data() || {}).some(v => String(v || "").trim().toLowerCase() === email));
      if (!allowed) { await auth.signOut(); window.location.href = "/"; }
    });

    return () => { unsub && unsub(); unsubAuth && unsubAuth(); };
  } catch (e) {
    console.error("Firebase init error:", e);
    loading = false;
  }
});

// Add new component
async function addEntry() {
  let id = DOMPurify.sanitize(componentId.trim());
  id = id.charAt(0).toUpperCase() + id.slice(1).toLowerCase();

  const discovered = DOMPurify.sanitize(discoveredBy.trim());
  const year = DOMPurify.sanitize(discoveryYear.trim());
  const cat = DOMPurify.sanitize(category.trim());

  if (!validator.isLength(id, { min: 1 })) { 
    alert("Component ID cannot be empty!"); 
    return; 
  }

  const regex = /^[A-Z][a-z0-9-]*$/;
  if (!regex.test(id)) {
    alert("ID must start with a capital letter and contain only letters, numbers, or hyphens!");
    return;
  }

  saving = true;
  try {
    await setDoc(doc(db, "catalog", id), {
      name: id,
      discovered,
      year,
      category: cat,
      sym_img: newImageUrl,
      symbol: ""
    });

    componentId = discoveredBy = discoveryYear = category = "";
  } catch (e) {
    alert("Failed to add: " + e.message);
  } finally { 
    saving = false; 
  }
}


// Edit entry
function editEntry(i) {
  entries[i].editing = true;
  entries = [...entries];
}

// Save edited entry
async function saveEditedEntry(i) {
  const e = entries[i];
  if (!db || !e || !e.id) return;

  saving = true;
  try {
    await updateDoc(doc(db, "catalog", e.id), {
      name: e.name,
      discovered: e.discovered,
      year: e.year,
      category: e.category,
      sym_img: e.sym_img,
      symbol: e.symbol
    });

    entries[i].editing = false;
    entries = [...entries];
  } catch (err) {
    console.error(err);
  } finally {
    saving = false;
  }
}


// Delete entry
async function deleteEntry(i) {
  const e = entries[i];
  if (!db || !e || !e.id) return;
  if (!confirm("Delete this component?")) return;

  try { await deleteDoc(doc(db,"catalog", e.id)); } 
  catch(err) { console.error(err); }
}

// Upload image
let newImageUrl = ""; // stores the uploaded image for new entry

async function uploadNewImage() {
  const fileInput = document.createElement("input");
  fileInput.type = "file";
  fileInput.accept = "image/*";

  fileInput.onchange = async (e) => {
    const file = e.target.files[0];
    if (!file) return;

    try {
      const storage = getStorage(app);
      const storageRef = ref(storage, `sym_img/${Date.now()}_${file.name}`);

      await uploadBytes(storageRef, file);
      const url = await getDownloadURL(storageRef);

      newImageUrl = url;
      alert("Image uploaded successfully!");
    } catch (err) {
      console.error("Upload failed:", err);
      alert("Upload failed: " + err.message);
    }
  };

  fileInput.click();
}

// Logout
function logout() {
  auth && signOut(auth)
    .then(()=> window.location.href = "/")
    .catch(err => console.error(err));
}
</script>

<div class="catalog-container">
 <h1>ADMIN CATALOG</h1>
 <div class="button-group">
   <button class="logout-button" class:active={activeTab === "logout"} on:click={logout}>LOG OUT</button>
   <button class="library-button" class:active={activeTab === "library"} on:click={() => goTo("library")}>LIBRARY</button>
   <button class="catalog-button" class:active={activeTab === "catalog"} on:click={() => goTo("catalog")}>CATALOG</button>
 </div>
</div>

<div class="catalog-component-box">
<input type="text" class="component-name-input" placeholder="Component Id..." bind:value={componentId}/>

<input type="text" class="discovered-input" placeholder="Discovered by..." bind:value={discoveredBy}/>
<input type="text" class="discovery-input" placeholder="Year of discovery..." bind:value={discoveryYear}/>
<input type="text" class="category-input" placeholder="Category..." bind:value={category}/>

 <button class="upload-image-button" on:click={uploadNewImage}>
    <img src="/add.svg" alt="Upload Image" class="icon-btn" />
  </button>

  <button class="add-button" on:click={addEntry}>ADD</button>
</div>

<div class="title">
  <div class="name-title">Name:</div>
  <div class="description-title">Description:</div>
</div>

{#each entries as entry, i}
  <div class="catalog-entry-box">
    {#if entry.editing}

<input
  class="name_edit"
  value={entry.id}
  disabled
/>

<input class="discovered_edit" placeholder="Discovered by..." bind:value={entry.discovered} on:input={() => (entries=[...entries])}/>
<input class="discovery_edit" placeholder="Year..." bind:value={entry.year} on:input={() => (entries=[...entries])}/>
<input class="category_edit" placeholder="Category..." bind:value={entry.category} on:input={() => (entries=[...entries])}/>

<button class="save-button" on:click={() => saveEditedEntry(i)}>Save</button>


    {:else}
      <div class="name-entry">{entry.name}</div>
      <div class="description-entry">
        {#if entry.discovered || entry.year || entry.category}
          <div>Discovered by: {entry.discovered}</div>
          <div>Year: {entry.year}</div>
          <div>Category: {entry.category}</div>
        {/if}
      </div>
      <div class="row-actions">
<div class="row-actions">

 <button class="upload-image-button" on:click={uploadNewImage}>
    <img src="/add.svg" alt="Upload Image" class="icon-btn" />
  </button>

  <button class="edit_button_input" on:click={() => editEntry(i)}>
    <img src="/write.svg" alt="Edit" class="icon-btn" />
  </button>
  <button class="delete_button_input" on:click={() => deleteEntry(i)}>
    <img src="/delete.svg" alt="Delete" class="icon-btn" />
  </button>
</div>


      </div>
    {/if}
  </div>
{/each}


<style>

  :global(body) {
    background-color: #19333c;
    margin: 0;
    padding: 0;
    color: #fff;
    font-family: Inter, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
  }

  .catalog-entry-box{
    display: flex;
    gap: 10px;
    padding: 20px;
    margin-top: 5px;
    align-items: flex-start;
  }

  .name-entry{
    background-color: #000000;
    padding: 10px;
    border-radius: 10px;
    border-color: #3E92B5;
    border-width: 2px;
    color: #CF8C44;
    border-style: solid;
    flex: 0 0 160px;
    width: 160px;
    height: 56px;
    box-sizing: border-box;
    display: flex;
    align-items: center;
    justify-content: center;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
  }

  .description-entry{
    background-color: #000000;
    padding: 10px;
    border-radius: 10px;
    border-color: #3E92B5;
    border-width: 2px;
    color: #CF8C44;
    border-style: solid;
    flex: 1 1 auto;
    min-height: 56px;
    box-sizing: border-box;
    word-break: break-word;
    white-space: normal;
    display: flex;
    align-items: flex-start;
    justify-content: space-between;
  }


  .row-actions{
    display: flex;
    gap: 8px;
    align-items: center;
  }


  .title {
    display: flex;
    gap: 10px;
    padding: 0 20px;
    margin-top: 12px;
    align-items: flex-start;
    flex-wrap: nowrap;
  }
  .name-title {
    flex: 0 0 160px;
    display: flex;
    align-items: flex-start;
    justify-content: flex-start;
    padding: 6px 10px;
    font-weight: 700;
    color: #CF8C44;
    box-sizing: border-box;
  }
  .description-title {
    flex: 1;
    display: flex;
    align-items: flex-start;
    padding: 6px 10px;
    font-weight: 700;
    color: #CF8C44;
    box-sizing: border-box;
  }
  /*.image-title {
    flex: 1;
    display: flex;
    align-items: flex-start;
    justify-content: flex-start;
    padding: 6px 10px;
    font-weight: 700;
    color: #CF8C44;
    box-sizing: border-box;
  }*/

  :global(.button-group) { display: flex; gap: 10px; margin-left: auto; padding: 0 20px; height: 50px; }
  :global(.library-button){ background-color: #3E92B5; color: #fff; border-radius: 10px; max-width: 100px; padding: 6px 10px; border-color: #3E92B5; border-width: 5px; }
  :global(.catalog-button){ background-color:  #CF8C44; color: #fff; border-radius: 10px; max-width: 100px; padding: 6px 10px; border-color: #3E92B5; border-width: 5px; }
  :global(.logout-button){ background-color: #b50b0b; color: #fff; border-radius: 10px; max-width: 100px; padding: 6px 10px; border-color: #841f03; border-width: 5px; }


  .catalog-container {
    background-color: #000000;
    height: 80px;
    width: auto;
    max-width: 2000px;
    margin: 10px 15px 0 15px;
    display: flex;
    justify-content: flex-start;
    align-items: center;
    padding: 0 20px;
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    z-index: 1000;
    color: #CF8C44;
    font-size: 24px;
    font-weight: bold;
    border-radius: 20px;
    border: 1px solid #3E92B5;
  }

  .catalog-component-box {
    display: flex;
    gap: 10px;
    padding: 20px;
    margin-top: 100px;
    height: 50px;
  }
  .component-name-input {
  flex: 0 0 160px;
  padding: 6px 10px;
  background-color: #000000;
  color: #CF8C44;
  border: 1px solid #3E92B5;
  border-radius: 10px;
  font-size: 16px;
}


.discovered-input,
.discovery-input,
.category-input {
  flex: 1;
  padding: 6px 10px;
  max-width: 1700px;
  background-color: #000000;
  color: #CF8C44;
  border: 1px solid #3E92B5;
  border-radius: 10px;
  font-size: 16px;
}

.upload-image-button{
    background-color: #0b0805;
    border-radius: 10px;
    max-width: 500px;
    padding: 6px 10px;
    border: solid #3E92B5 2px ;
    
  }
  .add-button,.save-button{
    background-color: #CF8C44;
    color: #fff;
    border-radius: 10px;
    max-width: 500px;
    padding: 6px 10px;
    border: none;
    font-size: 16px;
  }

.edit_button_input,
.delete_button_input {
  background-color: #000;
  border-radius: 10px;
  width: 50px;      /* give actual width */
  height: 50px;     /* give actual height */
  padding: 5px;
  border: none;
  display: flex; 
  justify-content: center;
  align-items: center;
}

.icon-btn {
  color: #CF8C44;
  width: 30px;
  height: 30px;
  pointer-events: none; /* So clicks go to the button, not the img */
}

.edit_button_input,
.delete_button_input {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 5px;
  border: solid #3E92B5 2px ;
  background: #000000;
  cursor: pointer;
}

 /*.img_path_input,
.edit_button_input,
.delete_button_input {
  width: 30px;
  height: 30px;
}
  
  .img_path_input{
    background-color: #000000;
    border-radius: 10px;
    max-width: 500px;
    max-height:70px;
    padding: 6px 10px;
    border: none;
    font-size: 16px;
  }
  .edit_button_input{
    background-color: #000000;
    border-radius: 10px;
    max-width: 500px;
    max-height:70px;
    padding: 6px 10px;
    border: none;
    font-size: 16px;
  }
  .delete_button_input{
    background-color: #000000;
    border-radius: 10px;
    max-width: 500px;
    max-height: 70px;
    padding: 6px 10px;
    border: none;
    font-size: 16px;
  }*/


  .name_edit {
    flex: 0 0 160px;
    width: 160px;
    height: 56px;
    padding: 6px 10px;
    background-color: #000000;
    color: #CF8C44;
    border: 1px solid #3E92B5;
    border-radius: 10px;
    font-size: 16px;
    box-sizing: border-box;
    align-self: center;
  }


  .discovered_edit,.discovery_edit,.category_edit {
    flex: 1;
    min-height: 140px;
    padding: 8px 10px;
    background-color: #000000;
    color: #CF8C44;
    border: 1px solid #3E92B5;
    border-radius: 10px;
    font-size: 16px;
    box-sizing: border-box;
    resize: vertical;
    white-space: pre-wrap;
  }

</style>