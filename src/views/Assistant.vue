<template>
  <div class="container-flex">
    <div class="menu assistant-panel assistant-layout">
      <div class="assistant-hero">
        <p class="title-menu">Assistant</p>
        <p class="small">
          Paste a job description, upload your CV, then step through the
          suggestions, cover letter, and revised CV outputs.
        </p>
      </div>

      <div class="assistant-grid">
        <section class="assistant-card">
          <div class="assistant-card-head">
            <div>
              <p class="eyebrow">Input</p>
              <h3>Job description</h3>
            </div>
            <span class="badge">Step 1</span>
          </div>

          <textarea
            v-model="jobDescription"
            rows="10"
            class="role-input"
            placeholder="Paste the job description here"
          />
        </section>

        <section class="assistant-card">
          <div class="assistant-card-head">
            <div>
              <p class="eyebrow">Input</p>
              <h3>Upload CV documents</h3>
            </div>
            <span class="badge">Step 2</span>
          </div>

          <label class="upload-box">
            <input
              type="file"
              multiple
              accept=".pdf,.doc,.docx,.txt"
              @change="handleFiles"
            />
            <span>Drop or choose CV files</span>
          </label>

          <div v-if="files.length" class="chip-row">
            <div v-for="file in files" :key="file.name" class="file-chip">
              {{ file.name }}
            </div>
          </div>
        </section>

        <section class="assistant-card assistant-card-wide">
          <div class="assistant-card-head">
            <div>
              <p class="eyebrow">Action</p>
              <h3>Run the analysis</h3>
            </div>
            <span class="badge">Step 3</span>
          </div>

          <div class="action-row">
            <button
              class="menu-button analyze-button"
              :disabled="!canAnalyze"
              @click="runAnalysis"
            >
              Analyse documents
            </button>
            <p class="small">
              {{ analysisStatus }}
            </p>
          </div>

          <div v-if="!canAnalyze && !analysisReady" class="warning-banner">
            Upload at least one CV document before analysing.
          </div>
        </section>

        <section class="assistant-card">
          <div class="assistant-card-head">
            <div>
              <p class="eyebrow">Output</p>
              <h3>Suggestions</h3>
            </div>
            <span class="badge">{{ analysisReady ? "Ready" : "Pending" }}</span>
          </div>

          <div class="suggestion-list">
            <div
              v-for="item in suggestions"
              :key="item"
              class="suggestion-item"
            >
              {{ item }}
            </div>
          </div>
        </section>

        <section class="assistant-card">
          <div class="assistant-card-head">
            <div>
              <p class="eyebrow">Output</p>
              <h3>Cover letter</h3>
            </div>
            <span class="badge">
              {{ coverLetterReady ? "Generated" : "Optional" }}
            </span>
          </div>

          <p class="small">
            {{
              coverLetterReady
                ? "Here it is."
                : "Would you like to generate a cover letter?"
            }}
          </p>
          <button
            class="menu-button analyze-button"
            :disabled="!analysisReady"
            @click="generateCoverLetter"
          >
            Generate cover letter
          </button>
          <template v-if="coverLetterReady">
            <p class="small">{{ coverLetterIntro }}</p>
            <div class="output-actions">
              <a
                :href="coverLetterUrl"
                class="menu-button"
                target="_blank"
                download
              >
                Open cover letter PDF
              </a>
              <a :href="coverLetterUrl" class="menu-button" target="_blank">
                View cover letter PDF
              </a>
            </div>
            <object
              :data="coverLetterUrl"
              type="application/pdf"
              class="pdf-frame"
            >
              <a :href="coverLetterUrl" target="_blank">
                Open cover letter PDF
              </a>
            </object>
          </template>
        </section>

        <section class="assistant-card">
          <div class="assistant-card-head">
            <div>
              <p class="eyebrow">Output</p>
              <h3>Apply suggestions to CV</h3>
            </div>
            <span class="badge">{{ cvReady ? "Applied" : "Optional" }}</span>
          </div>

          <p class="small">
            {{
              cvReady
                ? "Here you go."
                : "Would you like to apply the suggestions to your CV?"
            }}
          </p>
          <button
            class="menu-button analyze-button"
            :disabled="!analysisReady"
            @click="applySuggestions"
          >
            Apply suggestions to CV
          </button>
          <template v-if="cvReady">
            <p class="small">{{ cvIntro }}</p>
            <div class="output-actions">
              <a :href="cvUrl" class="menu-button" target="_blank" download>
                Open updated CV PDF
              </a>
              <a :href="cvUrl" class="menu-button" target="_blank">
                View updated CV PDF
              </a>
            </div>
            <object :data="cvUrl" type="application/pdf" class="pdf-frame">
              <a :href="cvUrl" target="_blank">Open updated CV PDF</a>
            </object>
          </template>
        </section>

        <section class="assistant-card assistant-card-wide">
          <div class="assistant-card-head">
            <div>
              <p class="eyebrow">ATS</p>
              <h3>Optimiser</h3>
            </div>
            <span class="badge">{{ atsBand }}</span>
          </div>

          <div class="optimizer-grid">
            <div class="optimizer-scorecard">
              <div class="optimizer-ring">
                <div class="optimizer-score">{{ atsScore }}%</div>
                <div class="optimizer-label">{{ atsBand }}</div>
              </div>
              <div class="optimizer-meter">
                <div
                  class="optimizer-meter-fill"
                  :style="{ width: `${atsScore}%` }"
                />
              </div>
              <p class="small">{{ atsSummary }}</p>
            </div>

            <div class="optimizer-panel">
              <p class="optimizer-heading">Top keywords to keep</p>
              <div class="chip-row">
                <p v-for="item in matchedKeywords" :key="item" class="feature">
                  {{ item }}
                </p>
                <p v-if="!matchedKeywords.length" class="feature muted-chip">
                  No keywords matched yet
                </p>
              </div>

              <p class="optimizer-heading">Priority gaps</p>
              <div class="suggestion-list">
                <div
                  v-for="item in priorityGaps"
                  :key="item"
                  class="suggestion-item"
                >
                  {{ item }}
                </div>
              </div>

              <div class="optimizer-actions">
                <button
                  class="menu-button analyze-button"
                  :disabled="!analysisReady"
                  @click="generateCoverLetter"
                >
                  {{
                    coverLetterReady
                      ? "Regenerate cover letter"
                      : "Generate cover letter"
                  }}
                </button>
                <button
                  class="menu-button analyze-button"
                  :disabled="!analysisReady"
                  @click="applySuggestions"
                >
                  {{
                    cvReady
                      ? "Refresh CV suggestions"
                      : "Apply suggestions to CV"
                  }}
                </button>
              </div>
            </div>
          </div>
        </section>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, ref } from "vue"

const coverLetterUrl = "/Cover-letter-template.pdf"
const cvUrl = "/CV-template.pdf"

const analysisReady = ref(false)
const coverLetterReady = ref(false)
const cvReady = ref(false)
const files = ref([])
const jobDescription = ref(
  `Senior Product Designer

We are looking for a product designer who can translate complex workflows into clear, accessible user experiences. You will collaborate with engineering and product teams, run design critiques, and use data to improve conversion and adoption.

Requirements:
- Experience with Figma, user research, and prototyping
- Strong communication and stakeholder management skills
- Comfort working in agile, cross-functional teams
- Ability to produce polished design systems and documentation`
)

const skillLibrary = [
  "figma",
  "user research",
  "prototyping",
  "communication",
  "stakeholder",
  "agile",
  "cross-functional",
  "conversion",
  "adoption",
  "design systems",
  "documentation",
  "javascript",
  "vue",
  "typescript",
  "testing",
  "leadership"
]

const matchedKeywords = computed(() => {
  const text = jobDescription.value.toLowerCase()
  return skillLibrary.filter((skill) => text.includes(skill))
})

const missingKeywords = computed(() => {
  const fileText = files.value.map((file) => file.name.toLowerCase()).join(" ")
  return matchedKeywords.value.filter((skill) => !fileText.includes(skill))
})

const canAnalyze = computed(() => files.value.length > 0)

const analysisStatus = computed(() => {
  if (analysisReady.value) return "Here are suggestions."
  if (canAnalyze.value) return "Ready when you are."
  return "Upload at least one CV document to analyse."
})

const suggestions = computed(() => {
  if (!analysisReady.value) {
    return [
      "Paste a job description to surface relevant gaps.",
      "Upload one or more CV files to review.",
      "Click Analyse documents to generate suggestions."
    ]
  }

  return [
    matchedKeywords.value.length
      ? `Matched focus areas: ${matchedKeywords.value.slice(0, 3).join(", ")}.`
      : "No obvious target skills detected yet.",
    missingKeywords.value.length
      ? `Add more evidence for: ${missingKeywords.value.slice(0, 3).join(", ")}.`
      : "The CV appears aligned with the key role terms.",
    files.value.length
      ? `Reviewed ${files.value.length} uploaded document${files.value.length > 1 ? "s" : ""}.`
      : "No CV file has been added yet."
  ]
})

const coverLetterIntro = computed(() => {
  if (!analysisReady.value) return ""
  return `Generated a cover letter draft for the ${jobTitle()} role.`
})

const cvIntro = computed(() => {
  if (!analysisReady.value) return ""
  return `Applied the suggested changes to a CV draft for the ${jobTitle()} role.`
})

const atsScore = computed(() => {
  if (!analysisReady.value) return 0
  return Math.min(
    62 + matchedKeywords.value.length * 4 + (files.value.length ? 6 : 0),
    96
  )
})

const atsSummary = computed(() => {
  if (!analysisReady.value)
    return "Run the analysis to see a prototype ATS score."
  if (atsScore.value >= 85)
    return "Strong keyword alignment with room to tighten wording."
  if (atsScore.value >= 70)
    return "Good baseline fit. Add missing keywords and clearer outcomes."
  return "The CV likely needs more role-specific language and evidence."
})

const atsBand = computed(() => {
  if (!analysisReady.value) return "Waiting"
  if (atsScore.value >= 85) return "Strong fit"
  if (atsScore.value >= 70) return "Promising"
  return "Needs work"
})

const priorityGaps = computed(() => {
  if (!analysisReady.value) {
    return [
      "Upload a CV to see matched keywords.",
      "The optimiser will highlight missing evidence."
    ]
  }

  if (!missingKeywords.value.length) {
    return [
      "The CV already covers the main role terms.",
      "Use stronger measurable outcomes in bullet points."
    ]
  }

  return missingKeywords.value
    .slice(0, 3)
    .map((skill) => `Add clearer evidence for ${skill}.`)
})

function handleFiles(event) {
  files.value = Array.from(event.target.files || [])
}

function runAnalysis() {
  if (!canAnalyze.value) return
  analysisReady.value = true
}

function generateCoverLetter() {
  if (!analysisReady.value) return
  coverLetterReady.value = true
}

function applySuggestions() {
  if (!analysisReady.value) return
  cvReady.value = true
}

function jobTitle() {
  const firstLine = jobDescription.value.split("\n").find((line) => line.trim())
  return firstLine?.trim() || "target"
}
</script>
