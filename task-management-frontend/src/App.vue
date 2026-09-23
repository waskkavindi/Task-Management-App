<script setup>
import { ref, computed, onMounted } from 'vue'

const showForm = ref(false)

const tasks = ref([])

const taskTitle = ref('')
const taskDescription = ref('')
const taskDueDate = ref('')
const taskCompleted = ref(false)
const editingTaskId = ref(null)

const API_URL = 'https://localhost:7171/api/Tasks'

const loadTasks = async () => {
      try {
        const response = await fetch(API_URL)

        if (!response.ok) {
          throw new Error('Failed to load tasks')
        }

        tasks.value = await response.json()
      } catch (error) {
        console.error('Error loading tasks:', error)
      }
}

const addTask = async () => {
    try {
        const taskData = {
            title: taskTitle.value,
            description: taskDescription.value,
            dueDate: taskDueDate.value
                ? new Date(taskDueDate.value).toISOString()
                : new Date().toISOString(),
            isCompleted: taskCompleted.value
        }

        console.log('Sending task:', taskData)

        let response

        if (editingTaskId.value !== null) {
            // Update existing task
            response = await fetch(`${API_URL}/${editingTaskId.value}`, {
                method: 'PUT',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({
                    id: editingTaskId.value,
                    ...taskData
                })
            })
        } else {
            // Add new task
            response = await fetch(API_URL, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(taskData)
            })
        }

        if (!response.ok) {
            const errorText = await response.text()
            throw new Error(errorText || 'Failed to save task')
        }

        await loadTasks()

        // Clear form
        taskTitle.value = ''
        taskDescription.value = ''
        taskDueDate.value = ''
        taskCompleted.value = false
        editingTaskId.value = null
        showForm.value = false

        console.log('Task saved successfully!')

    } catch (error) {
        console.error('Error saving task:', error)
    }
}

const editTask = (task) => {
    editingTaskId.value = task.id

    taskTitle.value = task.title
    taskDescription.value = task.description
    taskDueDate.value = task.dueDate
        ? task.dueDate.substring(0, 10)
        : ''
    taskCompleted.value = task.isCompleted

    showForm.value = true
}

const showDeleteModal = ref(false)
const taskToDelete = ref(null)

const deleteTask = (task) => {
    taskToDelete.value = task
    showDeleteModal.value = true
}

const confirmDelete = async () => {
    if (!taskToDelete.value) return

    try {
        const response = await fetch(
            `${API_URL}/${taskToDelete.value.id}`,
            {
                method: 'DELETE'
            }
        )

        if (!response.ok) {
            throw new Error('Failed to delete task')
        }

        await loadTasks()

        showDeleteModal.value = false
        taskToDelete.value = null

        console.log('Task deleted successfully!')

    } catch (error) {
        console.error('Error deleting task:', error)
    }
}

const cancelDelete = () => {
    showDeleteModal.value = false
    taskToDelete.value = null
} 

const totalTasks = computed(() => tasks.value.length)

const completedTasks = computed(() =>
      tasks.value.filter(task => task.isCompleted).length
)

const pendingTasks = computed(() =>
      tasks.value.filter(task => !task.isCompleted).length
)

onMounted(() => {
      loadTasks()
})
</script>

<template>
    <div class="min-h-screen bg-gray-100">

        <!-- Header -->
        <header class="bg-white shadow-sm">
            <div class="max-w-6xl mx-auto px-4 sm:px-6 py-5">
                <h1 class="text-2xl font-bold text-gray-800">
                    Task Management App
                </h1>

                <p class="text-gray-500 mt-1">
                    Manage your tasks easily and efficiently
                </p>
            </div>
        </header>

        <!-- Main -->
        <main class="max-w-6xl mx-auto px-4 sm:px-6 py-6 sm:py-8">

            <!-- Dashboard Cards -->
            <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8">

                <!-- Total Tasks -->
                <div class="bg-white rounded-2xl border border-gray-200 p-6
               shadow-sm hover:shadow-lg hover:-translate-y-1
               transition-all duration-200">
                    <div class="flex items-center justify-between">

                        <div>
                            <p class="text-sm font-medium text-gray-500">
                                Total Tasks
                            </p>

                            <h2 class="text-4xl font-bold text-gray-800 mt-2">
                                {{ totalTasks }}
                            </h2>

                            <p class="text-xs text-gray-400 mt-2">
                                All your tasks
                            </p>
                        </div>

                        <div class="w-14 h-14 rounded-2xl bg-blue-100
                       flex items-center justify-center">
                            <span class="text-2xl">📋</span>
                        </div>

                    </div>
                </div>


                <!-- Completed -->
                <div class="bg-white rounded-2xl border border-gray-200 p-6
               shadow-sm hover:shadow-lg hover:-translate-y-1
               transition-all duration-200">
                    <div class="flex items-center justify-between">

                        <div>
                            <p class="text-sm font-medium text-gray-500">
                                Completed
                            </p>

                            <h2 class="text-4xl font-bold text-green-600 mt-2">
                                {{ completedTasks }}
                            </h2>

                            <p class="text-xs text-gray-400 mt-2">
                                Tasks completed
                            </p>
                        </div>

                        <div class="w-14 h-14 rounded-2xl bg-green-100
                       flex items-center justify-center">
                            <span class="text-2xl">✓</span>
                        </div>

                    </div>
                </div>


                <!-- Pending -->
                <div class="bg-white rounded-2xl border border-gray-200 p-6
               shadow-sm hover:shadow-lg hover:-translate-y-1
               transition-all duration-200">
                    <div class="flex items-center justify-between">

                        <div>
                            <p class="text-sm font-medium text-gray-500">
                                Pending
                            </p>

                            <h2 class="text-4xl font-bold text-orange-500 mt-2">
                                {{ pendingTasks }}
                            </h2>

                            <p class="text-xs text-gray-400 mt-2">
                                Tasks remaining
                            </p>
                        </div>

                        <div class="w-14 h-14 rounded-2xl bg-orange-100
                       flex items-center justify-center">
                            <span class="text-2xl">⏳</span>
                        </div>

                    </div>
                </div>

            </div>

            <!-- Task Section -->
            <div class="bg-white rounded-xl shadow-sm p-4 sm:p-6">

                <div class="flex flex-col md:flex-row md:items-center md:justify-between gap-4 mb-6">

                    <div>
                        <h2 class="text-xl font-bold text-gray-800">
                            My Tasks
                        </h2>

                        <p class="text-gray-500 text-sm mt-1">
                            View and manage your tasks
                        </p>
                    </div>

                    <button @click="showForm = true"
                            class="bg-blue-600 hover:bg-blue-700 text-white px-5 py-2.5 rounded-lg font-medium transition">
                        + Add Task
                    </button>

                </div>

                <!-- Add Task Form -->
                <div v-if="showForm"
                     class="mb-6 p-6 bg-gray-50 rounded-lg border">

                    <h3 class="text-lg font-semibold text-gray-800 mb-4">
                        Add New Task
                    </h3>
                    <div class="space-y-4">

                        <!-- Title -->
                        <input v-model="taskTitle"
                               type="text"
                               placeholder="Task title"
                               class="w-full border border-gray-300 rounded-lg px-4 py-2.5 focus:outline-none focus:ring-2 focus:ring-blue-500" />

                        <!-- Description -->
                        <textarea v-model="taskDescription"
                                  placeholder="Task description"
                                  rows="4"
                                  class="w-full border border-gray-300 rounded-lg px-4 py-2.5 focus:outline-none focus:ring-2 focus:ring-blue-500"></textarea>

                        <!-- Due Date -->
                        <input v-model="taskDueDate"
                               type="date"
                               class="w-full border border-gray-300 rounded-lg px-4 py-2.5" />

                        <!-- Status -->
                        <div>
                            <label class="block text-gray-700 font-medium mb-2">
                                Status
                            </label>

                            <select v-model="taskCompleted"
                                    class="w-full border border-gray-300 rounded-lg px-4 py-2.5 focus:outline-none focus:ring-2 focus:ring-blue-500">

                                <option :value="false">Pending</option>
                                <option :value="true">Completed</option>

                            </select>
                        </div>

                        <!-- Buttons -->
                        <div class="flex gap-3">

                            <button @click="addTask"
                                    class="bg-green-600 hover:bg-green-700 text-white px-5 py-2.5 rounded-lg font-medium">
                                {{ editingTaskId !== null ? 'Update Task' : 'Save Task' }}
                            </button>

                            <button @click="showForm = false"
                                    class="bg-gray-300 hover:bg-gray-400 text-gray-800 px-5 py-2.5 rounded-lg font-medium">
                                Cancel
                            </button>

                        </div>

                    </div>
                </div>

                <!-- Task List -->
                <div v-if="tasks.length > 0" class="space-y-4">

                    <div v-for="task in tasks"
                         :key="task.id"
                         class="bg-white border border-gray-200 rounded-xl p-5
               hover:shadow-md hover:border-blue-200
               transition duration-200">

                        <!-- Top Row -->
                        <div class="flex flex-col sm:flex-row sm:items-start
                    sm:justify-between gap-4">

                            <!-- Task Information -->
                            <div class="flex-1 min-w-0">

                                <!-- Title -->
                                <h3 class="text-lg font-semibold text-gray-800">
                                    {{ task.title || 'Untitled Task' }}
                                </h3>

                                <!-- Description -->
                                <p class="text-gray-500 text-sm mt-2">
                                    {{ task.description || 'No description' }}
                                </p>

                                <!-- Due Date -->
                                <p class="text-sm text-gray-400 mt-3">
                                    <span class="font-medium text-gray-500">
                                        Due:
                                    </span>

                                    {{
                                 task.dueDate !== '0001-01-01T00:00:00'
                                 ? new Date(task.dueDate).toLocaleDateString('en-US', {
                                     month: 'short',
                                     day: 'numeric',
                                     year: 'numeric'
                                     })
                                 : 'No due date'
                                    }}
                                </p>

                                <!-- Status -->
                                <div class="mt-3">
                                    <span class="inline-flex items-center px-3 py-1
                               rounded-full text-xs font-semibold"
                                          :class="task.isCompleted
                            ? 'bg-green-100 text-green-700'
                            : 'bg-orange-100 text-orange-700'">
                                        <span class="w-2 h-2 rounded-full mr-2"
                                              :class="task.isCompleted
                                ? 'bg-green-500'
                                : 'bg-orange-500'"></span>

                                        {{ task.isCompleted ? 'Completed' : 'Pending' }}
                                    </span>
                                </div>

                            </div>

                            <!-- Action Buttons -->
                            <div class="flex flex-col sm:flex-row gap-2 shrink-0">

                                <button @click="editTask(task)"
                                        class="px-3 py-1.5 text-sm font-medium
                           text-blue-600 bg-blue-50
                           hover:bg-blue-100
                           rounded-lg transition">
                                    Edit
                                </button>

                                <button @click="deleteTask(task)"
                                        class="px-3 py-1.5 text-sm font-medium
                           text-red-600 bg-red-50
                           hover:bg-red-100
                           rounded-lg transition">
                                    Delete
                                </button>

                            </div>

                        </div>

                    </div>

                </div>

                <!-- Empty State -->
                <div v-else
                     class="text-center py-12 border-2 border-dashed border-gray-200 rounded-lg">

                    <h3 class="text-lg font-semibold text-gray-700">
                        No tasks yet
                    </h3>

                    <p class="text-gray-500 mt-2">
                        Add your first task to get started.
                    </p>

                </div>

            </div>

        </main>
        <!-- Delete Confirmation Modal -->
        <div v-if="showDeleteModal"
             class="fixed inset-0 bg-black/40 flex items-center
           justify-center z-50 px-4">
            <div class="bg-white rounded-2xl shadow-xl
               w-full max-w-md p-6">

                <!-- Icon -->
                <div class="w-12 h-12 bg-red-100 rounded-full
                   flex items-center justify-center mx-auto">
                    <span class="text-2xl">🗑️</span>
                </div>

                <!-- Title -->
                <h2 class="text-xl font-bold text-gray-800
                   text-center mt-4">
                    Delete Task?
                </h2>

                <!-- Message -->
                <p class="text-gray-500 text-center mt-2">
                    Are you sure you want to delete
                    <span class="font-semibold text-gray-700">
                        "{{ taskToDelete?.title }}"
                    </span>?
                </p>

                <p class="text-sm text-gray-400 text-center mt-1">
                    This action cannot be undone.
                </p>

                <!-- Buttons -->
                <div class="flex justify-center gap-3 mt-6">

                    <button @click="cancelDelete"
                            class="px-5 py-2.5 rounded-lg
                       bg-gray-100 hover:bg-gray-200
                       text-gray-700 font-medium transition">
                        Cancel
                    </button>

                    <button @click="confirmDelete"
                            class="px-5 py-2.5 rounded-lg
                       bg-red-600 hover:bg-red-700
                       text-white font-medium transition">
                        Delete Task
                    </button>

                </div>

            </div>
        </div>
    </div>
</template>