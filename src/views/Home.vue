<template>
    <AddTask v-show="showAddTask" @add-task="addTask" />

    <div v-show="loading" class="spin">
        <i class="fas fa-spinner fa-spin"></i>
    </div>

    <Tasks @toggle-reminder="toggleReminder" @delete-task="deleteTask" :tasks="tasks" />

</template>

<script>
import Tasks from '../components/Tasks';
import AddTask from '../components/AddTask';

export default {
    name: 'Home',
    components: {
        Tasks,
        AddTask,
    },
    props: {
        showAddTask: {
            type: Boolean,
        }
    },
    data() {
        return {
            loading: false,
            tasks: [],
        }
    },
    async created() {
        this.loading = true;
        this.tasks = await this.fetchTasks();
        this.loading = false;
    },
    methods: {
        async fetchTasks() {
            const res = await fetch('api/tasks');
            const data = await res.json();
            return data;
        },
        async fetchTask(taskId) {
            const res = await fetch(`api/tasks/${taskId}`);
            const data = await res.json();
            return data;
        },
        async addTask(newTask) {
            const res = await fetch('api/tasks', {
                method: 'POST',
                headers: {
                    'Content-type': 'application/json',
                },
                body: JSON.stringify(newTask),
            });

            const data = await res.json();

            this.tasks = [...this.tasks, data];
        },
        async toggleReminder(taskId) {
            const taskToToggle = await this.fetchTask(taskId);
            const updTask = { ...taskToToggle, reminder: !taskToToggle.reminder };

            const res = await fetch(`api/tasks/${taskId}`, {
                method: 'PUT',
                headers: {
                    'Content-type': 'application/json',
                },
                body: JSON.stringify(updTask)
            });

            const data = await res.json();

            this.tasks = this.tasks.map((task) => task.id === taskId ? { ...task, reminder: data.reminder } : task);
        },
        async deleteTask(taskId) {
            if (confirm('Are you sure?')) {
                const res = await fetch(`api/tasks/${taskId}`, {
                    method: 'DELETE',
                });

                res.status === 200
                    ? (this.tasks = this.tasks.filter((task) => task.id !== taskId))
                    : alert('Error deleting task!');
            }
        },
    }
}
</script>

<style scoped>
.spin {
    padding-top: 30px;
    text-align: center;
    font-size: 60px;
}
</style>