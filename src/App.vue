<script setup>
import { RouterLink, RouterView } from 'vue-router'
</script>

<template>
  <header>
    <nav class="navbar">
      <ul class="navbar-list">
        <li><RouterLink to="/">Home</RouterLink></li>
        <li><RouterLink to="/login">Login</RouterLink></li>
      </ul>
    </nav>
  </header>

  <main>
    <RouterView />
  </main>
</template>

<style scoped>
/* Стили для навбара */
.navbar {
  background-color: #333;
  padding: 1rem;
}

.navbar-list {
  display: flex;
  list-style: none;
  margin: 0;
  padding: 0;
}

.navbar-list li {
  margin-right: 1rem;
}

.navbar-list li a {
  color: white;
  text-decoration: none;
}

.navbar-list li a:hover {
  text-decoration: underline;
}
</style>
