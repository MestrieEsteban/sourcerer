<script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@0.1.0"></script>
<template>
  <div>
    <b-container>
      <b-container>
        <b-row class="topBar">
          <b-col>
            <span class="username">
              {{ user.login }}
            </span>
          </b-col>
          <b-col>
            <span class="country">{{ user.location }}</span>
          </b-col>
        </b-row>
        <b-row style="margin-top: 40px;">
          <b-col>
            <div class="mb-2" style="margin-top: -105px;">
              <b-row align-v="center">
                <b-col style="">
                  <b-avatar :src="user.avatarUrl" size="7rem"></b-avatar>
                </b-col>
                <b-col>
                  <div class="buttd btn1">
                    Commits <b>{{ countLoading != true ? '...' : countCommit }}</b>
                  </div>
                </b-col>
                <b-col>
                  <div  class="buttd btn1">
                    Repos <b>{{ user.repositories.totalCount }}</b>
                  </div>
                </b-col>
                <b-col>
                  <div class="buttd btn1">
                    Lines of code <b>...</b>
                  </div>
                </b-col>
                <b-col>
                  <div class="buttd btn2">
                    Followers <b>{{ user.followers.totalCount }}</b>
                  </div>
                </b-col>
                <b-col>
                  <div class="buttd btn2">
                    Following <b>{{ user.following.totalCount }}</b>
                  </div>
                </b-col>
              </b-row>
            </div>
          </b-col>
        </b-row>

        <b-row style="padding-top:40px">
			<h1>Overview</h1>
		</b-row>
		<b-row>
		    <div v-if="chartLoad == true" class="center">
				<BarChart :width="1000" :data="Overview" :options="barChartOptionsOver" />
			</div>
		    <div v-else class="center">
				<pulse-loader color="#C0F1E2" size="40px"></pulse-loader>
			</div>
		</b-row>
		
        <b-row style="padding-top:40px">
			<h1>languages</h1>
		</b-row>
		<b-row>
			<div v-if="chartLoadD == true" class="center">
				<DoughnutChart :width="1000" :data="languages" :options="barChartOptions" />
			</div>
			<div v-else class="center">
				<pulse-loader color="#C0F1E2" size="40px"></pulse-loader>
			</div>
		</b-row>

        <b-row style="padding-top:40px">
			<h1>Repositories</h1>
		</b-row>
		<b-row>
			<div v-if="chartLoadR == true" class="center">
				<b-card :width="1000" no-body v-for="item in this.repos" :key="item.name">
							<div v-if="item.primaryLanguage">
					<b-card-header style="width:1000px;" header-tag="header" :class=item.name>
						<b-button block v-b-toggle="`${item.name}`" variant="info">
								<h4>
									{{item.name}}<br>
									<b-badge :style="`background-color:${item.primaryLanguage.color}; color:#0f0f0f`">{{item.primaryLanguage.name}}</b-badge>
								</h4>
						</b-button>
					</b-card-header>
							</div>
					<b-collapse :id=item.name visible accordion="my-accordion" role="tabpanel">
						<b-card-body>
							<div v-if="item.defaultBranchRef">
							<h4>
								Number of commit in this repositories : <b-badge>{{item.defaultBranchRef.target.history.totalCount}}</b-badge>
							</h4>
							</div>
						</b-card-body>
					</b-collapse>
				</b-card>
			</div>
			<div v-else class="center">
				<pulse-loader color="#C0F1E2" size="40px"></pulse-loader>
			</div>
		</b-row>
		    
		</b-row>
      </b-container>
    </b-container>
  </div>
</template>
<style>
.buttd {
  background-color: black;
  color: white;
  display: block;
  height: 60px;
  line-height: 60px;
  text-decoration: none;
  width: 130px;
  padding-left: 5px;
}
.btn1{
	background-color: #e3eae5 !important; 
	color: #555555;
}
.btn2{
	background-color: #c0f1e2 !important; 
	color: #555555;
}
.username {
  font-size: 32px;
  font-weight: bold;
  margin-top: -10px;
  color: #555555;
}
.country {
  font-size: 16px;
  font-weight: bold;
  margin-top: -10px;
  color: #555555;
}
.topBar {
  background-color: #f0f4f1;
  height: 200px;
  line-height: 100px;
}
.center{
	display: block; 
	margin-left: auto; 
	margin-right: auto;
}

</style>
<script>
import gql from 'graphql-tag'
import BarChart from "~/components/BarChart.vue";
import DoughnutChart from "~/components/DoughnutChart.vue";
import PulseLoader from 'vue-spinner/src/PulseLoader.vue'

const ALL_CHARACTERS_QUERY = gql`
  query ALL_CHARACTERS_QUERY {
    user(login: "mestrieesteban") {
      login
      name
      bio
      url
      followers {
        totalCount
      }
      following {
        totalCount
      }
      location
      avatarUrl
      repositories(ownerAffiliations: OWNER, isFork: false, first: 100) {
        totalCount
        nodes {
          name
          isPrivate
          description
          url
          createdAt
          updatedAt
          primaryLanguage {
            name
            color
          }
          languages(first: 100) {
			edges {
				size
				node {
					color
					name
				}
			}
          }
          collaborators {
            nodes {
              name
              login
              avatarUrl
              url
            }
          }
          defaultBranchRef {
            target {
              ... on Commit {
                history {
                  totalCount
                }
              }
            }
          }
        }
      }
    }
  }
`
export default {
	apollo: {
		user: {
		query: ALL_CHARACTERS_QUERY,
		prefetch: true,
		},
	},
	components: { BarChart, PulseLoader, DoughnutChart },
  	data: function () {
		return {
			user: '',
			countCommit: '...',
			name: '',
			language: [],
			countLoading: false,
			chartLoad:  false,
			chartLoadD: false,
			chartLoadR: true,
			repoNodes: [],
			repos:[],
			Overview: {
				labels: [],
				datasets: [{
					data: [],
					backgroundColor: [],
					label: 'Commit',
				},],
			},
			languages: {
				labels: [],
				datasets: [{
					data: [],
					backgroundColor: [],
					label: 'Percentage',
				},],
			},
			barChartOptionsOver: {
				responsive: true,
				legend: {display: false},
				tooltips: {backgroundColor: "#555555"},
				scales: {y: {beginAtZero: true}},
			},
			barChartOptions: {
				responsive: true,
				legend: {
					display: true,
					position: 'left',
				},
				tooltips: {backgroundColor: "#555555"},
				scales: {y: {beginAtZero: true}},
				plugins: {
					datalabels: {
						anchor: "end",
						align:  "end"
					},
				},
			},
		}
	},
	beforeMount: function () {
		this.getBrain()
	},
	mounted: function () {
		this.getOverview()
		this.getLanguages()
	},
	methods: {
		countCommits: function () {
		// this.countLoading = true
		// this.countCommit = 0
		// this.user.repositories.nodes.forEach((repos) => {
		// 	if (repos.defaultBranchRef)
		// 		this.countCommit += repos.defaultBranchRef.target.history.totalCount
		// })
		// this.countLoading = false
		},
		getBrain: function () {
			// this.user.repositories.nodes.forEach(repos => {
			// if(repos.primaryLanguage){
			// 	let name =  repos.primaryLanguage.name ? repos.primaryLanguage.name : 'No language'
			// 	if(!this.Overview.labels.some(e => e === name))
			// 	{
			// 		this.Overview.labels.push(name)
			// 		if(!this.Overview.labels[name])
			// 			this.Overview.labels[name] = 0
			// 		this.Overview.labels[name] += repos.defaultBranchRef.target.history.totalCount
			// 	}
			// }
			// user.repositories.nodes.forEach(repos => {
			// 	let name =  element.primaryLanguage.name
			// 	if(languages[name] === undefined){
			// 		languages[name] = repos.defaultBranchRef.target.history.totalCount
			// 	}
			// 	else{
			// 		languages[name] += repos.defaultBranchRef.target.history.totalCount
			// 	}
			// 	});	
			// });	
			// console.log(this.Overview.labels)

			//Refactor
			let repoNodes = this.user.repositories.nodes;
			this.repos = this.user.repositories.nodes.reverse();
			let currentNode = {};
			repoNodes = repoNodes
				.filter((node) => {
				currentNode = node;
				if(node.languages.edges){
					return node.languages.edges.length > 0;
				}
				})
				.reduce((acc, curr) => curr.languages.edges.concat(acc) ? curr.languages.edges.concat(acc) : "" , [])
				.reduce((acc, prev) => {
				let langSize = acc[prev.node.name] ? acc[prev.node.name].commit : 0;
				langSize = langSize + currentNode.defaultBranchRef.target.history.totalCount;
				return {
					...acc,
					[prev.node.name]: {
					name: prev.node.name,
					color: prev.node.color,
					commit: langSize,
					percent: 0
					},
				};
				}, {});	
			this.repoNodes = Object.entries(repoNodes).sort((a, b) => b[1].commit - a[1].commit);
			return;
		},
		getOverview: function (){
			let repoNodes = this.repoNodes;
			 
			let nameArray = [], valueArray = [], colorArray = [], countCommit = 0
			repoNodes.forEach(function(repo) {
				nameArray.push([repo[0]])
				valueArray.push(repo[1].commit)
				countCommit += repo[1].commit
				colorArray.push(repo[1].color)
			});
			this.countCommit = countCommit
			this.countLoading = true

			this.Overview.labels = nameArray
			this.Overview.datasets[0].data = valueArray
			this.Overview.datasets[0].backgroundColor = colorArray
			this.chartLoad = true
		},
		getLanguages: function() {
			let repoNodes = this.repoNodes;
			
			let nameArray = [], valueArray = [], colorArray = [], countCommit = 0, percentArray = []
			repoNodes.forEach(function(repo) {
				nameArray.push([repo[0]])
				countCommit += repo[1].commit
				colorArray.push(repo[1].color)
			});
			repoNodes.forEach(function(repo) {
				percentArray.push(((repo[1].commit / countCommit) * 100).toFixed(2))
			});
			this.languages.labels = nameArray
			this.languages.datasets[0].data = percentArray
			this.languages.datasets[0].backgroundColor = colorArray
			this.chartLoadD = true
		}
		
	},
}
</script>
