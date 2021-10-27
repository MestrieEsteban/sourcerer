<template>
  <div>
    <b-container>
      <b-container>
      <b-row class="" style="height:200px; background-color:#F0F4F1; line-height: 100px;">
        <b-col style="color:#555555">
        <!-- USER NAME -->
          <span class="username" style="font-size:32px; font-weight:bold">{{user.login}}</span>
        </b-col>
        <b-col style="background-color:">
        <!-- PAYS, VILLE -->
          <span class="country">{{user.location}}</span>
        </b-col>
      </b-row>
      <b-row style="margin-top:40px; ;">
        <b-col>
        <div class="mb-2" style="margin-top:-105px;background-color:; ">
          <b-row align-v="center">
            <b-col style="">
              <!-- AVATAR -->
              <b-avatar :src="user.avatarUrl" size="7rem"></b-avatar>
            </b-col>
            <b-col>
              <!-- Commits -->
              <div class="buttd" style="background-color:#E3EAE5 !important; color:#555555">Commits <b>80</b></div>
            </b-col>
            <b-col>
              <div class="buttd" style="background-color:#E3EAE5 !important; color:#555555">Repos <b>{{user.repositories.totalCount}}</b></div>
            </b-col>
            <b-col>
              <div class="buttd" style="background-color:#E3EAE5 !important; color:#555555">Lines of code <b>80</b></div>
            </b-col>
            <b-col>
              <div class="buttd" style="background-color:#C0F1E2 !important; color:#555555">Followers <b>{{user.followers.totalCount}}</b></div>
            </b-col>
            <b-col>
              <div class="buttd" style="background-color:#C0F1E2 !important; color:#555555">Following <b>80</b></div>
            </b-col>
          </b-row>
        </div>
        </b-col>
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
    padding-left:5px;
}
</style>
<script>
import gql from 'graphql-tag'

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
    location
    avatarUrl
    repositories(first: 20, orderBy: {field: CREATED_AT, direction: DESC}) {
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
          	  nodes{
					name
					color
				} 
		  	}  
		  	collaborators{
				nodes{
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
`;

export default {
  apollo: {
    user: {
      query: ALL_CHARACTERS_QUERY,
      prefetch: true,
    },
  },
}
</script>
