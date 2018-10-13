<template>
  <div class="container">
    <h1>Vue.js + Github</h1>
    <p class="lead">
      Página que lista issues de um repositório do Github, usando Vue.js.
    </p>
    <div v-if="response.status==='error'" class="alert alert-danger">
      {{ response.message }}
    </div>
    <div class="row">
      <div class="col">
        <div class="form-group">
          <input  v-model="username"
                  type="text" class="form-control" placeholder="github username">
        </div>
      </div>
      <div class="col">
        <div class="form-group">
          <input  v-model="repository"
                  type="text" class="form-control" placeholder="github repositório">
        </div>
      </div>
      <div class="col-3">
        <div class="form-group">
          <button @click.prevent.stop="getIssues()"
                  class="btn btn-success">
                  <span v-if="loader.getIssues">
                      <img src="/static/loading.svg" alt="">
                  </span>GO</button>
          <button @click.prevent.stop="reset()"
                  class="btn btn-danger">LIMPAR</button>
        </div>
      </div>
    </div>
    <br><hr><br>
    <table class="table table-sm table-bordered">
      <thead>
        <tr>
          <th width="100">Número</th>
          <th>Título</th>
        </tr>
      </thead>
      <tbody>
        <tr v-if="loader.getIssues">
          <td colspan="2" class="text-center"> <img src="/static/loading.svg" alt=""> </td>
        </tr>
        <tr v-if="showIssues" v-for="issue in issues" :key="issue.number">
          <td>
            <router-link  :to="{  name: 'GitHubIssue',
                                  params: {
                                    username: username,
                                    repository: repository,
                                    issue: issue.number
                                }}">
              {{ issue.number }}
            </router-link>
          </td>
          <td>{{ issue.title }}</td>
        </tr>
        <tr v-if="noIssues">
          <td colspan="2" class="text-center">Nenhuma issue encontrada!</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
/*  eslint linebreak-style: ["error", "windows"]  */
import axios from 'axios';

export default{
  name: 'GitHubIssues',
  created() {
    this.getLocalData();
  },
  data() {
    return {
      username: '',
      repository: '',
      response: {
        status: '',
        message: '',
      },
      issues: [],
      loader: {
        getIssues: false,
      },
    };
  },
  computed: {
    showIssues() {
      return !!this.issues.length && !this.loader.getIssues;
    },
    noIssues() {
      return !this.issues.length && !this.loader.getIssues;
    },
  },
  watch: {
    username(newValue, oldValue) {
      if (newValue !== oldValue) {
        this.resetResponse();
      }
    },
    repository(newValue, oldValue) {
      if (newValue !== oldValue) {
        this.resetResponse();
      }
    },
  },
  methods: {
    reset() {
      this.username = '';
      this.repository = '';
      this.issues = [];
      localStorage.clear();
      this.resetResponse();
    },
    resetResponse() {
      this.response.status = '';
      this.response.message = '';
    },
    getIssues() {
      this.resetResponse();
      this.issues = [];
      if (this.username && this.repository) {
        this.response.status = '';
        localStorage.setItem('gitHubIssues', JSON.stringify({ username: this.username, repository: this.repository }));
        this.loader.getIssues = true;
        const url = `https://api.github.com/repos/${this.username}/${this.repository}/issues`;
        axios.get(url).then((response) => {
          this.issues = response.data;
        }).catch(() => {
          this.response.status = 'error';
          this.response.message = 'Respositório não encontrado.';
        }).finally(() => {
          this.loader.getIssues = false;
        });
      }
    },
    getLocalData() {
      const localData = JSON.parse(localStorage.getItem('gitHubIssues'));
      if (localStorage.getItem('gitHubIssues')) {
        this.username = localData.username;
        this.repository = localData.repository;
        this.getIssues();
      }
    },
  },
};
</script>
