<script setup>
import { computed, ref, onMounted, onUnmounted } from 'vue'

const inputTask = ref('')
const tasks = ref([])

// State untuk melacak posisi scroll dengan optimasi requestAnimationFrame
const scrollY = ref(0)
let ticking = false

const handleScroll = () => {
  if (!ticking) {
    window.requestAnimationFrame(() => {
      scrollY.value = window.scrollY
      ticking = false
    })
    ticking = true
  }
}

onMounted(() => {
  window.addEventListener('scroll', handleScroll, { passive: true })
})

onUnmounted(() => {
  window.removeEventListener('scroll', handleScroll)
})

// Efek parallax yang lembut dan mulus
const backgroundTransform = computed(() => ({
  transform: `translateY(${scrollY.value * 0.15}px)`
}))

const activeTasks = computed(() => tasks.value.filter(task => !task.status))
const completedTasks = computed(() => tasks.value.filter(task => task.status))
const completion = computed(() => tasks.value.length
  ? Math.round((completedTasks.value.length / tasks.value.length) * 100)
  : 0)

const date = new Intl.DateTimeFormat('en-US', {
  weekday: 'long', day: 'numeric', month: 'short'
}).format(new Date())

const greeting = new Intl.DateTimeFormat('en-US', { hour: 'numeric' })
  .format(new Date())
  .includes('AM') ? 'Good morning' : 'Good afternoon'

function addTask() {
  const title = inputTask.value.trim()
  if (!title) return

  tasks.value.push({
    id: Date.now(),
    title,
    status: false
  })

  inputTask.value = ''
}

function toggleTask(id) {
  const task = tasks.value.find(task => task.id === id)
  if (task) task.status = !task.status
}

function deleteTask(id) {
  tasks.value = tasks.value.filter(task => task.id !== id)
}
</script>

<template>
  <main class="relative min-h-screen overflow-hidden px-4 py-8 text-[#0a192f] sm:px-8 lg:px-12">
    <!-- Layer Latar Belakang Parallax Menggunakan mountain-bg3.jpg -->
    <div 
      class="parallax-bg fixed inset-0 -top-[30vh] h-[160vh] w-full pointer-events-none -z-10 bg-cover bg-center bg-no-repeat"
      :style="[backgroundTransform, { backgroundImage: `url('/mountain-bg3.jpg')` }]"
    />

    <!-- Ambient Glow Orbs -->
    <div class="pointer-events-none absolute -left-24 top-20 h-72 w-72 rounded-full bg-white/45 blur-3xl -z-10" />
    <div class="pointer-events-none absolute -right-16 bottom-0 h-96 w-96 rounded-full bg-sky-300/40 blur-3xl -z-10" />

    <div class="relative mx-auto max-w-6xl">
      <header class="mb-8 flex items-center justify-between px-1">
        <!-- Logo Pojok Kiri Atas Menggunakan logo.jpe -->
        <div class="flex items-center gap-3">
          <img src="/logo.jpg" alt="Taskify Logo" class="h-11 w-11 rounded-2xl object-cover shadow-lg border border-white/60" />
          <div>
            <p class="text-xs font-bold uppercase tracking-[0.22em] text-slate-500">Your daily space</p>
            <h1 class="text-xl font-bold tracking-tight">Taskify</h1>
          </div>
        </div>

        <!-- Badge Pojok Kanan Atas -->
        <div class="hidden items-center gap-2 rounded-full border border-white/60 bg-white/35 px-4 py-2 text-sm font-semibold text-slate-600 backdrop-blur-md sm:flex shadow-sm">
          <span class="text-sky-600">✦</span>
          Stay in flow
        </div>
      </header>

      <div class="grid gap-5 lg:grid-cols-3">
        <section class="glass-tile p-7 lg:col-span-2">
          <p class="mb-10 text-sm font-semibold text-slate-500">{{ date }}</p>

          <div class="flex flex-col justify-between gap-6 sm:flex-row sm:items-end">
            <div>
              <p class="mb-2 text-base font-medium text-slate-500">{{ greeting }}, there.</p>
              <h2 class="max-w-lg text-4xl font-bold leading-tight tracking-[-0.04em] sm:text-5xl">
                Make space for what matters.
              </h2>
            </div>

            <div class="shrink-0 rounded-2xl bg-white/45 px-5 py-4 backdrop-blur-sm">
              <p class="text-3xl font-bold">{{ activeTasks.length }}</p>
              <p class="text-xs font-bold uppercase tracking-wider text-slate-500">Tasks remaining</p>
            </div>
          </div>
        </section>

        <section class="glass-tile flex min-h-[220px] flex-col justify-between p-7">
          <div class="flex items-start justify-between">
            <div>
              <p class="text-xs font-bold uppercase tracking-wider text-slate-500">Progress</p>
              <p class="mt-2 text-3xl font-bold">{{ completion }}%</p>
            </div>

            <div
              class="flex h-14 w-14 items-center justify-center rounded-full border-[5px] border-white/70 text-sm font-bold"
              :style="{ background: `conic-gradient(#0a192f ${completion * 3.6}deg, rgba(255,255,255,.4) 0deg)` }"
            >
              <span class="flex h-10 w-10 items-center justify-center rounded-full bg-sky-200/90">
                {{ completion }}%
              </span>
            </div>
          </div>

          <div>
            <div class="mb-2 flex justify-between text-xs font-semibold text-slate-500">
              <span>Daily momentum</span>
              <span>{{ completedTasks.length }} of {{ tasks.length }}</span>
            </div>
            <div class="h-2 overflow-hidden rounded-full bg-white/55">
              <div class="h-full rounded-full bg-[#0a192f] transition-all duration-500" :style="{ width: `${completion}%` }" />
            </div>
          </div>
        </section>

        <section class="glass-tile p-6 lg:col-span-3">
          <form class="flex flex-col gap-3 sm:flex-row" @submit.prevent="addTask">
            <label for="new-task" class="sr-only">New task</label>
            <input
              id="new-task"
              v-model="inputTask"
              autocomplete="off"
              placeholder="What would you like to accomplish?"
              class="min-w-0 flex-1 rounded-2xl border border-white/75 bg-white/55 px-5 py-4 text-base font-medium text-[#0a192f] outline-none placeholder:text-slate-500 focus:border-sky-500 focus:bg-white/75 focus:ring-4 focus:ring-sky-200/60"
            />
            <button type="submit" class="flex items-center justify-center gap-2 rounded-2xl bg-[#0a192f] px-7 py-4 font-bold text-white shadow-lg transition-all hover:-translate-y-0.5 hover:bg-slate-800 active:scale-95">
              <span class="text-xl leading-none">+</span>
              Add task
            </button>
          </form>
        </section>

        <!-- Active Tasks Section -->
        <section class="glass-tile p-6 lg:col-span-2">
          <div class="mb-5 flex items-center justify-between">
            <div>
              <p class="text-xs font-bold uppercase tracking-wider text-slate-500">Focus list</p>
              <h3 class="mt-1 text-2xl font-bold">Active tasks</h3>
            </div>
            <span class="rounded-full bg-white/55 px-3 py-1 text-xs font-bold text-slate-600">{{ activeTasks.length }}</span>
          </div>

          <p v-if="!activeTasks.length" class="rounded-2xl border border-dashed border-white/70 bg-white/20 px-4 py-8 text-center text-sm font-medium text-slate-500">
            Your list is clear. Add a task to begin.
          </p>

          <TransitionGroup v-else name="task-list" tag="ul" class="space-y-3 overflow-hidden">
            <li v-for="task in activeTasks" :key="task.id" class="group flex items-center gap-3 rounded-2xl border border-white/60 bg-white/35 p-3.5 transition-all hover:-translate-y-0.5 hover:bg-white/60 hover:shadow-lg">
              <button type="button" :aria-label="`Complete ${task.title}`" class="flex h-6 w-6 shrink-0 items-center justify-center rounded-full border-2 border-slate-400 text-transparent transition-colors hover:border-[#0a192f]" @click="toggleTask(task.id)">○</button>
              <span class="flex-1 text-sm font-semibold text-slate-800">{{ task.title }}</span>
              <button type="button" :aria-label="`Delete ${task.title}`" class="rounded-lg p-2 text-slate-400 opacity-0 transition-all hover:bg-red-100 hover:text-red-600 group-hover:opacity-100 focus:opacity-100" @click="deleteTask(task.id)">×</button>
            </li>
          </TransitionGroup>
        </section>

        <!-- Completed Tasks Section -->
        <section class="glass-tile p-6">
          <div class="mb-5">
            <p class="text-xs font-bold uppercase tracking-wider text-slate-500">Completed</p>
            <h3 class="mt-1 text-2xl font-bold">Done &amp; dusted</h3>
          </div>

          <p v-if="!completedTasks.length" class="rounded-2xl border border-dashed border-white/70 bg-white/20 px-4 py-8 text-center text-sm font-medium text-slate-500">
            Completed tasks will appear here.
          </p>

          <TransitionGroup v-else name="task-list" tag="ul" class="space-y-3 overflow-hidden">
            <li v-for="task in completedTasks" :key="task.id" class="group flex items-center gap-3 rounded-2xl border border-white/60 bg-white/35 p-3.5">
              <button type="button" :aria-label="`Mark ${task.title} active`" class="flex h-6 w-6 shrink-0 items-center justify-center rounded-full border-2 border-[#0a192f] bg-[#0a192f] text-white" @click="toggleTask(task.id)">✓</button>
              <span class="flex-1 text-sm font-semibold text-slate-500 line-through opacity-70">{{ task.title }}</span>
              <button type="button" :aria-label="`Delete ${task.title}`" class="rounded-lg p-2 text-slate-400 hover:bg-red-100 hover:text-red-600" @click="deleteTask(task.id)">×</button>
            </li>
          </TransitionGroup>
        </section>
      </div>
    </div>
  </main>
</template>

<style>
.glass-tile {
  border: 1px solid rgba(255, 255, 255, 0.4);
  border-radius: 1.5rem;
  background: rgba(255, 255, 255, 0.25);
  box-shadow: 0 18px 50px rgba(0, 0, 0, 0.15);
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
}

.parallax-bg {
  will-change: transform;
  transition: transform 0.1s cubic-bezier(0, 0, 0.2, 1);
}

.task-list-enter-active,
.task-list-leave-active {
  transition: all 0.35s cubic-bezier(0.4, 0, 0.2, 1);
}

.task-list-enter-from,
.task-list-leave-to {
  opacity: 0;
  transform: translateY(-12px) scale(0.96);
}

.task-list-move {
  transition: transform 0.35s cubic-bezier(0.4, 0, 0.2, 1);
}
</style>