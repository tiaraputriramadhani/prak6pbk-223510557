<template>
  <div class="container">
    <form @submit.prevent="save" class="form">
      <input type="text" v-model="form.title" placeholder="Title" class="input"/><br />
      <textarea v-model="form.content" placeholder="Content" class="textarea"></textarea><br />
      <button type="submit" class="button save-button">Save</button>
    </form>
    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <strong>{{ article.title }}</strong> <br />
        <p>{{ article.content }}</p>
        <button @click="edit(article)" class="button edit-button">Edit</button>
        <button @click="remove(article.id)" class="button delete-button">Delete</button>
      </li>
    </ul>
    <button @click="load" class="button load-button">Load</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: '',
    });

    const articles = ref([]);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
      } catch (error) {
        console.error('Error loading articles:', error);
      }
    }

    async function save() {
      try {
        let url, method;
        if (form.id === null) {
          const lastArticle = articles.value[articles.value.length - 1];
          const newId = lastArticle ? parseInt(lastArticle.id) + 1 : 1;
          form.id = newId.toString(); 
          url = 'http://localhost:3000/articles/';
          method = 'post';
        } else {
          url = 'http://localhost:3000/articles/' + form.id;
          method = 'put';
        }
        const response = await axios[method](url, form);
        articles.value = articles.value.map((article) =>
          article.id === response.data.id ? response.data : article
        );
        form.id = null;
        form.title = '';
        form.content = '';
      } catch (error) {
        console.log('Error saving article: ', error);
      }
    }

    async function remove(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter(article => article.id !== id);
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, edit, remove, load };
  },
};
</script>

<style scoped>
body {
  margin: 0;
  font-family: 'Helvetica Neue', Arial, sans-serif;
  background-color: #eef2f5;
}

.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  max-width: 800px;
  margin: 40px auto;
  padding: 20px;
  background-color: #f0f4f8;
  border-radius: 10px;
  box-shadow: 0 0 15px rgba(0, 0, 0, 0.1);
}

.form {
  margin-bottom: 30px;
  padding: 20px;
  background-color: #fff;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.form h2 {
  margin-bottom: 20px;
  font-size: 1.5em;
  color: #333;
}

.input,
.textarea {
  width: 100%;
  padding: 12px;
  margin-bottom: 15px;
  border: 1px solid #ddd;
  border-radius: 6px;
  font-size: 1em;
  transition: border-color 0.3s;
}

.input:focus,
.textarea:focus {
  border-color: #007BFF;
}

.button {
  padding: 10px 15px;
  margin: 5px 0;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 1em;
  transition: background-color 0.3s;
}

.save-button {
  background-color: #28a745;
  color: white;
}

.save-button:hover {
  background-color: #218838;
}

.edit-button {
  background-color: #007BFF;
  color: white;
}

.edit-button:hover {
  background-color: #0056b3;
}

.delete-button {
  background-color: #dc3545;
  color: white;
}

.delete-button:hover {
  background-color: #c82333;
}

.load-button {
  background-color: #17a2b8;
  color: white;
  margin-top: 20px;
}

.load-button:hover {
  background-color: #138496;
}

.article-list {
  list-style: none;
  padding: 0;
}

.article-item {
  padding: 15px;
  margin-bottom: 20px;
  background-color: #fff;
  border: 1px solid #ddd;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s, box-shadow 0.3s;
}

.article-item:hover {
  transform: scale(1.02);
  box-shadow: 0 0 15px rgba(0, 0, 0, 0.2);
}

.article-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.article-header strong {
  font-size: 1.2em;
  color: #333;
}

.article-actions .button {
  margin-left: 10px;
}

.article-item p {
  margin: 10px 0 0;
  color: #666;
}
</style>