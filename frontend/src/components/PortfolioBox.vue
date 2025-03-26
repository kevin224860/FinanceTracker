<template>
  <div class="portfolio-box" v-if="portfolio.length > 0">
    <h2>Your Finance Entries</h2>

    <!-- Desktop View -->
    <table v-if="!isMobile" class="desktop-view">
      <thead>
        <tr>
          <th>Category</th>
          <th>Title</th>
          <th>Amount</th>
          <th>Date</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in portfolio" :key="item.entry_ID">
          <td>{{ item.category_name }}</td>
          <td>{{ item.title }}</td>
          <td>${{ item.amount.toFixed(2) }}</td>
          <td>{{ formatDate(item.date) }}</td>
          <td>
            <div class="action-buttons">
              <button class="edit-btn" @click="editFinanceEntry(item)">Edit <i class="fa-regular fa-pen-to-square"></i></button>
              <button class="delete-btn" @click="deleteFinanceEntry(item.entry_ID)">Delete <i class="fa-solid fa-trash"></i></button>
            </div>
          </td>
        </tr>
      </tbody>
    </table>

    <!-- Mobile View -->
    <div class="mobile-view" v-if="isMobile">
      <div v-for="(item, index) in portfolio" :key="item.entry_ID" class="finance-card">
        <div class="finance-header" @click="toggleExpand(index)">
          <h3>{{ item.title }}</h3>
          <i :class="expandedIndex === index ? 'fa-solid fa-angle-up' : 'fa-solid fa-angle-down'"></i>
        </div>
        <transition name="expand">
          <div v-if="expandedIndex === index" class="finance-details">
            <p><strong>Category:</strong> {{ item.category_name }}</p>
            <p><strong>Amount:</strong> ${{ item.amount.toFixed(2) }}</p>
            <p><strong>Date:</strong> {{ formatDate(item.date) }}</p>
            <div class="button-group">
              <button class="edit-btn" @click="editFinanceEntry(item)">Edit <i class="fa-regular fa-pen-to-square"></i></button>
              <button class="delete-btn" @click="deleteFinanceEntry(item.entry_ID)">Delete <i class="fa-solid fa-trash"></i></button>
            </div>
          </div>
        </transition>
      </div>
    </div>
  </div>

  <EditComponent 
    :show="isModalVisible" 
    :financeEntry="selectedEntry"
    @close="isModalVisible = false"
  />
</template>

<script>
import axios from "axios";
import EditComponent from './EditComponent.vue';

export default {
  components: { EditComponent },
  props: {
    portfolio: Array,
    isMobile: Boolean
  },
  data() {
    return {
      localPortfolio: [...this.portfolio],
      expandedIndex: null,
      isModalVisible: false,
      selectedEntry: null
    };
  },
  methods: {
    toggleExpand(index) {
      this.expandedIndex = this.expandedIndex === index ? null : index;
    },

    formatDate(dateString) {
      const date = new Date(dateString);
      return new Intl.DateTimeFormat("en-US").format(date);
    },

    editFinanceEntry(entry) {
      this.selectedEntry = { ...entry }; // Clone entry to prevent direct modification
      this.isModalVisible = true;
    },

    async deleteFinanceEntry(entryId) {
      const confirmDelete = window.confirm("Are you sure you want to delete this finance entry?");

      if (!confirmDelete) {
        return;
      }

      try {
        const token = localStorage.getItem("token");
        await axios.delete(`http://localhost:5000/api/addentry/${entryId}`, {
          headers: { Authorization: `Bearer ${token}` },
        });

        // Refresh or fetch the updated portfolio
        window.location.reload();
      } catch (error) {
        console.error("Error deleting finance entry:", error);
      }
    }
  }
};
</script>

<style scoped>
.portfolio-box {
  background: white;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  margin-top: 20px;
  width: 90%;
  max-width: 800px;
  overflow-x: auto; 
  text-align: center;
  border: 2px solid black;
}

.portfolio-box h2 {
  font-size: 1.5rem;
  color: #333;
  margin-bottom: 10px;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 10px;
  border: 2px solid black;
  border-radius: 10px;
}

th, td {
  padding: 10px;
  text-align: left;
  border-bottom: 1px solid #ddd;
}

th {
  background: #007bff;
  color: white;
  font-weight: bold;
}

tbody tr:nth-child(even) {
  background: #f2f2f2;
}

tbody tr:hover {
  background: #ddd;
}

button {
  padding: 8px 12px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.edit-btn {
  background: #28a745;
  color: white;
}

.edit-btn:hover {
  background: #1a6b2d;
}

.delete-btn {
  background: #dc3545;
  color: white;
}

.delete-btn:hover {
  background: #a32734;
}

.mobile-view {
  display: none;
}

.finance-card {
  background: #ffffff;
  margin: 10px 0;
  padding: 10px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.finance-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  cursor: pointer;
  padding: 10px;
  font-size: 1.2rem;
  font-weight: bold;
  background: #007bff;
  color: white;
  border-radius: 6px;
}

.finance-details {
  text-align: left;
  padding: 10px;
}

.button-group {
  display: flex;
  gap: 10px;
  margin-top: 10px;
}

@media (max-width: 790px) {
  .desktop-view {
    display: none;
  }

  .mobile-view {
    display: block;
  }
}
</style>
