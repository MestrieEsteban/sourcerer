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
                    Commits <b>{{ countLoading == true ? '...' : countCommit }}</b>
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
		    <div v-if="chartLoag == true">
				<BarChart :data="Overview" :options="barChartOptions" />
			</div>
		    <div v-else>
				<pulse-loader color="#C0F1E2" size="40px"></pulse-loader>
			</div>

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

</style>
<script>
import gql from 'graphql-tag'
import BarChart from "~/components/BarChart.vue";
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
      repositories(
        first: 100
        orderBy: { field: CREATED_AT, direction: DESC }
      ) {
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
          languages(first: 20) {
            nodes {
              name
              color
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
	components: { BarChart, PulseLoader },
  	data: function () {
		return {
			user: '',
			countCommit: '...',
			countLoading: false,
			name: '',
			language: [],
			chartLoag: false,
			Overview: {
			labels: [],
			datasets: [
				{
					data: [3, 59],
				},
			],
		},
			barChartOptions: {
				responsive: true,
				legend: {
				display: true,
				},
				tooltips: {
				backgroundColor: "#17BF62",
				},
				scales: {
					y: {
						beginAtZero: true
					}
				},
			},
		}
	},
	mounted: function () {
		this.countCommits()
		this.getLanguages()

	},
	methods: {
		countCommits: function () {
		this.countLoading = true
		this.countCommit = 0
		this.user.repositories.nodes.forEach((repos) => {
			if (repos.defaultBranchRef)
			this.countCommit += repos.defaultBranchRef.target.history.totalCount
		})
		this.countLoading = false
		},
		getLanguages: function () {
			this.user.repositories.nodes.forEach(repos => {
			if(repos.primaryLanguage){

				let name =  repos.primaryLanguage.name ? repos.primaryLanguage.name : 'No language'
				if(!this.Overview.labels.some(e => e === name))
					this.Overview.labels.push(name)
				// console.log(name)
				// if(!this.labels.name){
				// 	// this.labels[name] = repos.defaultBranchRef.target.history.totalCount
				// }
				// else{
				// 	this.labels[name] += repos.defaultBranchRef.target.history.totalCount
				// }
			}
			});	
			this.chartLoag = true		
			
		},
	},
}
</script>
