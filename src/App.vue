<template>
  <div class="app-container">
    <h2>📋 Feladatkezelő (Laravel + Vue REST API)</h2>

    <!-- Új feladat hozzáadása űrlap -->
    <section class="card">
      <h3>Új feladat hozzáadása</h3>
      <form @submit.prevent="addTask" class="task-form">
        <input v-model="newTask.title" type="text" placeholder="Feladat megnevezése..." required />
        <textarea v-model="newTask.description" placeholder="Részletek (opcionális)..."></textarea>
        
        <div class="form-row">
          <label>Prioritás: </label>
          <select v-model="newTask.priority">
            <option value="low">Alacsony</option>
            <option value="medium">Közepes</option>
            <option value="high">Magas</option>
          </select>
          <button type="submit" :disabled="loading">Hozzáadás</button>
        </div>
      </form>
    </section>

    <!-- Szűrők -->
    <div class="filters">
      <button @click="filterStatus = 'all'" :class="{ active: filterStatus === 'all' }">Összes</button>
      <button @click="filterStatus = 'pending'" :class="{ active: filterStatus === 'pending' }">Folyamatban</button>
      <button @click="filterStatus = 'completed'" :class="{ active: filterStatus === 'completed' }">Kész</button>
    </div>

    <!-- Feladatok listája -->
    <section class="task-list">
      <p v-if="filteredTasks.length === 0" class="empty-list">Nincs a szűrésnek megfelelő feladat.</p>
      
      <div v-for="task in filteredTasks" :key="task.id" class="task-item" :class="{ completed: task.is_completed }">
        <div class="task-content">
          <input type="checkbox" :checked="task.is_completed" @change="toggleTaskStatus(task)" />
          <div>
            <h4>{{ task.title }}</h4>
            <p v-if="task.description">{{ task.description }}</p>
            <span class="badge" :class="task.priority">{{ task.priority.toUpperCase() }}</span>
          </div>
        </div>
        <button @click="deleteTask(task.id)" class="delete-btn">Törlés</button>
      </div>
    </section>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import axios from 'axios';

// Axios alapértelmezett beállítása (írd át, ha a Laravel más címen fut)
const API_URL = 'https://teszt_laravel-task-api.wasmer.app';


const tasks = ref([]);
const filterStatus = ref('all');
const loading = ref(false);

const newTask = ref({
  title: '',
  description: '',
  priority: 'medium'
});

// 1. Feladatok betöltése a Laravelből (Read)
const fetchTasks = async () => {
  try {
    const response = await axios.get(API_URL);
    tasks.value = response.data;
  } catch (error) {
    console.error("Hiba a letöltés során:", error);
  }
};

// 2. Új feladat mentése (Create)
// 2. Új feladat mentése (Create)
const addTask = async () => {
  if (!newTask.value.title.trim()) return; // Ne küldjön üres adatot
  
  loading.value = true; // Ez tiltja le a gombot, hogy ne lehessen duplán kattintani
  
  try {
    const response = await axios.post(API_URL, {
      title: newTask.value.title,
      description: newTask.value.description,
      priority: newTask.value.priority
    });
    
    // Frissítjük a listát a backendtől kapott új elemmel
    tasks.value.unshift(response.data); 
    
    // Űrlap azonnali alaphelyzetbe állítása
    newTask.value = { title: '', description: '', priority: 'medium' };
  } catch (error) {
    console.error("Mentési hiba:", error.response?.data || error.message);
    alert("Nem sikerült elmenteni a feladatot. Ellenőrizd a konzolt!");
  } finally {
    loading.value = false; // Gomb újraaktiválása
  }
};


// 3. Státusz frissítése (Update)
const toggleTaskStatus = async (task) => {
  const updatedStatus = !task.is_completed;
  try {
    // API kérés küldése a módosított státusszal
    await axios.put(`${API_URL}/${task.id}`, { is_completed: updatedStatus });
    task.is_completed = updatedStatus; // Helyi állapot frissítése siker esetén
  } catch (error) {
    alert("Nem sikerült frissíteni a státuszt.");
  }
};

// 4. Feladat törlése (Delete)
const deleteTask = async (id) => {
  if (!confirm("Biztosan törölni szeretnéd ezt a feladatot?")) return;
  try {
    await axios.delete(`${API_URL}/${id}`);
    tasks.value = tasks.value.filter(t => t.id !== id); // Eltávolítás a helyi listából
  } catch (error) {
    alert("Nem sikerült a törlés.");
  }
};

// Kliensoldali szűrés (Computed property)
const filteredTasks = computed(() => {
  if (filterStatus.value === 'pending') return tasks.value.filter(t => !t.is_completed);
  if (filterStatus.value === 'completed') return tasks.value.filter(t => t.is_completed);
  return tasks.value;
});

onMounted(fetchTasks);
</script>

<style scoped>
.app-container { max-width: 600px; margin: 30px auto; font-family: 'Segoe UI', sans-serif; color: #333; padding: 0 15px; }
.card { background: #f9f9f9; padding: 20px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.05); margin-bottom: 20px; }
.task-form input, .task-form textarea, .task-form select { width: 100%; padding: 10px; margin-bottom: 10px; border: 1px solid #ddd; border-radius: 4px; box-sizing: border-box; }
.form-row { display: flex; align-items: center; gap: 10px; justify-content: flex-end; }
.form-row button { padding: 10px 20px; background: #28a745; color: white; border: none; border-radius: 4px; cursor: pointer; }
.filters { display: flex; gap: 10px; margin-bottom: 20px; }
.filters button { padding: 8px 16px; border: 1px solid #ddd; background: white; border-radius: 20px; cursor: pointer; }
.filters button.active { background: #007bff; color: white; border-color: #007bff; }
.task-item { display: flex; justify-content: space-between; align-items: center; padding: 15px; background: white; border: 1px solid #eee; border-radius: 6px; margin-bottom: 10px; box-shadow: 0 1px 3px rgba(0,0,0,0.02); }
.task-item.completed { background: #f4f4f4; opacity: 0.7; }
.task-item.completed h4 { text-decoration: line-through; color: #888; }
.task-content { display: flex; gap: 15px; align-items: flex-start; }
.task-content input[type="checkbox"] { margin-top: 5px; scale: 1.2; cursor: pointer; }
.task-content h4 { margin: 0 0 5px 0; font-size: 16px; }
.task-content p { margin: 0 0 8px 0; font-size: 14px; color: #666; }
.badge { display: inline-block; padding: 3px 8px; font-size: 11px; font-weight: bold; border-radius: 3px; color: white; }
.badge.low { background: #6c757d; }
.badge.medium { background: #ffc107; color: #333; }
.badge.high { background: #dc3545; }
.delete-btn { background: #dc3545; color: white; border: none; padding: 6px 12px; border-radius: 4px; cursor: pointer; }
.empty-list { text-align: center; color: #888; font-style: italic; margin-top: 20px; }
</style>
