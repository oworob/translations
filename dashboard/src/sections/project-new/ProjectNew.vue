<script setup lang="ts">
import LanguageMultiSelect from '@/components/LanguageMultiSelect.vue'
import LanguageSelect from '@/components/LanguageSelect.vue'
import Loading from '@/components/Loading.vue'
import UserMultiSelect from '@/components/UserMultiSelect.vue'
import type IApiLanguage from '@/models/project/language'
import type { IApiUser } from '@/models/user/user'
import LanguageService from '@/services/LanguageService'
import { Icon } from '@iconify/vue/dist/iconify.js'
import { onMounted, ref } from 'vue'
import ImportEntries from './ImportEntries.vue'

const loading = ref(true) // load languages
const notes = ref<string[]>([])
const languages = ref<IApiLanguage[]>([])
const selected_original_language = ref<IApiLanguage>()
const selected_desired_languages = ref<IApiLanguage[]>([])
const selected_users = ref<IApiUser[]>([])
const entries = ref<string[]>([])

const import_window_open = ref(false)

onMounted(() => {
  LanguageService.GetLanguages().then((res) => {
    languages.value = res.data
    selected_original_language.value = languages.value[0]
    loading.value = false
  })
})

function Submit() {
  console.log('submit')
}

function AddNewNote() {
  if (notes.value.length < 5) {
    notes.value.push('')
  }
}

function DeleteNote(index: number) {
  notes.value.splice(index, 1)
}

function AddNewEntry() {
  if (entries.value.length < 1000) {
    entries.value.push('')
  }
}

function DeleteEntry(index: number) {
  entries.value.splice(index, 1)
}

function HandleOriginalLanguageSelected(id: number) {
  selected_original_language.value = languages.value.find((language) => language.id === id)!
}

function HandleDesiredLanguageSelected(id: number) {
  const language = languages.value.find((language) => language.id === id)!
  if (
    !selected_desired_languages.value.find(
      (selected_language) => selected_language.id === language.id,
    )
  ) {
    selected_desired_languages.value.push(language)
  } else {
    selected_desired_languages.value.splice(selected_desired_languages.value.indexOf(language), 1)
  }
}

function HandleUserSelected(user: IApiUser) {
  if (!selected_users.value.find((selected_user) => selected_user.id === user.id)) {
    selected_users.value.push(user)
  } else {
    selected_users.value.splice(selected_users.value.indexOf(user), 1)
  }
}

function SaveImportedEntries(new_entries: string[], mode: string) {
  if (mode === 'append') {
    entries.value = entries.value.concat(new_entries)
  } else {
    entries.value = new_entries
  }
  import_window_open.value = false
}
</script>

<template>
  <Loading v-if="loading" />
  <main class="ProjectNew" v-if="!loading">
    <form class="project-form" @submit.prevent="Submit">
      <div class="form-section">
        <h2>Create New Project</h2>
        <h4>Details</h4>
        <div class="form-item">
          <label class="hint">Project Name*</label>
          <input type="text" placeholder="My Awesome Project" />
        </div>
        <div class="form-item">
          <label class="hint">Description</label>
          <textarea placeholder="A brief description of your project" spellcheck="false"></textarea>
        </div>
        <div class="form-item">
          <div class="notes-header">
            <h4>Notes</h4>
            <button class="secondary" type="button" @click="AddNewNote">New Note</button>
          </div>
          <div class="notes">
            <div v-if="notes.length === 0" class="hint">
              Add some notes to guide your contributors! Some examples include:
              <ul>
                <li>Please remember about punctuation.</li>
                <li>Try to keep similar character count to original.</li>
                <li>Do not use slang.</li>
              </ul>
            </div>
            <div class="note" v-for="(_, index) in notes" :key="index">
              <input
                type="text"
                v-model="notes[index]"
                spellcheck="false"
                placeholder="This is an empty note."
              />
              <button type="button" class="tertiary icon" @click="DeleteNote(index)">
                <Icon icon="solar:trash-bin-2-bold" />
              </button>
            </div>
          </div>
        </div>
      </div>

      <div class="form-section">
        <div class="form-item">
          <h4>Original Language</h4>
          <p class="hint">
            The language in which the original text is written. This will be the base for all
            translations.
          </p>
          <LanguageSelect
            :languages="languages"
            :selected_language="selected_original_language!"
            @language-selected="HandleOriginalLanguageSelected"
          />
        </div>

        <div class="form-item">
          <h4>Languages*</h4>
          <p class="hint">The languages you would like your project to be translated into.</p>
          <LanguageMultiSelect
            :languages="languages"
            :selected_languages="selected_desired_languages"
            @language-selected="HandleDesiredLanguageSelected"
          />

          <div class="selected-languages">
            <button
              type="button"
              class="secondary with-icon"
              v-for="language in selected_desired_languages"
              @click="
                selected_desired_languages.splice(selected_desired_languages.indexOf(language), 1)
              "
            >
              <Icon :icon="'circle-flags:' + language.code" />
              <span>{{ language.title_native }}</span>
              <Icon icon="solar:trash-bin-2-bold" />
            </button>
          </div>
        </div>
      </div>

      <div class="form-section">
        <h4>Contributors</h4>
        <p class="hint">Invite users to contribute to your project and translate your entries.</p>
        <UserMultiSelect :selected_users="selected_users" @userSelected="HandleUserSelected" />
        <div class="selected-users">
          <button
            class="secondary with-icon"
            v-for="user in selected_users"
            @click="selected_users.splice(selected_users.indexOf(user), 1)"
          >
            <span>{{ user.username }}</span>
            <Icon icon="solar:trash-bin-2-bold" />
          </button>
        </div>
      </div>

      <div class="form-section entry-section">
        <div class="form-item">
          <header class="entry-header">
            <h2>Entries</h2>
            <div class="entry-actions">
              <button type="button" class="primary with-icon" @click="AddNewEntry">
                <Icon icon="solar:add-circle-bold" />New Entry
              </button>
              <button type="button" class="secondary with-icon" @click="import_window_open = true">
                <Icon icon="solar:upload-square-bold" />Import from CSV
              </button>
            </div>
          </header>
          <p class="hint">Add entries to be translated. Each entry should be unique.</p>
          <div class="entries">
            <div class="entry" v-for="(_, index) in entries" :key="index">
              <p class="hint">{{ index + 1 }}.</p>
              <input
                type="text"
                v-model="entries[index]"
                spellcheck="false"
                placeholder="This is an empty entry."
              />
              <button type="button" class="tertiary icon" @click="DeleteEntry(index)">
                <Icon icon="solar:trash-bin-2-bold" />
              </button>
            </div>
          </div>
        </div>
      </div>
    </form>
  </main>
  <ImportEntries
    v-if="import_window_open"
    @close="import_window_open = false"
    @save-entries="SaveImportedEntries"
  />
</template>

<style scoped lang="scss">
.ProjectNew {
  padding: 1rem;
}

.project-form {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  gap: 1rem;
  .form-section {
    padding: 1rem;
    display: flex;
    flex-direction: column;
    gap: 1rem;
    .form-item {
      display: flex;
      flex-direction: column;
      gap: 0.5rem;
      textarea {
        min-height: 5rem;
      }
    }
  }
}

.notes-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.notes {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  .note {
    input {
      flex-grow: 1;
    }
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }
}

.language-select-wrapper {
  position: relative;
  input {
    width: calc(100% - 2rem);
  }
  .language-select {
    display: flex;
    flex-direction: column;
    button.tertiary {
      padding-left: 0;
    }
  }
}

.selected-languages {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.entry-section {
  grid-column: span 3;
  .entry-header {
    display: flex;
    justify-content: space-between;
  }
  .entry-actions {
    display: flex;
    gap: 0.5rem;
  }
  .entries {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
    .entry {
      display: flex;
      gap: 0.5rem;
      align-items: center;
      input {
        flex-grow: 1;
      }
    }
  }
}
</style>
