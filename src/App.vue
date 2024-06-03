<template>
  <div class="container">
    <h1>Manajemen Artikel</h1>
    <form @submit.prevent="save" class="article-form">
      <input type="text" v-model="form.title" placeholder="Judul" /><br />
      <textarea v-model="form.content" placeholder="Konten"></textarea><br />
      <button type="submit" class="btn save">Simpan</button>
    </form>
    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <strong>{{ article.title }}</strong
        ><br />
        <p>{{ article.content }}</p>
        <button v-if="article.title && article.content" @click="edit(article)" class="btn edit">
          Ubah
        </button>
        <button
          v-if="article.title && article.content"
          @click="deleteArticle(article.id)"
          class="btn delete"
        >
          Hapus
        </button>
      </li>
    </ul>
    <button @click="load" class="btn load">Muat Artikel</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue'
import axios from 'axios'

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: ''
    })

    const articles = ref([])

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles')
        articles.value = response.data
        console.log('Artikel dimuat:', response.data)
      } catch (error) {
        console.error('Kesalahan memuat artikel:', error)
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : 'http://localhost:3000/articles'
        const method = form.id ? 'put' : 'post'
        console.log(`Menyimpan artikel dengan metode: ${method}, url: ${url}, data:`, form)
        const response = await axios({
          method: method,
          url: url,
          data: { title: form.title, content: form.content }
        })
        console.log('Artikel disimpan:', response.data)

        if (form.id) {
          // Perbarui artikel dalam array articles dengan data baru dari respon server
          articles.value = articles.value.map((article) =>
            article.id === form.id ? response.data : article
          )
        } else {
          // Tambahkan artikel baru dengan ID dari respon server
          articles.value.push(response.data)
        }

        // Reset form
        form.id = null
        form.title = ''
        form.content = ''
      } catch (error) {
        console.error('Kesalahan menyimpan artikel:', error)
      }
    }

    async function deleteArticle(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`)
        articles.value = articles.value.filter((article) => article.id !== id)
        console.log(`Artikel dengan id ${id} dihapus`)
      } catch (error) {
        console.error('Kesalahan menghapus artikel:', error)
      }
    }

    function edit(article) {
      form.id = article.id
      form.title = article.title
      form.content = article.content
    }

    onMounted(load)

    return { form, articles, save, edit, load, deleteArticle }
  }
}
</script>
<style>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

h1 {
  text-align: center;
  margin-bottom: 20px;
}

.article-form input[type='text'],
.article-form textarea {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.article-form textarea {
  height: 150px;
}

.btn {
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.btn.save {
  background-color: #4caf50;
  color: white;
}

.btn.edit,
.btn.delete {
  background-color: #f44336;
  color: white;
  margin-left: 10px;
}

.article-list {
  list-style-type: none;
  padding: 0;
}

.article-item {
  margin-bottom: 20px;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.article-item strong {
  font-size: 1.2em;
}

.article-item p {
  margin-top: 10px;
}

.btn.load {
  display: block;
  margin: 20px auto;
}
</style>
