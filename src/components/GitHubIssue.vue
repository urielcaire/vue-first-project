<template>
  <div class="container">
    <div v-if="loader.getIssue" class="text-center">
      <img src="/static/loading.svg" alt="">
    </div>
    <div v-if="!loader.getIssue && issue.number" class="container">
      <h1>Issue #{{ issue.number }}</h1>
      <h2>{{ issue.title }}</h2>
      <p>{{ issue.body }}</p>
      <a href="javascript:history.go(-1)" class="btn btn-primary">Voltar</a>
    </div>
  </div>
</template>

<script>
/*  eslint linebreak-style: ["error", "windows"]  */
import axios from 'axios';

export default{
  name: 'GitHubIssue',
  created() {
    this.getIssue();
  },
  data() {
    return {
      issue: {},
      loader: {
        getIssue: false,
      },
    };
  },
  methods: {
    getIssue() {
      this.loader.getIssue = true;
      const url = `https://api.github.com/repos/${this.$route.params.username}/${this.$route.params.repository}/issues/${this.$route.params.issue}`;
      axios.get(url).then((response) => {
        this.issue = response.data;
      }).finally(() => {
        this.loader.getIssue = false;
      });
    },
  },
};
</script>
