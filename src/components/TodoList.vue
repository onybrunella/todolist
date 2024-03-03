<template>
  <div id="app">
    <h1>TO-DO LIST</h1>
    <div class="filters-container">
      <div class="select-container">
        <select v-model="filter" class="filter-select">
          <option value="all">All tasks</option>
          <option value="completed">Completed tasks</option>
          <option value="uncompleted">Uncompleted tasks</option>
        </select>
      </div>
      <div class="select-container">
        <select v-model="priorityFilter" class="filter-select">
          <option value="all">All priorities</option>
          <option value="low">Low priority</option>
          <option value="medium">Medium priority</option>
          <option value="high">High priority</option>
        </select>
      </div>
    </div>

      <div class="task-list-container">
        <ul>
          <li v-for="(task, index) in filteredTasks" :key="index" class="task-item">
            <span :class="{ completed: task.completed }" @click="showTaskDetails(task)">{{ task.name }}</span>
            <button @click="toggleTaskCompletion(index)" v-if="!task.completed" class="complete-btn">Complete</button>
          </li>
        </ul>
      </div>

    <div class="container">
      <div class="input-container">
        <input v-model="newTaskName" type="text" placeholder="Task Name" @keyup.enter="addTask">
        <input v-model="newTaskDescription" type="text" placeholder="Task description" @keyup.enter="addTask">
        <input v-model="newTaskDueDate" type="date" placeholder="Due date" @keyup.enter="addTask">
        <input v-model="newTaskDueTime" type="time" placeholder="Due time" @keyup.enter="addTask">
        <select v-model="newTaskPriority" @keyup.enter="addTask" class="filter-select">
          <option value="low">Low</option>
          <option value="medium">Medium</option>
          <option value="high">High</option>
        </select>
        <select v-model="newTaskCategory" @keyup.enter="addTask" class="filter-select">
          <option value="general">General</option>
          <option value="work">Work</option>
          <option value="personal">Personal</option>
        </select>
        <button @click="addTask" class="add-btn" alt="Add the task"><ion-icon name="add-outline"></ion-icon></button>
      </div>

      <div class="task-details" v-if="selectedTask">
          <h2>{{ selectedTask.name }}</h2>
          <p>Description: {{ selectedTask.description }}</p>
          <p>Due Date: {{ selectedTask.dueDate }}</p>
          <p>Due Time: {{ selectedTask.dueTime }}</p>
          <p>Priority: {{ selectedTask.priority }}</p>
          <p>Category: {{ selectedTask.category }}</p>
          <button @click="closeTaskDetails" class="close-btn">Close</button>
      </div>
      
      <ul>
        <li v-for="(task, index) in tasks" :key="index" class="task-item-actions">
          <span>{{ task.name }}
            <button @click="toggleTaskCompletion(index)" class="complete-btn">
              <ion-icon :name="task.completed ? 'checkmark-circle-outline' : 'ellipse-outline'"></ion-icon>
            </button>
          </span>
          <div class="actions">
            <button @click="showEditForm(index)" class="edit-btn"><ion-icon name="pencil-outline"></ion-icon></button>
            <button @click="showDeleteConfirmation(index)" class="delete-btn"><ion-icon name="trash-outline"></ion-icon></button>
          </div>
          <div v-if="editingTaskIndex === index">
            <form @submit.prevent="saveEditedTask(index)" class="edit-form">
              <input v-model="editedTask.name" type="text">
              <input v-model="editedTask.description" type="text">
              <input v-model="editedTask.dueDate" type="date">
              <input v-model="editedTask.dueTime" type="time">
              <select v-model="editedTask.priority">
                <option value="low">Low</option>
                <option value="medium">Medium</option>
                <option value="high">High</option>
              </select>
              <select v-model="editedTask.category">
                <option value="general">General</option>
                <option value="work">Work</option>
                <option value="personal">Personal</option>
              </select>
              <button type="submit" @click="showEditConfirmation(index)">Save</button>
            </form>
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      filter: 'all',
      priorityFilter: 'all',
      newTaskName: '',
      newTaskDescription: '',
      newTaskDueDate: '',
      newTaskDueTime: '',
      newTaskPriority: 'low',
      newTaskCategory: 'general',
      tasks: [],
      selectedTask: null,
      editedTask: {
        name: '',
        description: '',
        dueDate: '',
        dueTime: '',
        priority: 'low',
        category: 'general'
      },
      editingTaskIndex: null
    };
  },
  computed: {
    filteredTasks() {
    let filtered = this.tasks;
    if (this.filter === 'completed') {
      filtered = filtered.filter(task => task.completed);
    } else if (this.filter === 'uncompleted') {
      filtered = filtered.filter(task => !task.completed);
    }
    // Apply priority filter
    if (this.priorityFilter !== 'all') {
      filtered = filtered.filter(task => task.priority === this.priorityFilter);
    }
    return filtered;
  },
  },
  mounted() {
    const savedTasks = localStorage.getItem('tasks');
    if (savedTasks) {
      this.tasks = JSON.parse(savedTasks);
    }
  },
  watch: {
    tasks: {
      handler(tasks) {
        localStorage.setItem('tasks', JSON.stringify(tasks));
      },
      deep: true,
    },
  },
  created() {
    this.loadScript('https://cdn.jsdelivr.net/npm/vue@2');
    this.loadScript('https://unpkg.com/ionicons@7.1.0/dist/ionicons/ionicons.js');
  },
  methods: {
    addTask() {
      if (this.newTaskName.trim() === '') {
        return;
      }
      if (this.tasks.some((task) => task.name === this.newTaskName.trim())) {
        alert('Cette tâche existe déjà !');
        return;
      }
      this.tasks.push({
        name: this.newTaskName.trim(),
        description: this.newTaskDescription.trim(),
        completed: false,
        dueDate: this.newTaskDueDate + ' ' + this.newTaskDueTime,
        priority: this.newTaskPriority,
        category: this.newTaskCategory,
      });
      this.resetTaskFields();
    },
    resetTaskFields() {
      this.newTaskName = '';
      this.newTaskDescription = '';
      this.newTaskDueDate = '';
      this.newTaskDueTime = '';
      this.newTaskPriority = 'low';
      this.newTaskCategory = 'general';
    },
    showTaskDetails(task) {
      this.selectedTask = task;
    },
    closeTaskDetails() {
      this.selectedTask = null;
    },
    editTask(index) {
      const newTaskName = prompt('Modifier la tâche :', this.tasks[index].name);
      if (newTaskName === null || newTaskName === '') {
        return;
      }
      // this.$set(this.tasks, index, { name: newTaskName, completed: this.tasks[index].completed });
      this.tasks[index].name = newTaskName;
    },
    showEditForm(index) {
      this.editingTaskIndex = index;
      this.editedTask = { ...this.tasks[index] };
    },
    saveEditedTask(index) {
      if (this.editedTask.name.trim() === "") {
    return;
  }
  this.tasks[index].name = this.editedTask.name;
  this.tasks[index].description = this.editedTask.description;
  this.tasks[index].dueDate = this.editedTask.dueDate;
  this.tasks[index].dueTime = this.editedTask.dueTime;
  this.tasks[index].priority = this.editedTask.priority;
  this.tasks[index].category = this.editedTask.category;

  this.editedTask = { name: '', description: '', dueDate: '', dueTime: '', priority: 'low', category: 'general' };
  this.editingTaskIndex = null;
},
    toggleTaskCompletion(index) {
      this.tasks[index].completed = !this.tasks[index].completed;
    },
    deleteTask(index) {
      this.tasks.splice(index, 1);
    },
    showEditConfirmation(index) {
    if (confirm("Are you sure you want to edit this task?")) {
      this.saveEditedTask(index);
    }
  },
  showDeleteConfirmation(index) {
    if (confirm("Are you sure you want to delete this task?")) {
      this.deleteTask(index);
    }
  },
    loadScript(src,) {
      return new Promise((resolve, reject) => {
        const script = document.createElement('script');
        script.src = src;
        script.onload = resolve;
        script.onerror = reject;
        document.head.appendChild(script);
      });
    }
  }
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Quicksand:wght@300..700&family=Roboto+Mono:ital,wght@0,100..700;1,100..700&display=swap');
#app {
  font-family: "Quicksand", sans-serif !important;
  align-items: center;
  margin-top: 60px;
  display: flex;
  flex-direction: column;
}

body {
  font-family: "Quicksand", sans-serif !important;
}

h1 {
  color: #4a4e69;
  margin-bottom: 30px;
  margin-top: -50px;
}

.container {
  max-width: 400px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
}

.input-container {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.input-container input,
.input-container select {
  padding: 10px;
  border: 1px solid #42b983;
  border-radius: 5px;
  width: calc(50% - 5px);
  box-sizing: border-box;
}

.input-container .add-btn {
  padding: 10px 20px;
  border: none;
  background-color: #42b983;
  color: white;
  border-radius: 5px;
  cursor: pointer;
  width: 100%;
}

.input-container .add-btn:hover {
  background-color: #36a07e;
}

.filters-container {
  display: flex;
  justify-content: space-between;
}

.select-container {
  flex-grow: 1; 
  margin-right: 10px;
}

.select-container select {
  width: 100%;
  padding: 10px;
  border: 1px solid #42b983;
  border-radius: 5px;
  box-sizing: border-box;
}

.task-list-container {
  margin-top: 20px; 
}

.filter-select {
  padding: 8px 12px;
  border-radius: 5px;
  border: 1px solid #42b983;
}

ul {
  list-style-type: none;
  padding: 0;
  margin: 0;
}

.task-item {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
  background-color: #f9f9f9;
  padding: 10px;
  border-radius: 5px;
  cursor: pointer;
}

.task-item span {
  flex-grow: 1;
}

.task-item-actions {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 10px;
  background-color: #f9f9f9;
  padding: 10px;
  border-radius: 5px;
  margin-top: 10px;
}

.actions button {
  margin-left: 5px;
  border: none;
  background: none;
  cursor: pointer;
}

.actions button:hover {
  color: #42b983;
}

.actions .edit-btn:hover ion-icon[name='pencil-outline'] {
  color: #42b983;
}

.actions .delete-btn:hover ion-icon[name='trash-outline'] {
  color: #f44336;
}

.completed {
  text-decoration: line-through;
}

select {
  padding: 8px 12px;
  border-radius: 5px;
  border: 1px solid #42b983;
}

button {
  padding: 8px 16px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.complete-btn {
  color: #9b2226;
  border: none;
  background: none;
  cursor: pointer;
}

.complete-btn:hover {
  background-color: #bb3e03;
  color: #fefae0;
}

form button[type="submit"] {
  background-color: #42b983;
  color: white;
}

form button[type="submit"]:hover {
  background-color: #5cb85c;
}

.task-item {
  transition: transform 0.3s ease;
}

.task-item:hover {
  transform: scale(1.02); 
}

ion-icon {
  font-size: 1.2rem;
}

.actions button {
  padding: 6px;
}

.task-item {
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

ul {
  margin-bottom: 20px;
}

.edit-form {
  max-width: 400px;
  margin: 0 auto;
}

.edit-form input[type="text"],
.edit-form input[type="date"],
.edit-form input[type="time"],
.edit-form select {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #42b983;
  border-radius: 5px;
  box-sizing: border-box;
}

.edit-form select {
  appearance: none;
  -webkit-appearance: none;
  -moz-appearance: none;
  padding-right: 30px;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='8' viewBox='0 0 12 8'%3E%3Cpath fill='%23000' d='M6 8L0 0h12L6 8z'/%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position-x: calc(100% - 12px);
  background-position-y: center;
  background-size: 12px 8px;
}

.edit-form button[type="submit"] {
  padding: 10px 20px;
  border: none;
  background-color: #42b983;
  color: white;
  border-radius: 5px;
  cursor: pointer;
}

.edit-form button[type="submit"]:hover {
  background-color: #36a07e;
}

.task-details {
  max-width: 400px;
  margin: 0 auto;
  padding: 20px;
  background-color: #fff;
  border-radius: 5px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.task-details h2 {
  color: #42b983;
  margin-bottom: 10px;
}

.task-details p {
  color: #666;
  margin-bottom: 5px;
}

.task-details button {
  padding: 8px 16px;
  border: none;
  background-color: #9d4edd;
  color: white;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s ease;
  margin-top: 10px;
}

.task-details button:hover {
  background-color: #5a189a;
}

.task-details {
  margin-top: 10px;
  background-color: #f0f0f0;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
}

.task-details h2 {
  font-size: 20px;
  margin-bottom: 10px;
}

.task-details p {
  margin-bottom: 8px;
}

.task-details button {
  width: 100%;
}
.filters-container select {
  font-family: "Quicksand", sans-serif !important;
}

.filter-select, input {
  font-family: "Quicksand", sans-serif !important;
}

</style>
