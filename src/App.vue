<template>
  <div class="container">
    <form @submit.prevent="save" class="form">
      <input
        type="text"
        v-model="form.title"
        placeholder="Title"
        class="input"
      /><br />
      <textarea
        v-model="form.content"
        placeholder="Content"
        class="textarea"
      ></textarea
      ><br />
      <button type="submit" class="btn">Save</button>
    </form>
    <button @click="load" class="btn-load">Load Articles</button>
    <table class="article-table">
      <thead>
        <tr>
          <th>Title</th>
          <th>Content</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="article in articles" :key="article.id">
          <td>{{ article.title }}</td>
          <td>{{ article.content }}</td>
          <td>
            <button @click="edit(article)" class="btn-edit">Edit</button>
            <button @click="remove(article.id)" class="btn-delete">Delete</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import axios from "axios";
import { onMounted, reactive, ref } from "vue";

export default {
  setup() {
    const form = reactive({
      id: null,
      title: "",
      content: "",
    });

    const articles = ref([]);

    async function load() {
      try {
        const response = await axios.get("http://localhost:3000/articles");
        articles.value = response.data;
      } catch (error) {
        console.error("Error loading articles:", error);
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : "http://localhost:3000/articles";
        const method = form.id ? "put" : "post";
        const response = await axios[method](url, form);
        if (form.id) {
          const index = articles.value.findIndex(
            (article) => article.id === form.id
          );
          if (index !== -1) {
            articles.value[index] = response.data;
          }
        } else {
          articles.value.push(response.data);
        }
        // Reset form fields and ID after save
        form.id = null; // Reset ID
        form.title = "";
        form.content = "";
      } catch (error) {
        console.error("Error saving article:", error);
      }
    }

    async function remove(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        // Remove article from local list after successful deletion
        articles.value = articles.value.filter((article) => article.id !== id);
      } catch (error) {
        console.error("Error deleting article:", error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, edit, remove };
  },
};
</script>

<style scoped>
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.form {
  width: 100%;
  margin-bottom: 20px;
}

.input,
.textarea {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
}

.btn {
  padding: 10px 15px;
  margin-right: 10px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  background-color: #4caf50;
  color: #fff;
}

.btn:hover {
  background-color: #45a049;
}

.btn-edit {
  background-color: #2196f3;
}

.btn-delete {
  background-color: #f44336;
}

.article-table {
  width: 100%;
  border-collapse: collapse;
}

.article-table th,
.article-table td {
  padding: 10px;
  border-bottom: 1px solid #ddd;
}

.article-table th {
  background-color: #f2f2f2;
}

.article-table td {
  text-align: left;
}

.article-table td:last-child {
  text-align: center;
}
</style>
