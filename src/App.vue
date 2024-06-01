<template>
  <div class="container">
    <h2 class="header">✨ Your Awesome Notes ✨</h2>
    <form @submit.prevent="save" class="form">
      <input
        type="text"
        v-model="form.title"
        placeholder="What's the title?"
        class="input"
      />

      <textarea
        v-model="form.content"
        placeholder="Write down your thoughts..."
        class="textarea"
      ></textarea>
      <button type="submit" class="button green-button">Save Your Note</button>
    </form>

    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <h3>{{ article.title }}</h3>
        <p>{{ article.content }}</p>
        <div class="button-group">
          <button @click="edit(article)" class="button edit-button">Edit Note</button>
          <button @click="deleteArticle(article.id)" class="button delete-button">Delete Note</button>
        </div>
      </li>
    </ul>

    <button @click="load" class="button load-button">Load All Notes</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from "vue";
import axios from "axios";

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
        console.error("Oops! Failed to load notes:", error);
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : "http://localhost:3000/articles";
        const method = form.id ? "put" : "post";

        if (!form.id) {
          const newId = (articles.value.length + 1).toString();
          form.id = newId;
        }

        const response = await axios[method](url, form);

        if (method === "put") {
          articles.value = articles.value.map((article) =>
            article.id === response.data.id ? response.data : article
          );
        } else {
          articles.value.push(response.data);
        }

        form.id = null;
        form.title = "";
        form.content = "";
      } catch (error) {
        console.error("Oops! Failed to save note:", error);
      }
    }

    async function deleteArticle(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter((article) => article.id !== id);
      } catch (error) {
        console.error("Oops! Failed to delete note:", error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, edit, deleteArticle, load };
  },
};
</script>

<style>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  background: linear-gradient(145deg, #282c34, #3c3f41);
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.header {
  text-align: center;
  font-family: 'Comic Sans MS', cursive, sans-serif;
  color: #f4a261;
  margin-bottom: 20px;
}

.form {
  display: flex;
  flex-direction: column;
  margin-bottom: 20px;
}

.input,
.textarea {
  margin-bottom: 10px;
  padding: 10px;
  background: #444;
  border: 2px solid #555;
  border-radius: 4px;
  color: #f1faee;
  font-family: 'Arial', sans-serif;
}

.button {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s ease;
  color: #fff;
  font-family: 'Arial', sans-serif;
}

.green-button {
  background: #2a9d8f;
}

.green-button:hover {
  background: #21867a;
}

.delete-button {
  background: #e76f51;
}

.delete-button:hover {
  background: #cf593e;
}

.edit-button {
  background: #f4a261;
}

.edit-button:hover {
  background: #d78a50;
}

.load-button {
  background: #264653;
}

.load-button:hover {
  background: #1e3a47;
}

.article-list {
  list-style-type: none;
  padding: 0;
}

.article-item {
  background-color: #2a9d8f;
  padding: 20px;
  border-radius: 4px;
  margin-bottom: 10px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  color: #f1faee;
}

.button-group {
  display: flex;
  justify-content: flex-end;
  margin-top: 10px;
}

.button-group .button {
  margin-left: 10px;
}
</style>

