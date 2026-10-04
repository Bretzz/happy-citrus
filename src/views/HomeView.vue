<!--
1. fetches /p (returns a json with a [] of strings)
2. displays the strings and add a link to /project for each one of them
3. fetch the metadata at /p/project
4. the link is a vue-router route to Project.vue passing the data as prop
-->
<script setup lang="ts">
import { ref, onMounted } from 'vue'

const projects = ref<string[]>([])
const loading = ref(true)
const error = ref('')

const URL = 'http://localhost:8080'

onMounted(async () => {
  try {
    const res = await fetch('http://localhost:8080/p')
    if (!res.ok) throw new Error('Failed to load projects')
    projects.value = await res.json()
  } catch (e: any) {
    error.value = e.message || 'Error fetching projects'
  } finally {
    loading.value = false
  }
})
</script>

<template>
  <div class="max-w-3xl mx-auto p-6 font-mono text-zinc-200">
    <!-- Header -->
    <div class="mb-6 pb-4 border-b border-zinc-800 flex items-center justify-between">
      <h1 class="text-2xl font-bold tracking-tight text-[#E3ACD3]">
        <span class="text-zinc-500 mr-2">$</span>projects
      </h1>
      <span class="text-xs px-2 py-0.5 rounded border border-zinc-800 bg-zinc-900 text-[#ACD3E3]">
        happy-citrus v1.0
      </span>
    </div>

    <!-- Loading / Error States -->
    <div v-if="loading" class="text-zinc-500 text-sm animate-pulse">
      &gt; fetching index...
    </div>

    <div v-else-if="error" class="text-red-400 bg-red-950/30 p-4 rounded-md border border-red-900/50 text-sm">
      [ERROR] {{ error }}
    </div>

    <!-- Projects List -->
    <div v-else class="border border-zinc-800 rounded-lg bg-zinc-950 overflow-hidden shadow-2xl">
      <div class="bg-zinc-900/80 px-4 py-2 border-b border-zinc-800 text-xs text-zinc-500 uppercase tracking-wider flex justify-between">
        <span>Directory</span>
        <span>Actions</span>
      </div>

      <ul class="divide-y divide-zinc-800/60">
        <li 
          v-for="proj in projects" 
          :key="proj" 
          class="flex items-center justify-between p-4 hover:bg-zinc-900/50 transition-colors group"
        >
          <router-link 
            :to="`/p/${proj}`" 
            class="text-base font-medium text-[#ACD3E3] group-hover:text-[#D3E3AC] transition-colors flex items-center"
          >
            <span class="text-zinc-600 font-normal">./</span>{{ proj }}
          </router-link>

          <a 
            :href="`${URL}/p/download?project=${encodeURIComponent(proj)}`"
            class="px-3 py-1 text-xs font-semibold rounded bg-[#D3E3AC]/10 text-[#D3E3AC] border border-[#D3E3AC]/30 hover:bg-[#D3E3AC] hover:text-zinc-950 transition-all"
          >
            ZIP
          </a>
        </li>

        <li v-if="projects.length === 0" class="p-8 text-center text-zinc-600 text-sm">
          No projects found.
        </li>
      </ul>
    </div>
  </div>
</template>