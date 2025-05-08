<template>
  <div class="app-container">
    <h1 class="app-title">📚 Jadwal Kuliah</h1>
    
    <div class="controls">
      <div class="input-group">
        <input
          v-model="newActivity"
          @keyup.enter="addActivity"
          placeholder="Tambahkan kegiatan kuliah..."
          class="input-field"
          ref="activityInput"
        />
        <select v-model="newPriority" class="priority-select">
          <option value="low">Rendah</option>
          <option value="medium">Sedang</option>
          <option value="high">Tinggi</option>
        </select>
        <input
          type="date"
          v-model="newDeadline"
          class="deadline-input"
          :min="today"
        />
      </div>
      
      <button @click="addActivity" class="add-button">Tambah</button>
      
      <div class="filter-container">
        <label>
          <input
            type="checkbox"
            v-model="showOnlyUncompleted"
            class="filter-checkbox"
          />
          Tampilkan hanya yang belum selesai
        </label>
      </div>
    </div>
    
    <ul class="activity-list">
      <li
        v-for="(activity, index) in filteredActivities"
        :key="index"
        :class="[
          { completed: activity.completed },
          `priority-${activity.priority}`,
          { 'near-deadline': isNearDeadline(activity.deadline) && !activity.completed },
          { 'passed-deadline': isPassedDeadline(activity.deadline) && !activity.completed }
        ]"
        class="activity-item"
      >
        <input
          type="checkbox"
          v-model="activity.completed"
          class="activity-checkbox"
        />
        <div class="activity-content">
          <span class="activity-text">{{ activity.text }}</span>
          <div class="activity-meta">
            <span class="priority-badge">{{ getPriorityLabel(activity.priority) }}</span>
            <span v-if="activity.deadline" class="deadline-badge">
              ⏰ {{ formatDate(activity.deadline) }}
              <span v-if="isPassedDeadline(activity.deadline) && !activity.completed" class="deadline-warning">(TERLAMBAT!)</span>
              <span v-else-if="isNearDeadline(activity.deadline) && !activity.completed" class="deadline-warning">(SEGERA!)</span>
            </span>
          </div>
        </div>
        <button
          @click="removeActivity(index)"
          class="remove-button"
        >
          ❌
        </button>
      </li>
    </ul>
    
    <div v-if="activities.length === 0" class="empty-state">
      Tidak ada kegiatan kuliah. Tambahkan yang baru!
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      newActivity: '',
      newPriority: 'medium',
      newDeadline: '',
      activities: [],
      showOnlyUncompleted: false,
      today: new Date().toISOString().split('T')[0]
    }
  },
  computed: {
    filteredActivities() {
      if (this.showOnlyUncompleted) {
        return this.activities.filter(activity => !activity.completed)
      }
      return this.activities
    }
  },
  methods: {
    addActivity() {
      if (this.newActivity.trim() === '') return
      
      this.activities.push({
        text: this.newActivity,
        completed: false,
        priority: this.newPriority,
        deadline: this.newDeadline
      })
      this.newActivity = ''
      this.newPriority = 'medium'
      this.newDeadline = ''
      this.$refs.activityInput.focus()
    },
    removeActivity(index) {
      this.activities.splice(index, 1)
    },
    getPriorityLabel(priority) {
      const labels = {
        low: 'Rendah',
        medium: 'Sedang',
        high: 'Tinggi'
      }
      return labels[priority]
    },
    formatDate(dateString) {
      if (!dateString) return ''
      const options = { year: 'numeric', month: 'short', day: 'numeric' }
      return new Date(dateString).toLocaleDateString('id-ID', options)
    },
    isNearDeadline(dateString) {
      if (!dateString || this.isPassedDeadline(dateString)) return false
      const deadline = new Date(dateString)
      const today = new Date()
      const diffTime = deadline - today
      const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24))
      return diffDays <= 3
    },
    isPassedDeadline(dateString) {
      if (!dateString) return false
      const deadline = new Date(dateString)
      const today = new Date()
      today.setHours(0, 0, 0, 0)
      return deadline < today
    }
  },
  mounted() {
    this.$refs.activityInput.focus()
  }
}
</script>

<style>
:root {
  --primary-color: #4a6fa5;
  --secondary-color: #6b8cae;
  --accent-color: #ff7e5f;
  --background-color: #f5f7fa;
  --text-color: #333;
  --light-gray: #e0e0e0;
  --completed-color: #888;
  
  /* Priority colors */
  --priority-low: #6bbf70;
  --priority-medium: #f8c537;
  --priority-high: #e74c3c;
  
  /* Deadline colors */
  --near-deadline: #fff3cd;
  --passed-deadline: #f8d7da;
  --deadline-warning-color: #dc3545;
}

/* ... (previous style remain the same) ... */

.deadline-input {
  padding: 0.75rem;
  border: 2px solid var(--light-gray);
  border-radius: 5px;
  font-size: 1rem;
}

.activity-content {
  flex-grow: 1;
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.activity-meta {
  display: flex;
  gap: 0.5rem;
  align-items: center;
  flex-wrap: wrap;
}

.deadline-badge {
  font-size: 0.8rem;
  padding: 0.25rem 0.5rem;
  border-radius: 12px;
  background-color: #e9ecef;
  color: #495057;
  display: flex;
  align-items: center;
  gap: 0.25rem;
}

.deadline-warning {
  color: var(--deadline-warning-color);
  font-weight: bold;
}

.near-deadline {
  background-color: var(--near-deadline);
  animation: pulse 2s infinite;
}

.passed-deadline {
  background-color: var(--passed-deadline);
}

@keyframes pulse {
  0% {
    box-shadow: 0 0 0 0 rgba(255, 193, 7, 0.4);
  }
  70% {
    box-shadow: 0 0 0 10px rgba(255, 193, 7, 0);
  }
  100% {
    box-shadow: 0 0 0 0 rgba(255, 193, 7, 0);
  }
}
</style>