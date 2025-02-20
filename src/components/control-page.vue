<template>
  <div class="item-list">
    <h1>Joints</h1>
    <table>
      <thead>
      <tr>
        <th>Name</th>
        <th>Degree</th>
        <th>Action</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="item in items" :key="item.key">
        <td>{{ item.key }}</td>
        <td>
          <input
              type="number"
              v-model.number="item.value"
              min="45"
              max="135"
          >
        </td>
        <td>
          <button @click="updateItem(item)">Update</button>
        </td>
      </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'ItemList',
  data() {
    return {
      items: []
    }
  },
  mounted() {
    this.fetchItems()
  },
  methods: {
    async fetchItems() {
      try {
        const response = await axios.get('http://localhost:8000/items')
        this.items = response.data
      } catch (error) {
        console.error('Error fetching items:', error)
        this.handleError(error)
      }
    },
    async updateItem(item) {
      try {
        if (item.value < 45 || item.value > 135) {
          alert('Value must be between 45 and 135')
          return
        }

        await axios.put('http://localhost:8000/items', {
          key: item.key,
          value: item.value
        })

        alert('Item updated successfully')
      } catch (error) {
        console.error('Error updating item:', error)
        this.handleError(error)
      }
    },
    handleError(error) {
      const errorMessage = error.response?.data?.detail || 'An unexpected error occurred'
      alert(errorMessage)
    }
  }
}
</script>

<style scoped>
.item-list {
  max-width: 600px;
  margin: 0 auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}

th {
  background-color: #f2f2f2;
}

input {
  width: 100%;
  padding: 5px;
  box-sizing: border-box;
}

button {
  width: 100%;
  padding: 5px;
  background-color: #4CAF50;
  color: white;
  border: none;
  cursor: pointer;
}

button:hover {
  background-color: #45a049;
}
</style>