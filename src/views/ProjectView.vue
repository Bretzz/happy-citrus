<!--
```json
    {
        "description": "Brief description",         // Description of the project
        "resources": [                              // Array of resources
            {
                "filename": "secret code",          // name of the file
                "time_of_upload": "tomorrow",       // when was uploaded
                "size": "so fat (bytes)"               // size of the file (number)
            }
        ],
        "res_count": "it's over 9000",              // how many resources in the project (number)
        "project_size": "really fat (bytes)"        // size of the whole project (number)
    }
```
-->
<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { useRoute } from 'vue-router'

interface Resource {
  filename: string
  time_of_upload: string
  size: number | string
}

interface ProjectData {
  description: string
  resources: Resource[]
  res_count: number | string
  project_size: number | string
}

const route = useRoute()
const projectName = route.params.project as string

const project = ref<ProjectData | null>(null)
const loading = ref(true)
const error = ref('')

const formatBytes = (bytes: number | string) => {
  const num = Number(bytes)
  if (isNaN(num) || num === 0) return '0 B'
  const k = 1024
  const sizes = ['B', 'KB', 'MB', 'GB']
  const i = Math.floor(Math.log(num) / Math.log(k))
  return `${(num / Math.pow(k, i)).toFixed(1)} ${sizes[i]}`
}

onMounted(async () => {
  try {
    const res = await fetch(`http://localhost:8080/p/${projectName}`)
    if (!res.ok) throw new Error('Project not found')
    project.value = await res.json()
  } catch (e: any) {
    error.value = e.message || 'Error loading project'
  } finally {
    loading.value = false
  }
})
</script>

<template>
  <div class="max-w-4xl mx-auto p-6 font-mono text-zinc-200 space-y-6">
    <!-- Back Button -->
    <router-link to="/" class="inline-flex items-center text-xs text-zinc-500 hover:text-[#ACD3E3] transition-colors gap-1">
      cd ..
    </router-link>

    <div v-if="loading" class="text-zinc-500 text-sm animate-pulse">
      &gt; reading project structure...
    </div>

    <div v-else-if="error" class="text-red-400 bg-red-950/30 p-4 rounded-md border border-red-900/50 text-sm">
      [ERROR] {{ error }}
    </div>

    <template v-else-if="project">
      <!-- Header / Metadata -->
      <header class="bg-zinc-950 border border-zinc-800 rounded-lg p-5 flex flex-col md:flex-row md:items-center justify-between gap-4">
        <div>
          <h1 class="text-xl font-bold text-[#D3E3AC] flex items-center gap-2">
            <span class="text-zinc-600 font-normal">/p/</span>{{ projectName }}
          </h1>
          <p class="text-zinc-400 text-xs mt-1">{{ project.description }}</p>
        </div>

        <div class="flex items-center gap-3">
          <div class="bg-zinc-900 border border-zinc-800 px-3 py-1.5 rounded text-center">
            <span class="block text-[10px] text-zinc-500 uppercase">Files</span>
            <span class="font-bold text-[#E3ACD3] text-sm">{{ project.res_count }}</span>
          </div>
          <div class="bg-zinc-900 border border-zinc-800 px-3 py-1.5 rounded text-center">
            <span class="block text-[10px] text-zinc-500 uppercase">Size</span>
            <span class="font-bold text-[#ACD3E3] text-sm">{{ formatBytes(project.project_size) }}</span>
          </div>
          <a 
            :href="`http://localhost:8080/p/download?project=${encodeURIComponent(projectName)}`"
            class="px-3 py-2 text-xs font-semibold rounded bg-[#D3E3AC] text-zinc-950 hover:bg-[#D3E3AC]/80 transition-colors"
          >
            ZIP
          </a>
        </div>
      </header>

      <!-- Resource Table -->
      <div class="bg-zinc-950 border border-zinc-800 rounded-lg overflow-hidden shadow-2xl">
        <table class="w-full text-left text-xs">
          <thead class="bg-zinc-900/80 border-b border-zinc-800 text-zinc-500 uppercase">
            <tr>
              <th class="py-2.5 px-4">Filename</th>
              <th class="py-2.5 px-4">Uploaded</th>
              <th class="py-2.5 px-4">Size</th>
              <th class="py-2.5 px-4 text-right">Action</th>
            </tr>
          </thead>
          <tbody class="divide-y divide-zinc-800/60">
            <tr v-for="res in project.resources" :key="res.filename" class="hover:bg-zinc-900/50 transition-colors">
              <td class="py-3 px-4 text-[#ACD3E3] font-medium">{{ res.filename }}</td>
              <td class="py-3 px-4 text-zinc-400">{{ res.time_of_upload }}</td>
              <td class="py-3 px-4 text-[#E3ACD3]">{{ formatBytes(res.size) }}</td>
              <td class="py-3 px-4 text-right">
                <a 
                  :href="`http://localhost:8080/p/${projectName}/${res.filename}`" 
                  target="_blank"
                  class="px-2.5 py-1 rounded bg-zinc-900 border border-zinc-800 text-zinc-300 hover:border-[#ACD3E3]/50 hover:text-[#ACD3E3] transition-colors"
                >
                  GET ↗
                </a>
              </td>
            </tr>
            <tr v-if="!project.resources || project.resources.length === 0">
              <td colspan="4" class="py-8 text-center text-zinc-600">No resources available.</td>
            </tr>
          </tbody>
        </table>
      </div>
    </template>
  </div>
</template>