<template>
  <div class="notes" v-el:notes>
      <note
        v-for="note in notes"
        :note="note"
        v-on:click="selectNote(note)"
        >
      </note>
  </div>
</template>


<script>
import Masonry from 'masonry-layout'
import Note from './Note.vue'
import noteRepository from '../../data/NoteRepository'
export default {
  components: {
    Note
  },
  data () {
    return {
      notes: []
    }
  },
  methods: {
    selectNote ({key, title, content}) {
      // notify listeners that user selected a note
      // pass in a copy of the note to prevent edits on the original note in the array
      this.$dispatch('note.selected', {key, title, content})
    }
  },
  watch: {
    'notes': { // watch the notes array for changes
      handler () {
        this.masonry.reloadItems()
        this.masonry.layout()
      },
      deep: true // we also want to watch changed inside individual notes
    }
  },
  ready () {
    this.masonry = new Masonry(this.$els.notes, {
      itemSelector: '.note',
      gutter: 5,
      fitWidth: true
    })
    noteRepository.on('added', (note) => {
      this.notes.unshift(note) // add the note to the beginning of the array
    })
    noteRepository.on('changed', ({key, title, content}) => {
      let outdatedNote = noteRepository.find(this.notes, key) // get specific note from the notes in our VM by key
      outdatedNote.title = title
      outdatedNote.content = content
    })
    noteRepository.on('removed', ({key}) => {
      let noteToRemove = noteRepository.find(this.notes, key) // get specific note from the notes in our VM by key
      this.notes.$remove(noteToRemove) // remove note from notes array
    })
  }
}
</script>