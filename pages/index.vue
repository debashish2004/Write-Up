<template>
  <div class="flex bg-zinc-900 h-screen">
    <!-- sidebar -->
    <div class="bg-black w-[338px] p-8 flex flex-col overflow-scroll">
      <div>
        <Logo />
      </div>

      <!-- today main container -->
      <div class="flex-grow">
        <p class="text-xs font-bold text-[#C2C2C5] mt-12 mb-4">Today</p>
        <div class="ml-2 space-y-2">
          <div
            v-for="note in todaysNotes"
            class="p-2 rounded-lg cursor-pointer"
            :class="{
              'bg-[#A1842C]': note.id === selectedNote.id,
              'hover:bg-[#A1842C]/50': note.id !== selectedNote.id,
            }"
            @click="setNote(note)"
          >
            <h3 class="text-sm font-bold text-[#F4F4F5] truncate">
              {{ note.text.substring(0, 50) }}
            </h3>
            <div class="leading-none truncate text-[#D6D6D6]">
              <span class="text-xs text-[#F4F4F5] mr-4">{{
                new Date(note.updatedAt).toLocaleDateString()
              }}</span>
              <span v-if="note.text.length > 50" class="text-xs text-[#D6D6D6]"
                >... {{ note.text.substring(50, 100) }}</span
              >
            </div>
          </div>
        </div>
      </div>
      <!-- /today main container -->

      <!-- yesterday main container -->
      <div>
        <p class="text-xs font-bold text-[#C2C2C5] mt-12 mb-4">Yesterday</p>
        <div class="ml-2 space-y-2">
          <div
            v-for="note in yesterdaysNotes"
            class="p-2 rounded-lg cursor-pointer"
            :class="{
              'bg-[#A1842C]': note.id === selectedNote.id,
              'hover:bg-[#A1842C]/50': note.id !== selectedNote.id,
            }"
            @click="setNote(note)"
          >
            <h3 class="text-sm font-bold text-[#F4F4F5] truncate">
              {{ note.text.substring(0, 50) }}
            </h3>
            <div class="leading-none truncate text-[#D6D6D6]">
              <span class="text-xs text-[#F4F4F5] mr-4">{{
                new Date(note.updatedAt).toDateString() ===
                new Date().toDateString()
                  ? 'Today'
                  : new Date(note.updatedAt).toLocaleDateString()
              }}</span>
              <span v-if="note.text.length > 50" class="text-xs text-[#D6D6D6]"
                >... {{ note.text.substring(50, 100) }}</span
              >
            </div>
          </div>
        </div>
      </div>
      <!-- /yesterday main container -->

      <!-- everything else main container -->
      <div>
        <p class="text-xs font-bold text-[#C2C2C5] mt-12 mb-4">Earlier</p>
        <div class="ml-2 space-y-2">
          <div
            v-for="note in earlierNotes"
            class="p-2 rounded-lg cursor-pointer"
            :class="{
              'bg-[#A1842C]': note.id === selectedNote.id,
              'hover:bg-[#A1842C]/50': note.id !== selectedNote.id,
            }"
            @click="setNote(note)"
          >
            <h3 class="text-sm font-bold text-[#F4F4F5] truncate">
              {{ note.text.substring(0, 50) }}
            </h3>
            <div class="leading-none truncate text-[#D6D6D6]">
              <span class="text-xs text-[#F4F4F5] mr-4">{{
                new Date(note.updatedAt).toDateString() ===
                new Date().toDateString()
                  ? 'Today'
                  : new Date(note.updatedAt).toLocaleDateString()
              }}</span>
              <span v-if="note.text.length > 50" class="text-xs text-[#D6D6D6]"
                >... {{ note.text.substring(50, 100) }}</span
              >
            </div>
          </div>
        </div>
      </div>
      <!-- /everything else main container -->
    </div>
    <!-- /sidebar -->
    <!-- note container -->
    <div class="w-full flex flex-col">
      <div class="  flex justify-between w-full items-start p-8">
        <button
          class="inline-flex items-center text-xs text-[#C2C2C5] hover:text-white font-bold space-x-2"
          @click="createNewNote"
        >
          <CreateNote/>
        </button>

        <button>
          <Trash
            class="text-[#6D6D73] "
            @click="deleteNote"
          />
        </button>
      </div>

      <div class="max-w-[437px] mx-auto w-full flex-grow flex flex-col">
        <p class="text-[#929292] font-playfair  hover:text-white">
          {{ new Date(selectedNote.updatedAt).toLocaleDateString() }}
        </p>
          <textarea
              ref="textarea"
              v-model="updatedNote"
              name="note"
              id="note"
              class="border border-white rounded text-white my-4 font-playfair w-full bg-transparent focus:outline-none resize-none flex-grow"
              @input="
            () => {
              debouncedFn()
              selectedNote.text = updatedNote
            }
          "
          >
        </textarea>
      </div>

      <button
        class="text-zinc-400 hover:text-white text-sm font-bold absolute right-0 bottom-0 p-8"
        @click="logout"
      >
        <Logout />
      </button>
    </div>
    <!-- /note container -->
  </div>
</template>

<script setup>
import Swal from 'sweetalert2'

const updatedNote = ref('')
const notes = ref([])
const selectedNote = ref({})
const textarea = ref(null)
definePageMeta({
  middleware: ['auth'],
})

function setNote(note) {
  selectedNote.value = note
  updatedNote.value = note.text
}

function logout() {
  const jwtCookie = useCookie('NoteNestJWT')
  jwtCookie.value = null
  navigateTo('/login')
}

async function deleteNote() {
  const { isConfirmed } = await Swal.fire({
    title: 'Are you sure?',
    text: 'This will delete your note permanently. ',
    icon: 'warning',
    confirmButtonText: 'Yes, delete!',
    showCancelButton: true,
  })

  if (isConfirmed) {
    // truly delete
    await $fetch(`/api/notes/${selectedNote.value.id}`, {
      method: 'DELETE',
    })

    const index = notes.value.findIndex((note) => {
      return note.id === selectedNote.value.id
    })
    console.log(index)
    notes.value.splice(index, 1)
  }
}

async function createNewNote() {
  try {
    const newNote = await $fetch(`/api/write`, {
      method: 'POST',
    })

    notes.value.unshift(newNote)
    selectedNote.value = notes.value[0]
    updatedNote.value = ''
    textarea.value.focus()
  } catch (err) {
    console.log(err)
  }
}

const debouncedFn = useDebounceFn(async () => {
  await updateNote()
}, 1000)

async function updateNote() {
  try {
    await $fetch(`/api/notes/${selectedNote.value.id}`, {
      method: 'PATCH',
      body: {
        updatedNote: updatedNote.value,
      },
    })
  } catch (err) {
    console.log(err)
  }
}

const todaysNotes = computed(() => {
  return notes.value.filter((note) => {
    const noteDate = new Date(note.updatedAt)
    return noteDate.toDateString() === new Date().toDateString()
  })
})

const yesterdaysNotes = computed(() => {
  const yesterday = new Date()
  yesterday.setDate(yesterday.getDate() - 1)

  return notes.value.filter((note) => {
    const noteDate = new Date(note.updatedAt)
    return noteDate.toDateString() === yesterday.toDateString()
  })
})

const earlierNotes = computed(() => {
  const yesterday = new Date()
  yesterday.setDate(yesterday.getDate() - 1)

  return notes.value.filter((note) => {
    const noteDate = new Date(note.updatedAt)
    return (
      noteDate < yesterday &&
      noteDate.toDateString() !== yesterday.toDateString()
    )
  })
})

onMounted(async () => {
  notes.value = await $fetch('/api/notes')
  notes.value.sort((a, b) => new Date(b.updatedAt) - new Date(a.updatedAt))

  if (notes.value.length > 0) selectedNote.value = notes.value[0]
  else {
    await createNewNote()
    selectedNote.value = notes.value[0]
  }

  updatedNote.value = selectedNote.value.text

  textarea.value.focus()
})
</script>
