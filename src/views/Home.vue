<template>
  <div class="home">
    <PlayerSearch v-bind:players="players" v-on:add-player="addPlayer" />
    <BatterMatchups v-bind:batterMatchups="batterMatchups" />
    <PitcherMatchups v-bind:pitcherMatchups="pitcherMatchups" />
    <PlayerList v-bind:players="players" v-on:del-player="deletePlayer" />
  </div>
</template>

<script>
// @ is an alias to /src
import PlayerSearch from '@/components/PlayerSearch.vue'
import PlayerList from '@/components/PlayerList.vue'
import BatterMatchups from '@/components/BatterMatchups.vue'
import PitcherMatchups from '@/components/PitcherMatchups.vue'
import Axios from 'axios'

export default {
  name: 'home',
  components: {
    PlayerSearch,
    PlayerList,
    BatterMatchups,
    PitcherMatchups
  },
  data() {
    return {
      teams: [],
      games: [],
      probableStarters: [],
      players: [],
      batters: [],
      pitchers: [],
      batterMatchups: [],
      pitcherMatchups: []
    }
  },
  methods: {
    deletePlayer(id){
      this.players = this.players.filter(player => player.player_id !== id);
      this.batters = this.batters.filter(player => player.player_id !== id);
      this.pitchers = this.pitchers.filter(player => player.player_id !== id);
      this.batterMatchups = this.batterMatchups.filter(matchup => matchup.batter.player_id !== id);
      this.pitcherMatchups = this.pitcherMatchups.filter(matchup => matchup.pitcher.id !== id);
    },
    addPlayer(newPlayer){
      this.players = [...this.players, newPlayer];
      newPlayer.position === 'P' ? this.addPitchingMatchups(newPlayer) : this.addBattingMatchup(newPlayer);
    },
    addPitchingMatchups(pitcher){
      this.pitchers = [...this.pitchers, pitcher];
    },
    addBattingMatchup(batter){
      this.batters = [...this.batters, batter];

      var batterGame = this.games.find(function(game) { 
        return game.teams.away.team.id === batter.team_id || game.teams.home.team.id === batter.team_id; 
      });

      var opposingPitcher = this.probableStarters.find(function(starter) {
        return starter.currentTeam.id !== batter.team_id 
        && (starter.currentTeam.id === batterGame.teams.home.team.id || starter.currentTeam.id === batterGame.teams.away.team.id);
      });

      if (batterGame !== undefined){
        if (opposingPitcher !== undefined){

          const pitcherVsBatterUrl = `https://statsapi.mlb.com/api/v1/people/${opposingPitcher.id}/stats`;
          const params = { params: { stats: 'vsTeam', group: 'pitching', opposingPlayerId: batter.player_id } };
        
          Axios.get(pitcherVsBatterUrl, params)
          .then((response) => {
            if (response.data.stats.length > 0){
              // eslint-disable-next-line no-console
              let matchup = {}
              matchup.batter = batter;
              matchup.pitcher = opposingPitcher;
              matchup.matchup = response.data.stats.find(stat => stat.type.displayName === 'vsTeamTotal').splits[0].stat;
              matchup.pitcher.currentTeam.team_abbrev = this.teams.find(team => team.id === opposingPitcher.currentTeam.id).abbreviation;
              this.batterMatchups.push(matchup);
            }
            else {
              //should push something here in a bit
              // eslint-disable-next-line no-console
              console.log(`${batter.name_display_first_last} doesn't have any batting history against ${opposingPitcher.firstLastName}`);
            }
          })
          .catch((error) => {
            // eslint-disable-next-line no-console
            console.log(error);
          });
        }
        else{
          // eslint-disable-next-line no-console
          console.log(`${batter.name_display_first_last}'s game is found at https://statsapi.mlb.com${batterGame.link} but probable pitching is unknown or I have a bug.`);
        }
      }
      else{
        // eslint-disable-next-line no-console
        console.log(`${batter.name_display_first_last} doesn't seem to be playing today, or I have a bug.`);
      }
    },
    addProbablePitcher(player){
      
      // Player by ID - JSON
      const playerByIdUrl = `https://statsapi.mlb.com${player.link}`;
      Axios.get(playerByIdUrl, { params: { hydrate: 'currentTeam' } })
      .then((response) => {
        this.probableStarters.push(response.data.people[0]);
      })
      .catch((error) => {
        // eslint-disable-next-line no-console
        console.log(error);
      });
    }
  },
  created(){

    let date = { params: { date: '09/29/2019' } };

    // 1. Get all teams in the league
    const mlbTeamsUrl = 'https://statsapi.mlb.com/api/v1/teams/?sportId=1';
    Axios.get(mlbTeamsUrl)
    .then((response) => {
      this.teams = response.data.teams;
    })
    .catch((error) => {
      // eslint-disable-next-line no-console
      console.log(error);
    });

    // 2. Get all games and then matchups
    const gamesTodayUrl = 'http://statsapi.mlb.com/api/v1/schedule/games/?sportId=1';
    Axios.get(gamesTodayUrl, date)
    .then((response) => {
      this.games = response.data.dates[0].games;

      // 3. Get probable pitchers for each game
      var i;
      for (i = 0; i < this.games.length; i++){
        let gameInfoWithProbablePitchersUrl = `https://statsapi.mlb.com/api/v1.1/game/${this.games[i].gamePk}/feed/live`;
        Axios.get(gameInfoWithProbablePitchersUrl)
        .then((response) => {
          this.addProbablePitcher(response.data.gameData.probablePitchers.home);
          this.addProbablePitcher(response.data.gameData.probablePitchers.away);
        })
        .catch((error) => {
          // eslint-disable-next-line no-console
          console.log(error);
        });
      }
    })
    .catch((error) => {
      // eslint-disable-next-line no-console
      console.log(error);
    });
  }
}
</script>

<style>
.home {
  width: 100%;
  max-width: 1000px;
  margin: 0 auto;
  text-align: center;
}
</style>