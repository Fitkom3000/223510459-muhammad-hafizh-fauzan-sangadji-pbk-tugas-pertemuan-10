<template>
  <div id="app">
    <header>
      <nav>
        <ul>
          <li @click="switchView('todos')">Todos</li>
          <li @click="switchView('posts')">Post</li>
        </ul>
      </nav>
    </header>

    <div class="container">
      <div v-if="currentView === 'todos'">
        <h2>Todos</h2>
        <form @submit.prevent="addTodo">
          <input type="text" v-model="newTodo" placeholder="Tambah kegiatan baru" required>
          <button type="submit">Tambah</button>
        </form>
        <div v-for="(todo, index) in filteredTodos" :key="index">
          <!-- Menggunakan @click untuk updateTodo -->
          <input type="checkbox" v-model="todo.completed" @click="updateTodo(index)">
          <span :class="{ 'completed': todo.completed }">{{ todo.text }}</span>
          <button @click="removeTodo(index)">Remove</button>
        </div>
        <!-- Tombol filter untuk menampilkan hanya kegiatan yang belum selesai -->
        <button @click="filterTodos">{{ showCompleted ? 'Tampilkan Semua' : 'Filter Belum Selesai' }}</button>
      </div>
      <!-- Bagian lain dari aplikasi -->
      <div v-else-if="currentView === 'posts'">
        <h2>Posts</h2>
        <select v-model="selectedUser" @change="fetchPosts">
          <option v-for="user in users" :key="user.id" :value="user.id">{{ user.name }}</option>
        </select>

        <div v-if="loading">Loading...</div>
        <div v-else>
          <div v-for="post in posts" :key="post.id">
            <h3>{{ post.title }}</h3>
            <p>{{ post.body }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      todos: [
        { text: 'Susah banget', completed: false },
        { text: 'Woila', completed: false }
      ],
      newTodo: '',
      currentView: 'todos',
      showCompleted: false,
      users: [],
      selectedUser: null,
      posts: [],
      loading: false
    };
  },
  computed: {
    // Menggunakan computed property untuk memfilter kegiatan yang belum selesai
    filteredTodos() {
      if (this.showCompleted) {
        return this.todos;
      } else {
        return this.todos.filter(todo => !todo.completed);
      }
    }
  },
  mounted() {
    this.fetchUsers();
  },
  methods: {
    switchView(view) {
      this.currentView = view;
    },
    addTodo() {
      if (this.newTodo.trim() !== '') {
        this.todos.push({ text: this.newTodo, completed: false });
        this.newTodo = '';
      }
    },
    updateTodo(index) {
      // Memeriksa apakah sudah selesai, jika ya, ubah menjadi tidak selesai
      if (this.todos[index].completed) {
        this.todos[index].completed = false;
      } else {
        this.todos[index].completed = true;
      }
    },
    removeTodo(index) {
      this.todos.splice(index, 1);
    },
    // Mengubah status tombol filter dan menampilkan semua kegiatan
    filterTodos() {
      this.showCompleted = !this.showCompleted;
    },
    async fetchUsers() {
      this.loading = true;
      try {
        const response = await fetch('https://jsonplaceholder.typicode.com/users');
        this.users = await response.json();
      } catch (error) {
        console.error('Error fetching users:', error);
      } finally {
        this.loading = false;
      }
    },
    async fetchPosts() {
      this.loading = true;
      try {
        const response = await fetch(`https://jsonplaceholder.typicode.com/posts?userId=${this.selectedUser}`);
        this.posts = await response.json();
      } catch (error) {
        console.error('Error fetching posts:', error);
      } finally {
        this.loading = false;
      }
    }
  }
};
</script>

<style>
header {
  background-color: #333;
  color: white;
}

nav ul {
  list-style-type: none;
  display: flex;
}

nav ul li {
  padding: 10px;
  cursor: pointer;
}

nav ul li:hover {
  background-color: #555;
}

.container {
  margin-top: 20px;
}

.completed {
  text-decoration: line-through;
}
</style>
