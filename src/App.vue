<script setup>
import { computed, ref } from 'vue'

const schoolSites = [1, 2, 3, 4, 5]
const grades = Array.from({ length: 9 }, (_, index) => index)
const topics = [
  'Math Reteach',
  'ELA Reteach',
  'Intervention',
  'Number Stories',
  'Guided Reading',
]

const schoolSite = ref(4)
const grade = ref(4)
const printDate = ref('2026-08-18')
const topic = ref('Math Reteach')
const scholarPages = ref(new Set([3, 5, 7, 8]))
const labelAsScholar = ref(true)
const enlargedPage = ref(null)

const previewPages = [
  { number: 1, title: 'Cover', scholar: false },
  { number: 2, title: 'Directions', scholar: false },
  { number: 3, title: 'Worksheet A', scholar: true },
  { number: 4, title: 'Worksheet B', scholar: false },
  { number: 5, title: 'Teacher Notes', scholar: true },
  { number: 6, title: 'Practice Set', scholar: false },
  { number: 7, title: 'Guided Work', scholar: true },
  { number: 8, title: 'Exit Ticket', scholar: true },
]

const formattedDate = computed(() => {
  if (!printDate.value) return '000000'

  const [year, month, day] = printDate.value.split('-')
  return `${month}${day}${year.slice(-2)}`
})

const baseFileName = computed(() => {
  return `S${schoolSite.value}.${grade.value}.${formattedDate.value}.${topic.value.replaceAll(' ', '')}`
})

const teacherFileName = computed(() => `${baseFileName.value} Teacher Packet`)
const scholarFileName = computed(() => `${baseFileName.value} Scholar Packet`)

function isScholarPage(pageNumber) {
  return scholarPages.value.has(pageNumber)
}

function toggleScholarPage(pageNumber) {
  const next = new Set(scholarPages.value)

  if (next.has(pageNumber)) {
    next.delete(pageNumber)
  } else {
    next.add(pageNumber)
  }

  scholarPages.value = next
}

function handlePageClick(pageNumber) {
  if (labelAsScholar.value) {
    toggleScholarPage(pageNumber)
    return
  }

  enlargedPage.value = previewPages.find((page) => page.number === pageNumber) ?? null
}

function closeEnlargedPage() {
  enlargedPage.value = null
}
</script>

<template>
  <div class="shell">
    <header class="topbar">
      <nav class="topnav" aria-label="Primary">
        <a href="#">Home</a>
        <a class="active" href="#">Unscripted Print</a>
        <a href="#">More</a>
      </nav>

      <div class="brand">Giraldo Utils</div>
    </header>

    <main class="workspace">
      <section class="panel setup-panel" aria-labelledby="setup-title">
        <h1 id="setup-title">Setup</h1>

        <div class="field-grid">
          <label class="field">
            <span>School Site</span>
            <select v-model="schoolSite">
              <option v-for="site in schoolSites" :key="site" :value="site">{{ site }}</option>
            </select>
          </label>

          <label class="field">
            <span>Grade</span>
            <select v-model="grade">
              <option v-for="level in grades" :key="level" :value="level">{{ level }}</option>
            </select>
          </label>

          <label class="field">
            <span>Date</span>
            <input v-model="printDate" type="date" />
          </label>

          <label class="field">
            <span>Topic</span>
            <select v-model="topic">
              <option v-for="option in topics" :key="option" :value="option">{{ option }}</option>
            </select>
          </label>
        </div>

        <div class="upload-box">
          <div class="upload-row">
            <div>
              <p class="label">Upload .docx</p>
              <p class="muted">Drop the files that need to be glued into one packet.</p>
            </div>
            <button type="button">Upload</button>
          </div>

          <div class="upload-list">
            <span class="file-pill">p1.docx</span>
            <span class="file-pill">p2.docx</span>
            <span class="file-pill file-pill--ghost">+ more files</span>
          </div>
        </div>

        <div class="filename-preview">
          <p class="label">Output name</p>
          <p class="filename">{{ teacherFileName }}</p>
          <p class="muted">Scholar downloads use the same naming pattern with Scholar Packet.</p>
        </div>
      </section>

      <section class="panel preview-panel" aria-labelledby="preview-title">
        <div class="preview-head">
          <h2 id="preview-title">Preview</h2>

          <label class="scholar-toggle">
            <input v-model="labelAsScholar" type="checkbox" />
            <span>Label as Scholar</span>
          </label>
        </div>

        <div class="preview-grid" aria-label="Page preview">
          <button
            v-for="page in previewPages"
            :key="page.number"
            type="button"
            class="page-card"
            :class="{ 'page-card--scholar': isScholarPage(page.number) }"
            @click="handlePageClick(page.number)"
          >
            <div class="page-art">
              <div class="page-sheet">
                <div class="page-sheet__body">
                  <span class="page-line page-line--wide"></span>
                  <span v-for="line in page.lines" :key="line" class="page-line">{{ line }}</span>
                </div>
              </div>
            </div>

            <span class="page-number">{{ page.number }}</span>
            <span v-if="isScholarPage(page.number)" class="page-scholar-tag">S</span>
          </button>
        </div>

        <div class="footer-actions">
          <button type="button" class="download-button download-button--muted">Download whole packet</button>
          <button type="button" class="download-button">Download Scholar packet</button>
        </div>

        <div class="download-info">
          <div>
            <p class="label">Full PDF</p>
            <p class="muted">Includes every imported page.</p>
          </div>
          <div>
            <p class="label">Scholar PDF</p>
            <p class="muted">Includes only the pages marked as scholar pages.</p>
          </div>
          <div>
            <p class="label">Scholar pages</p>
            <p class="muted">Selected: {{ Array.from(scholarPages).sort((a, b) => a - b).join(', ') }}</p>
          </div>
        </div>

        <div class="filename-row">
          <div>
            <p class="label">Teacher packet</p>
            <p class="filename">{{ teacherFileName }}</p>
          </div>
          <div>
            <p class="label">Scholar packet</p>
            <p class="filename">{{ scholarFileName }}</p>
          </div>
        </div>
      </section>
    </main>

    <teleport to="body">
      <div v-if="enlargedPage" class="preview-overlay" @click.self="closeEnlargedPage">
        <section class="preview-modal" role="dialog" aria-modal="true" aria-labelledby="modal-title">
          <div class="preview-modal__head">
            <h3 id="modal-title">Page {{ enlargedPage.number }}</h3>
            <button type="button" class="close-button" @click="closeEnlargedPage">Close</button>
          </div>

          <div class="preview-modal__body">
            <span class="page-number">{{ enlargedPage.number }}</span>
            <span class="page-art page-art--large"></span>
            <p class="muted">{{ enlargedPage.title }}</p>
          </div>
        </section>
      </div>
    </teleport>
  </div>
</template>
