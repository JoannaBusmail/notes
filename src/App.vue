<template>
  <main>
    <!--V-SHOW  OR V-IF-->
    <!--both do the same, the difference is that V-SHOW  has the whole HTML ca be good for SEO  improving, V-IF the html is removed-->
    <div
      v-if="showModal"
      class="overlay"
    >

      <div class="modal">
        <button
          @click="closeModal"
          class="close"
        >X</button>
        <!--TRIM: removes the white spaces-->
        <textarea
          v-model.trim="newNoteText"
          name="note"
          id="note"
          cols="30"
          rows="10"
        ></textarea>

        <p class="character-counter">{{ newNoteText.length }}/180</p>
        <p v-if="errorMsg">{{ errorMsg }}</p>
        <button
          :style="{ 'margin-top': errorMsg ? '5px' : '30px' }"
          @click="handleButtonAddUpdate"
        >{{ buttonText }}</button>

      </div>
    </div>
    <div class="container">
      <header>
        <h1>Notes</h1>
        <button @click="openModal">+</button>
      </header>

      <div class="cards-container">

        <!--iterate over the noteList to render, text, date and color-->
        <!--DINAMYC STYLING: add vinding to tag style, open object-->
        <div
          class="card"
          v-for="note in noteList"
          :key="note.id"
          :style="{ backgroundColor: note.backgroundColor }"
        >
          <div class="cardTextClose">
            <p class="main-text"> {{ note.text }}</p>
            <button
              @click="deleteNote(note.id)"
              class="closeCard"
            >X</button>
          </div>
          <div class="cardTextClose">
            <p class="date">{{ note.date }}</p>
            <button
              class="editCard"
              @click="editNote(note.id)"
            >Edit</button>
          </div>
        </div>
      </div>

    </div>
  </main>
</template>

<script setup>

import { onMounted, ref, watchEffect, computed } from 'vue'


//refs
const showModal = ref(false)
// two way binding means that the value of the input is the same as the value of the variable, if one changes the other changes
// use v-model in the text area. Is in the same component we can do directly this
const newNoteText = ref('')
// we can use it in the v-if as a falsy or truthy value, if is empty is falsy, then no error message will be rendered
// if is not empty is truthy and the error message will be rendered
const errorMsg = ref('')
//we need to save and push every note in a list
const noteList = ref([])

const mode = ref('add')

const editedNote = ref(null)


onMounted(() =>
{
  //get the notes from local storage
  const notes = localStorage.getItem('notes')
  //if there are notes in local storage, parse them and set them in the note list
  if (notes) {
    noteList.value = JSON.parse(notes)
  }
})




//methods for modal
const openModal = () =>
{
  showModal.value = true
  mode.value = 'add'
}

const closeModal = () =>
{
  showModal.value = false
}

//function to get random loght color from stack overflow
function getRandomLightColor ()
{
  return "hsl(" + Math.random() * 360 + ", 100%, 75%)"

}

watchEffect(() =>
{
  // Check if newNoteText length is within the acceptable range
  if (newNoteText.value.length >= 10 && newNoteText.value.length <= 180) {
    errorMsg.value = ''
  }
})


const buttonText = computed(() =>
{
  console.log(mode.value)
  return mode.value === 'add' ? 'Add note' : 'Update note'
})


const handleButtonAddUpdate = () =>
{
  if (mode.value === 'add') {
    addNote()
  } else {
    if (editedNote.value) {
      // Update the note being edited
      editedNote.value.text = newNoteText.value
      localStorage.setItem('notes', JSON.stringify(noteList.value))
      closeModal()
      newNoteText.value = ''
      mode.value = 'add'
      editedNote.value = null // Reset noteBeingEdited after updating
    }
  }
}


const editNote = (id) =>
{
  const note = noteList.value.find(note => note.id === id)
  editedNote.value = note
  newNoteText.value = note.text

  openModal()
  mode.value = 'edit'

}



//methos to add notes
const addNote = () =>
{
  errorMsg.value = ''
  if (newNoteText.value.length < 10) { errorMsg.value = 'The note is too short, 10 characters min'; return }
  if (newNoteText.value.length > 180) { errorMsg.value = 'The note is too long, 180 characters max'; return }

  //push the data of the note in the note list
  //create an object with everything we have to push
  noteList.value.push({
    id: Math.floor(Math.random() * 1000000),
    text: newNoteText.value,
    date: new Date().toLocaleDateString(),
    backgroundColor: getRandomLightColor(),

  })
  localStorage.setItem('notes', JSON.stringify(noteList.value))
  closeModal()
  newNoteText.value = ''


}

const deleteNote = (id) =>
{
  noteList.value = noteList.value.filter(note => note.id !== id)
  localStorage.setItem('notes', JSON.stringify(noteList.value))
}


</script>

<style scoped>
main {
  height: 100vh;
  width: 100vw;
}

.container {
  max-width: 1000px;
  padding: 10px;
  margin: 0 auto;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

h1 {
  font-weight: bold;
  margin-bottom: 25px;
  font-size: 75px;
}

header button {
  border: none;
  padding: 10px;
  width: 50px;
  height: 50px;
  cursor: pointer;
  background-color: rgb(21, 20, 20);
  border-radius: 100%;
  color: white;
  font-size: 20px;
}

.card {
  position: relative;
  width: 225px;
  height: 225px;
  background-color: rgb(237, 182, 44);
  padding: 10px;
  border-radius: 15px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  margin-right: 20px;
  margin-bottom: 20px;
}



.cardTextClose {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.main-text {
  max-width: 200px;
  margin-right: 10px;
  overflow: hidden;
  white-space: pre-wrap;
  text-overflow: ellipsis;
  /* Optionally add ellipsis (...) for visually indicating truncated text */
}

.closeCard {
  position: absolute;
  top: 10px;
  right: 10px;
  border: 2px solid white;
  padding: 10px;
  width: 20px;
  height: 20px;
  cursor: pointer;
  color: white;
  border-radius: 100%;
  font-size: 14px;
  display: flex;
  align-items: center;
  justify-content: center;
  padding-left: 10px;
  background-color: rgba(255, 0, 0, 0);


}

.date {
  font-size: 12.5px;
  font-weight: bold;
}

.editCard {
  border: none;
  cursor: pointer;
  color: white;
  font-size: 13px;
  font-weight: bold;
  background-color: rgba(255, 0, 0, 0);


}

.cards-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-around;
}

.overlay {
  position: absolute;
  height: 100%;
  width: 100%;
  background-color: rgba(0, 0, 0, 0.77);
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal {
  width: 750px;
  background-color: white;
  border-radius: 10px;
  padding: 30px;
  position: relative;
  display: flex;
  flex-direction: column;
}

.modal button {
  padding: 10px 20px;
  font-size: 20px;
  width: 100%;
  background-color: blueviolet;
  border: none;
  color: white;
  cursor: pointer;
  margin-top: 15px;

}

.modal character-counter {
  color: red;
  font-size: 12px;
  margin-top: 5px;
}

.modal .close {
  border: none;
  padding: 10px;
  width: 30px;
  height: 30px;
  cursor: pointer;
  background-color: rgb(193, 15, 15);
  border-radius: 100%;
  color: white;
  font-size: 12px;
  margin-bottom: 5px;
  align-self: flex-end;
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal p {
  color: rgb(193, 15, 15);

}
</style>
```