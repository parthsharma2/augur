<template>
  <d-container fluid class="main-content-container px-4">
    <!-- Page Header -->
    <div class="page-header row no-gutters py-4">
      <div class="col-12 col-sm-4 text-center text-sm-left mb-0">
        <h3 class="page-title" style="font-size: 1rem">Insights</h3>
      </div>
    </div>

    <!-- First Row of Posts -->

        <d-row>
          <div v-if="!loadedInsights" class="col-md-8 col-lg-9">
            <spinner style="top: 30%; padding: 1rem 0 1rem 0; position: relative; transform: translateY(-50%);"></spinner>
          </div>
          <d-col v-if="loadedInsights" v-for="(group, idx) in Object.keys(insights).slice(0,3)" :key="idx" lg="3" md="4" sm="8" class="mb-4">
            
            <d-card v-if="idx < 4" v-for="repo in Object.keys(insights[group]).slice(0,1)" class="card-small card-post card-post--1">
              <div class="card-post__image" v-for="metric in Object.keys(insights[group][repo]).slice(0,1)">
                <d-badge pill :class="['card-post__category', 'bg-' + themes[idx] ]">{{ group }}</d-badge>
                <insight-chart style="transform: translateX(-3.35rem)" :data="insights[group][repo][metric]" :url="repo" :color="colors[idx]"></insight-chart>

                <div class="card-post__author d-flex">
                  <a href="#" :style="colors[idx]" class="card-post__author-avatar card-post__author-avatar--small" style="text-indent: 0; text-align: center; font-size: 1rem">
                    <i class="material-icons" style="position: relative; top: 50%; transform: translateY(-60%)">{{ getDirection(insights[group][repo][metric]) }}</i>
                  </a>
                </div>
              </div>
              <d-card-body v-for="metric in Object.keys(insights[group][repo]).slice(0,1)">
                <h5 class="card-title">
                  <a href="#" class="text-fiord-blue">{{ repo.substr(19) }}</a>
                </h5>
                <p class="card-text d-inline-block mb-1" style="font-size: .75rem">This repository had a sharp {{ getPhrase(insights[group][repo][metric]) }} in Code Commits within the past month</p>
                <span class="text-muted" style="font-size: .75rem">1 month</span>
              </d-card-body>
            </d-card>
          </d-col>
        </d-row>

        <div style="transform: translateY(-0px)">
          <div class="page-header row no-gutters py-4" style="padding-top: 5 !important;">
            <div class="col-12 col-sm-4 text-center text-sm-left mb-0">
              <!-- <span class="text-uppercase page-subtitle">Components</span> -->
              <h3 class="page-title" style="font-size: 1rem">Most Frequent Repo Groups</h3>
            </div>
          </div>
          <!-- Second Row of Posts -->
          <d-row>
            <div style="padding-top: 3rem" v-if="apiGroups == {}" class="col-md-8 col-lg-9">
              <spinner></spinner>
            </div>

            <d-col v-else v-for="(group, idx) in Object.keys(insights).slice(0,6)" :key="idx" lg="4" sm="12" class="mb-4">
              <d-card class="card-small card">
                <div class="border-bottom card-header">
                  <h6 class="m-0">{{ group }}</h6>
                  <div class="block-handle"></div>
                </div>
                <div class="p-0 card-body">
                  <div class="list-group-small list-group list-group-flush">
                    <div v-for="(repo, i) in Object.keys(insights[group]).slice(0,5)" class="d-flex px-3 list-group-item" style="text-align: left">
                      <d-link :to="{name: 'repo_overview', params: {repo: repo}}" @click="setBaseRepo(repo)">
                        <span class="text-semibold text-fiord-blue" style="font-size: .65rem; padding: 0">{{ repo }}</span>
                      </d-link>
                      <div v-if="loadedInsights" v-for="metric in Object.keys(insights[group][repo]).slice(0,1)" style="margin: 0 0 0 auto; float:right">
                        <spark-chart :color="colors[idx]" :url="repo" :data="insights[group][repo][metric]" style="max-height: 50px; padding-bottom: 0px; "/>
                      </div>
                      
                    </div>
                  </div>
                </div>
              </d-card>
            </d-col>
          </d-row>
        </div>
  </d-container>
</template>

<script lang="ts">
import {mapActions, mapGetters, mapMutations} from "vuex";
import Component from 'vue-class-component';
import Vue from 'vue';
import SparkChart from '../components/charts/SparkChart.vue';
import InsightChart from '../components/charts/InsightChart.vue';
import Spinner from '../components/Spinner.vue';

interface FlexObject<TValue> {
  [id: string]: TValue;
}

@Component({
  methods: {
    ...mapActions('common',[
      'loadRepos',
      'loadRepoGroups',
      'createAPIObjects',
      'endpoint'
    ])
  },
  computed: {
    ...mapGetters('common', [
      'repoRelations',
      'repoGroups',
      'repos',
      'apiRepos',
      'apiGroups',
      'cache'
      // 'repoRelations'
    ]),
    // repoRelations() {
    //   return this.$store.getters['common/repoRelations']
    // },
    // repoGroups() {
    //   return this.$store.getters['common/repoGroups']
    // },
    // repos() {
    //   return this.$store.getters['common/repos']
    // },
    // apiRepos() {
    //   return this.$store.getters['common/apiRepos']
    // }
  },
  components: {
    SparkChart,
    InsightChart,
    Spinner
  }
})
export default class Dashboard extends Vue {
  
  // Data properties
  chart: any = null //"#343A40", 
  colors: string[] = ["#24a2b7", "#FF3647","#159dfb", "#FFC107","#4736FF","#3cb44b","#ffe119","#f58231","#911eb4","#42d4f4","#f032e6"];
  tempInsightEndpoints: string[] = ['issuesClosed', 'codeChangesLines', 'issueNew'];
  tempInsightRepos: any[] = [];
  tempInsightTimeframes: string[] = ['past 1 month', 'past 3 months', 'past 2 weeks'];
  themes: string[] = ['info', 'danger','royal-blue', 'warning', 'dark'];
  loadedRelations: boolean = false
  loadedInsights: boolean = false
  desiredReposPerGroup: number = 5
  insights: any = {}
  test: any[] = ['https://github.com/rails/ruby-coffee-script.git', 'https://github.com/Comcast/Hygieia.git','https://github.com/apache/jclouds-site.git',
    'https://github.com/apache/karaf-jclouds.git', 'https://github.com/openssl/openssl', 'https://github.com/rails/ruby-coffee-script.git']
  testGroups: any[] = ['Rails', 'Comcast', 'Risk Working Group', 'Apache']

  // Allow access to vuex getters
  repoRelations!: any;
  repos!: any;
  repoGroups!:any;
  apiRepos!:any;
  apiGroups!:any;
  cache!:any;

  // Allow access to vuex actions
  loadRepoGroups!:any;
  loadRepos!:any;
  createAPIObjects!:any;
  endpoint!:any;

  // 'created' lifecycle hook
  // Gets ran on component initialization, data collection should be handled here
  created () {

    // Load the data we need
    this.loadRepoGroups().then(() => {
      this.loadRepos().then(() => {
        // Creating AugurAPI objects for the entities we will query
        // for (let n = 0; n < 3; n++){
        //   let group = this.repoGroups[n]
        //   let relatedRepos:any[] = []
        //   for (let i = 0; i < this.desiredReposPerGroup && Object.keys(this.repoRelations[group.rg_name]).length > i; i++) {
        //     relatedRepos.push(this.repoRelations[group.rg_name][Object.keys(this.repoRelations[group.rg_name])[i]])
        //   }
        //   this.createAPIObjects({groups: [group], repos: relatedRepos})
        //   // Spark data
        //   let sparkRepos:any[] = []
        //   relatedRepos.forEach((repo:any) => {
        //     sparkRepos.push(this.apiRepos[repo.url])
        //   })
        // }
        this.endpoint({endpoints: ['topInsights']}).then((tuples:any) => {
          console.log(Object.keys(tuples))
          tuples.topInsights;
          // sleep(5000)
          // console.log(Object.keys(tuples))
          if ('topInsights' in tuples){
            tuples.topInsights.forEach((tuple:any) => {
              // tuple.value = +tuple.value
              if (this.insights[tuple.rg_name]){
                if (this.insights[tuple.rg_name][tuple.repo_git]) {
                  if (this.insights[tuple.rg_name][tuple.repo_git][tuple.ri_metric]) {
                    this.insights[tuple.rg_name][tuple.repo_git][tuple.ri_metric].push(tuple)
                  } else {
                    this.insights[tuple.rg_name][tuple.repo_git][tuple.ri_metric] = [tuple]
                  } 
                } else {
                  this.insights[tuple.rg_name][tuple.repo_git] = {}
                  this.insights[tuple.rg_name][tuple.repo_git][tuple.ri_metric] = [tuple]
                }
              } else {
                this.insights[tuple.rg_name] = {}
                this.insights[tuple.rg_name][tuple.repo_git] = {}
                this.insights[tuple.rg_name][tuple.repo_git][tuple.ri_metric] = [tuple]
              }
            })
            this.loadedInsights = true
          } else {
            console.log("top insights did not load correctly")
          }
        })
        this.loadedRelations = true
      })
    })
  }

  getOwner (url: string) {
    let first = url.indexOf(".")
    let last = url.lastIndexOf(".")
    let domain = null
    let owner = null
    let repo = null
    let extension = false
    if (url.includes("https://")){
      url = url.substr(8)
      console.log(url)
    }

    if (first == last) { //normal github
      domain = url.substring(0, first)
      owner = url.substring(url.indexOf('/') + 1, url.lastIndexOf('/'))
      repo = url.slice(url.lastIndexOf('/') + 1)
      console.log(owner+ "/" + repo)
      return owner
    } else if (url.slice(last) == '.git') { //github with extension
      domain = url.substring(0, first)
      extension = true
      owner = url.substring(url.indexOf('/') + 1, url.lastIndexOf('/'))
      repo = url.substring(url.lastIndexOf('/') + 1, url.length - 4)
      return owner
    } else { //gluster
      domain = url.substring(first + 1, last)
      owner = null //url.substring(url.indexOf('/') + 1, url.lastIndexOf('/'))
      repo = url.slice(url.lastIndexOf('/') + 1)
      return domain
    }
  }

  getRepo (url: string) {
    let first = url.indexOf(".")
    let last = url.lastIndexOf(".")
    let domain = null
    let owner = null
    let repo = null
    let extension = false

    if (first == last){ //normal github
      domain = url.substring(0, first)
      owner = url.substring(url.indexOf('/') + 1, url.lastIndexOf('/'))
      repo = url.slice(url.lastIndexOf('/') + 1)
      return repo
    } else if (url.slice(last) == '.git'){ //github with extension
      domain = url.substring(0, first)
      extension = true
      owner = url.substring(url.indexOf('/') + 1, url.lastIndexOf('/'))
      repo = url.substring(url.lastIndexOf('/') + 1, url.length - 4)
      return repo
    } else { //gluster
      domain = url.substring(first + 1, last)
      owner = null //url.substring(url.indexOf('/') + 1, url.lastIndexOf('/'))
      repo = url.slice(url.lastIndexOf('/') + 1)
      return repo
    }
  }

  getDirection (values: any[]) {
    let i = 0
    for (i = 0; i < values.length; i++){
      if (values[i].discovered){
        break
      }
    }
    if (values[i+1].value > values[i].value) 
      return 'arrow_upward'
    else
      return 'arrow_downward'
  }

  getPhrase (values: any[]) {
    let i = 0
    for (i = 0; i < values.length; i++){
      if (values[i].discovered){
        break
      }
    }
    if (values[i+1].value > values[i].value) 
      return 'increase'
    else
      return 'decrease'
  }

  setBaseRepo (e: any) {
    // this.$store.commit('setBaseRepo', store.AugurAPI.Repo({ gitURL: e.url}))
  }


}
</script>

