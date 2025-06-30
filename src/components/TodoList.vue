<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';

const todos = ref([]);
const newTodo = ref('');
const loading = ref(false);
const error = ref('');

const editId = ref(null);
const editTitle = ref('');

const API_URL = 'http://localhost:3000/todos';

const loadTodos = async () => {
  loading.value = true;
  error.value = '';
  try {
    const response = await axios.get(API_URL);
    todos.value = response.data;
  } catch (err) {
    error.value = 'Gagal memuat data todo.';
    console.error(err);
  } finally {
    loading.value = false;
  }
};

const addTodo = async () => {
  if (!newTodo.value.trim()) return;

  loading.value = true;
  error.value = '';
  try {
    const newData = {
      title: newTodo.value,
      completed: false,
    };
    const response = await axios.post(API_URL, newData);
    todos.value.push(response.data);
    newTodo.value = '';
  } catch (err) {
    error.value = 'Gagal menambah todo.';
    console.error(err);
  } finally {
    loading.value = false;
  }
};

const toggleCompleted = async (todo) => {
  loading.value = true;
  error.value = '';
  try {
    const response = await axios.put(`${API_URL}/${todo.id}`, {
      ...todo,
      completed: !todo.completed,
    });
    const idx = todos.value.findIndex(t => t.id === todo.id);
    if (idx !== -1) todos.value[idx] = response.data;
  } catch (err) {
    error.value = 'Gagal memperbarui todo.';
    console.error(err);
  } finally {
    loading.value = false;
  }
};

const deleteTodo = async (id) => {
  loading.value = true;
  error.value = '';
  try {
    await axios.delete(`${API_URL}/${id}`);
    todos.value = todos.value.filter(todo => todo.id !== id);
  } catch (err) {
    error.value = 'Gagal menghapus todo.';
    console.error(err);
  } finally {
    loading.value = false;
  }
};

const startEdit = (todo) => {
  editId.value = todo.id;
  editTitle.value = todo.title;
};

const cancelEdit = () => {
  editId.value = null;
  editTitle.value = '';
};

const updateTodo = async (todo) => {
  if (!editTitle.value.trim()) return;

  loading.value = true;
  error.value = '';
  try {
    const response = await axios.put(`${API_URL}/${todo.id}`, {
      ...todo,
      title: editTitle.value,
    });
    const idx = todos.value.findIndex(t => t.id === todo.id);
    if (idx !== -1) todos.value[idx] = response.data;
    editId.value = null;
    editTitle.value = '';
  } catch (err) {
    error.value = 'Gagal memperbarui todo.';
    console.error(err);
  } finally {
    loading.value = false;
  }
};

onMounted(loadTodos);
</script>
<template>
  <div class="todo-app">
    <h1>SISTEM</h1>
    <h1>toDoList Menggunakan axios dan Json-Server<h2>(CRUD)</h2></h1>
    <h5>Laras-233510656</h5>

    <!-- Form tambah todo baru -->
    <form @submit.prevent="addTodo">
      <input v-model="newTodo" placeholder="Tambah todo baru..." required />
      <button type="submit" :disabled="loading">Tambah</button>
    </form>

    <p v-if="loading">Memuat data...</p>
    <p v-if="error" style="color: red;">{{ error }}</p>

    <ul v-if="todos.length">
      <li v-for="todo in todos" :key="todo.id">
        <input
          type="checkbox"
          :checked="todo.completed"
          @change="toggleCompleted(todo)"
        />

        <!-- Tampilan normal -->
        <template v-if="editId !== todo.id">
          <span :style="{ textDecoration: todo.completed ? 'line-through' : 'none' }">
            {{ todo.title }}
          </span>
          <button @click="startEdit(todo)">Edit</button>
        </template>

        <!-- Form edit -->
        <template v-else>
          <input v-model="editTitle" />
          <button @click="updateTodo(todo)">Simpan</button>
          <button @click="cancelEdit">Batal</button>
        </template>

        <button @click="deleteTodo(todo.id)">Hapus</button>
      </li>
    </ul>

    <p v-else>Belum ada  ToDoList.</p>
  </div>
</template>

<style scoped>
.todo-app {
  max-width: 500px;
  margin: 50px auto;
  padding: 30px;
  background-color: #e6f0fa;
  border-radius: 12px;
  box-shadow: 0 0 10px rgba(0, 122, 255, 0.2);
  font-family: 'Segoe UI', sans-serif;
}

h1 {
  color: #1b4f72;
  text-align: center;
  margin-bottom: 20px;
  font-size: 22px;
}

h1::after {
  
  display: block;
  font-size: 14px;
  color: #2c3e50;
  margin-top: 8px;
}
h5 {
  text-align: center;
  color: #34495e;
  font-size: 16px;
  margin-bottom: 20px;
}

form {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

input[type="text"] {
  flex-grow: 1;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 6px;
  font-size: 14px;
}

button {
  padding: 10px 14px;
  background-color: #3498db;
  border: none;
  border-radius: 6px;
  color: white;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

button:hover {
  background-color: #2980b9;
}

ul {
  list-style: none;
  padding-left: 0;
}

li {
  display: flex;
  align-items: center;
  background-color: white;
  padding: 10px;
  border-radius: 6px;
  margin-bottom: 10px;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.05);
}

li input[type="checkbox"] {
  margin-right: 10px;
  transform: scale(1.2);
}

li span {
  flex-grow: 1;
  font-size: 15px;
  margin-left: 4px;
}

p {
  text-align: center;
  color: #2c3e50;
  margin-top: 10px;
  font-size: 14px;
}

p[style*="red"] {
  color: #e74c3c !important;
}
</style>