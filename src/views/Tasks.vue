<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { Input } from '@/components/ui/input'
import { Button } from '@/components/ui/button'
import { Label } from '@/components/ui/label'
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogTrigger } from '@/components/ui/dialog'
import { Plus, Pencil, Trash2, LogOut, CheckCircle2, Clock, Loader2, ListTodo } from 'lucide-vue-next'

interface Task {
  _id: string
  title: string
  description: string
  status: 'pending' | 'in-progress' | 'completed'
  createdAt: string
}

const router = useRouter()
const tasks = ref<Task[]>([])
const loading = ref(false)
const isDialogOpen = ref(false)
const isEditing = ref(false)

// Form fields
const taskTitle = ref('')
const taskDescription = ref('')
const taskStatus = ref<'pending' | 'in-progress' | 'completed'>('pending')
const editingTaskId = ref('')

// Filter and pagination
const filterStatus = ref('')
const filterTitle = ref('')
const sortBy = ref('createdAt')
const sortOrder = ref('desc')
const page = ref(1)
const limit = ref(10)
const totalPages = ref(1)

const getAuthHeader = () => {
  const token = localStorage.getItem('token')
  return {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  }
}

const fetchTasks = async () => {
  loading.value = true
  try {
    const params = new URLSearchParams({
      page: page.value.toString(),
      limit: limit.value.toString(),
      sort: sortBy.value,
      order: sortOrder.value,
    })

    if (filterStatus.value) params.append('status', filterStatus.value)
    if (filterTitle.value) params.append('title', filterTitle.value)

    const response = await fetch(`https://backend-mileapp-fullstack-test.vercel.app/tasks?${params}`, {
      headers: getAuthHeader()
    })

    if (response.ok) {
      const data = await response.json()
      tasks.value = data.data
      totalPages.value = data.meta.totalPages
    } else if (response.status === 401) {
      localStorage.removeItem('token')
      router.push('/login')
    }
  } catch (err) {
    console.error('Failed to fetch tasks:', err)
  } finally {
    loading.value = false
  }
}

const openCreateDialog = () => {
  isEditing.value = false
  taskTitle.value = ''
  taskDescription.value = ''
  taskStatus.value = 'pending'
  isDialogOpen.value = true
}

const openEditDialog = (task: Task) => {
  isEditing.value = true
  editingTaskId.value = task._id
  taskTitle.value = task.title
  taskDescription.value = task.description
  taskStatus.value = task.status
  isDialogOpen.value = true
}

const handleSubmit = async () => {
  loading.value = true
  try {
    const url = isEditing.value
      ? `https://backend-mileapp-fullstack-test.vercel.app/tasks/${editingTaskId.value}`
      : 'https://backend-mileapp-fullstack-test.vercel.app/tasks'

    const method = isEditing.value ? 'PUT' : 'POST'

    const response = await fetch(url, {
      method,
      headers: getAuthHeader(),
      body: JSON.stringify({
        title: taskTitle.value,
        description: taskDescription.value,
        status: taskStatus.value
      })
    })

    if (response.ok) {
      isDialogOpen.value = false
      await fetchTasks()
    }
  } catch (err) {
    console.error('Failed to save task:', err)
  } finally {
    loading.value = false
  }
}

const deleteTask = async (id: string) => {
  if (!confirm('Are you sure you want to delete this task?')) return

  loading.value = true
  try {
    const response = await fetch(`https://backend-mileapp-fullstack-test.vercel.app/tasks/${id}`, {
      method: 'DELETE',
      headers: getAuthHeader()
    })

    if (response.ok) {
      await fetchTasks()
    }
  } catch (err) {
    console.error('Failed to delete task:', err)
  } finally {
    loading.value = false
  }
}

const handleLogout = () => {
  localStorage.removeItem('token')
  router.push('/login')
}

const applyFilters = () => {
  page.value = 1
  fetchTasks()
}

const changePage = (newPage: number) => {
  page.value = newPage
  fetchTasks()
}

const getStatusIcon = (status: string) => {
  switch (status) {
    case 'completed':
      return CheckCircle2
    case 'in-progress':
      return Loader2
    default:
      return Clock
  }
}

const getStatusColor = (status: string) => {
  switch (status) {
    case 'completed':
      return 'from-green-500/20 to-emerald-500/20 border-green-500/30 text-green-400'
    case 'in-progress':
      return 'from-blue-500/20 to-cyan-500/20 border-blue-500/30 text-blue-400'
    default:
      return 'from-yellow-500/20 to-orange-500/20 border-yellow-500/30 text-yellow-400'
  }
}

onMounted(() => {
  fetchTasks()
})
</script>

<template>
  <div class="min-h-screen bg-gradient-to-br from-slate-950 via-blue-950 to-slate-900 relative overflow-hidden">
    <!-- Animated background -->
    <div class="absolute inset-0 overflow-hidden">
      <div class="absolute top-0 right-0 w-[500px] h-[500px] bg-blue-500/5 rounded-full blur-3xl animate-pulse-slow"></div>
      <div class="absolute bottom-0 left-0 w-[600px] h-[600px] bg-purple-500/5 rounded-full blur-3xl animate-pulse-slow" style="animation-delay: 1.5s"></div>
    </div>

    <!-- Grid overlay -->
    <div class="absolute inset-0 bg-[linear-gradient(rgba(59,130,246,0.02)_1px,transparent_1px),linear-gradient(90deg,rgba(59,130,246,0.02)_1px,transparent_1px)] bg-[size:50px_50px]"></div>

    <div class="relative z-10 p-4 md:p-6 lg:p-8 max-w-7xl mx-auto">
      <!-- Header -->
      <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4 mb-8">
        <div>
          <h1 class="text-3xl md:text-4xl font-bold text-white flex items-center gap-3">
            <ListTodo class="w-8 h-8 text-blue-400" />
            My Tasks
          </h1>
          <p class="text-slate-400 mt-2">Manage your daily tasks</p>
        </div>
        <Button
          variant="outline"
          @click="handleLogout"
          class="bg-slate-800/50 border-slate-700 text-slate-300 hover:bg-red-500/10 hover:border-red-500/50 hover:text-red-400 transition-all duration-300"
        >
          <LogOut class="w-4 h-4 mr-2" />
          Logout
        </Button>
      </div>

      <!-- Filters -->
      <Card class="mb-6 bg-slate-900/50 backdrop-blur-xl border-slate-700/50 shadow-xl">
        <CardHeader>
          <CardTitle class="text-slate-200">
            Filters & Search
          </CardTitle>
        </CardHeader>
        <CardContent>
          <div class="grid grid-cols-1 md:grid-cols-4 gap-4">
            <div>
              <Label class="text-slate-300 mb-2">
                Search
              </Label>
              <Input
                v-model="filterTitle"
                placeholder="Search by title"
                class="bg-slate-800/50 border-slate-700 text-white placeholder:text-slate-500 focus:border-blue-500 focus:ring-blue-500/20 transition-all duration-200"
              />
            </div>
            <div>
              <Label class="text-slate-300 mb-2">
                Status
              </Label>
              <select
                v-model="filterStatus"
                class="flex h-10 w-full rounded-md border bg-slate-800/50 border-slate-700 text-white px-3 py-2 text-sm focus:border-blue-500 focus:ring-2 focus:ring-blue-500/20 transition-all duration-200"
              >
                <option value="">All</option>
                <option value="pending">Pending</option>
                <option value="in-progress">In Progress</option>
                <option value="completed">Completed</option>
              </select>
            </div>
            <div>
              <Label class="text-slate-300 mb-2">
                Sort By
              </Label>
              <select
                v-model="sortBy"
                class="flex h-10 w-full rounded-md border bg-slate-800/50 border-slate-700 text-white px-3 py-2 text-sm focus:border-blue-500 focus:ring-2 focus:ring-blue-500/20 transition-all duration-200"
              >
                <option value="createdAt">Created Date</option>
                <option value="title">Title</option>
              </select>
            </div>
            <div class="flex items-end">
              <Button
                @click="applyFilters"
                class="w-full bg-gradient-to-r from-blue-600 to-cyan-600 hover:from-blue-700 hover:to-cyan-700 text-white transition-all duration-200"
              >
                Apply
              </Button>
            </div>
          </div>
        </CardContent>
      </Card>

      <!-- Create Task Button -->
      <div class="mb-6">
        <Dialog v-model:open="isDialogOpen">
          <DialogTrigger as-child>
            <Button
              @click="openCreateDialog"
              class="bg-gradient-to-r from-blue-600 to-cyan-600 hover:from-blue-700 hover:to-cyan-700 text-white transition-all duration-200"
            >
              <Plus class="w-5 h-5 mr-2" />
              New Task
            </Button>
          </DialogTrigger>
          <DialogContent class="bg-slate-900 border-slate-700 text-white">
            <DialogHeader>
              <DialogTitle class="text-xl text-white">
                {{ isEditing ? 'Edit Task' : 'New Task' }}
              </DialogTitle>
            </DialogHeader>
            <form @submit.prevent="handleSubmit" class="space-y-4 mt-4">
              <div>
                <Label for="title" class="text-slate-300">Title</Label>
                <Input
                  id="title"
                  v-model="taskTitle"
                  required
                  class="bg-slate-800/50 border-slate-700 text-white placeholder:text-slate-500 focus:border-blue-500 focus:ring-blue-500/20 transition-all duration-200 mt-2"
                />
              </div>
              <div>
                <Label for="description" class="text-slate-300">Description</Label>
                <Input
                  id="description"
                  v-model="taskDescription"
                  required
                  class="bg-slate-800/50 border-slate-700 text-white placeholder:text-slate-500 focus:border-blue-500 focus:ring-blue-500/20 transition-all duration-200 mt-2"
                />
              </div>
              <div>
                <Label for="status" class="text-slate-300">Status</Label>
                <select
                  id="status"
                  v-model="taskStatus"
                  class="flex h-10 w-full rounded-md border bg-slate-800/50 border-slate-700 text-white px-3 py-2 text-sm focus:border-blue-500 focus:ring-2 focus:ring-blue-500/20 transition-all duration-200 mt-2"
                >
                  <option value="pending">Pending</option>
                  <option value="in-progress">In Progress</option>
                  <option value="completed">Completed</option>
                </select>
              </div>
              <Button
                type="submit"
                class="w-full bg-gradient-to-r from-blue-600 to-cyan-600 hover:from-blue-700 hover:to-cyan-700 text-white transition-all duration-200"
                :disabled="loading"
              >
                {{ isEditing ? 'Update' : 'Create' }}
              </Button>
            </form>
          </DialogContent>
        </Dialog>
      </div>

      <!-- Tasks List -->
      <div class="space-y-4">
        <Card
          v-for="(task, index) in tasks"
          :key="task._id"
          class="bg-slate-900/50 backdrop-blur-xl border-slate-700/50 shadow-xl hover:shadow-2xl hover:shadow-blue-500/10 transition-all duration-300 hover:scale-[1.01] hover:border-blue-500/30 animate-slide-up group"
          :style="{ animationDelay: `${index * 0.05}s` }"
        >
          <CardContent class="p-6">
            <div class="flex flex-col md:flex-row md:justify-between md:items-start gap-4">
              <div class="flex-1 space-y-3">
                <h3 class="text-xl font-semibold text-white group-hover:text-blue-400 transition-colors duration-300">
                  {{ task.title }}
                </h3>
                <p class="text-slate-400">{{ task.description }}</p>
                <div class="flex flex-wrap gap-3 items-center">
                  <span
                    :class="[
                      'px-3 py-1.5 text-xs rounded-full font-medium flex items-center gap-2 bg-gradient-to-r border backdrop-blur-sm transition-all duration-300 hover:scale-105',
                      getStatusColor(task.status)
                    ]"
                  >
                    <component :is="getStatusIcon(task.status)" class="w-3.5 h-3.5" :class="{ 'animate-spin': task.status === 'in-progress' }" />
                    {{ task.status }}
                  </span>
                  <span class="text-xs text-slate-500 flex items-center gap-1">
                    <Clock class="w-3.5 h-3.5" />
                    {{ new Date(task.createdAt).toLocaleDateString() }}
                  </span>
                </div>
              </div>
              <div class="flex gap-2">
                <Button
                  variant="outline"
                  size="icon"
                  @click="openEditDialog(task)"
                  class="bg-slate-800/50 border-slate-700 text-blue-400 hover:bg-blue-500/10 hover:border-blue-500/50 transition-all duration-300 hover:scale-110 hover:rotate-12"
                >
                  <Pencil class="w-4 h-4" />
                </Button>
                <Button
                  variant="destructive"
                  size="icon"
                  @click="deleteTask(task._id)"
                  class="bg-slate-800/50 border-slate-700 text-red-400 hover:bg-red-500/10 hover:border-red-500/50 transition-all duration-300 hover:scale-110 hover:rotate-12"
                >
                  <Trash2 class="w-4 h-4" />
                </Button>
              </div>
            </div>
          </CardContent>
        </Card>
      </div>

      <!-- Pagination -->
      <div v-if="totalPages > 1" class="flex justify-center gap-3 mt-8 animate-fade-in">
        <Button
          variant="outline"
          @click="changePage(page - 1)"
          :disabled="page === 1"
          class="bg-slate-800/50 border-slate-700 text-slate-300 hover:bg-blue-500/10 hover:border-blue-500/50 hover:text-blue-400 transition-all duration-300 disabled:opacity-30"
        >
          Previous
        </Button>
        <span class="flex items-center px-6 py-2 bg-slate-800/50 border border-slate-700 rounded-md text-slate-300">
          Page <span class="text-blue-400 font-semibold mx-2">{{ page }}</span> of <span class="text-blue-400 font-semibold ml-2">{{ totalPages }}</span>
        </span>
        <Button
          variant="outline"
          @click="changePage(page + 1)"
          :disabled="page === totalPages"
          class="bg-slate-800/50 border-slate-700 text-slate-300 hover:bg-blue-500/10 hover:border-blue-500/50 hover:text-blue-400 transition-all duration-300 disabled:opacity-30"
        >
          Next
        </Button>
      </div>

      <!-- Loading State -->
      <div v-if="loading && tasks.length === 0" class="text-center py-16 animate-pulse">
        <Loader2 class="w-12 h-12 text-blue-400 mx-auto mb-4 animate-spin" />
        <p class="text-slate-400">Loading tasks...</p>
      </div>

      <!-- Empty State -->
      <div v-if="!loading && tasks.length === 0" class="text-center py-16 animate-scale-in">
        <div class="w-24 h-24 bg-gradient-to-br from-blue-500/10 to-cyan-500/10 rounded-full flex items-center justify-center mx-auto mb-4 border border-blue-500/20">
          <CheckCircle2 class="w-12 h-12 text-blue-400" />
        </div>
        <p class="text-slate-400 text-lg mb-2">No tasks found</p>
        <p class="text-slate-500 text-sm">Create your first task to get started!</p>
      </div>
    </div>
  </div>
</template>
