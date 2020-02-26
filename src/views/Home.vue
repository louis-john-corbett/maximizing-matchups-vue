<template>
  <div class="home">
    <PlayerSearch v-bind:players="players" v-on:add-player="addPlayer" />
    <MatchupGrid v-bind:matchups="batterMatchups" title="Batter Matchups"/>
    <MatchupGrid v-bind:matchups="pitcherMatchups" title="Pitcher Matchups"/>
    <PlayerList v-bind:players="players" v-on:del-player="deletePlayer" />
    <MatchupGrid v-bind:matchups="matchups" title="Other Matchups"/>
  </div>
</template>

<script>
// @ is an alias to /src
import PlayerSearch from "@/components/PlayerSearch.vue";
import PlayerList from "@/components/PlayerList.vue";
import MatchupGrid from "@/components/MatchupGrid.vue";
import Axios from "axios";

const mlbApiBaseRoute = "https://statsapi.mlb.com";

export default {
  name: "home",
  components: {
    PlayerSearch,
    PlayerList,
    MatchupGrid
  },
  data() {
    return {
      games: [],
      players: [],
      matchups: [],
      batterMatchups: [],
      pitcherMatchups: []
    };
  },
  methods: {
    deletePlayer(player) {
      this.players = this.players.filter(p => p.player_id !== player.player_id);

      if (player.position == "P") {
        this.matchups = [...this.matchups, ...this.pitcherMatchups.filter(m => m.pitcher.id === player.player_id)];
        this.pitcherMatchups = this.pitcherMatchups.filter(m => m.pitcher.id !== player.player_id);
      } else {
        this.matchups = [...this.matchups, ...this.batterMatchups.filter(m => m.batter.id === player.player_id)];
        this.batterMatchups = this.batterMatchups.filter(m => m.batter.id !== player.player_id);
      }
    },
    addPlayer(player) {
      this.players = [...this.players, player];

      if (player.position == "P") {
        this.pitcherMatchups = [...this.pitcherMatchups, ...this.matchups.filter(m => m.pitcher.id === player.player_id)];
        this.matchups = this.matchups.filter(m => m.pitcher.id !== player.player_id);
      } else {
        this.batterMatchups = [...this.batterMatchups, ...this.matchups.filter(m => m.batter.id === player.player_id)];
        this.matchups = this.matchups.filter(m => m.batter.id !== player.player_id);
      }
    },
    addMatchup(batter, pitcher) {
      const params = {
        params: {
          stats: "vsTeam",
          group: "pitching",
          opposingPlayerId: batter.id
        }
      };
      const pitcherVsBatterUrl = `${mlbApiBaseRoute}${pitcher.link}/stats`;

      Axios.get(pitcherVsBatterUrl, params)
        .then(response => {
          if (response.data.stats.length > 0) {
            let matchup = {};
            matchup.batter = batter;
            matchup.pitcher = pitcher;
            matchup.matchup = response.data.stats.find(
              stat => stat.type.displayName === "vsTeamTotal"
            ).splits[0].stat;
            matchup.myBatter = false;
            matchup.myPitcher = false;
            this.matchups.push(matchup);
          }
        })
        .catch(error => {
          // eslint-disable-next-line no-console
          console.log(error);
        });
    }
  },
  created() {
    //let date = { params: { date: "09/29/2019" } };
    let date = { params: { date: "02/26/2020" } };

    // 1. Get all games for date and then matchups
    const gamesTodayUrl = `${mlbApiBaseRoute}/api/v1/schedule/games/?sportId=1`;
    Axios.get(gamesTodayUrl, date)
      .then(response => {
        let gamesToday = response.data.dates[0].games;

        // 2. Enrich game data for each game
        var i;
        for (i = 0; i < gamesToday.length; i++) {
          let enrichedGameInfoUrl = `${mlbApiBaseRoute}${gamesToday[i].link}`;
          Axios.get(enrichedGameInfoUrl)
            .then(response => {
              let curGame = {
                id: response.data.gamePk,
                link: response.data.link,
                teams: []
              };

              let homeAndAway = ["home", "away"];
              var k;
              for (k = 0; k < homeAndAway.length; k++) {
                let team = {
                  id: response.data.gameData.teams[homeAndAway[k]].id,
                  name: response.data.gameData.teams[homeAndAway[k]].name,
                  link: response.data.gameData.teams[homeAndAway[k]].link,
                  abbreviation:
                    response.data.gameData.teams[homeAndAway[k]].abbreviation,
                  homeAway: homeAndAway[k],
                  starter: {
                    id:
                      response.data.gameData.probablePitchers[homeAndAway[k]]
                        .id,
                    link:
                      response.data.gameData.probablePitchers[homeAndAway[k]]
                        .link,
                    name:
                      response.data.gameData.probablePitchers[homeAndAway[k]]
                        .fullName
                  },
                  batters: []
                };
                curGame.teams = [...curGame.teams, team];
              }

              this.games.push(curGame);
              return curGame;
            })
            .then(curGame => {
              var teamNum;
              for (teamNum = 0; teamNum < curGame.teams.length; teamNum++) {
                const rosterUrl = `${mlbApiBaseRoute}${curGame.teams[teamNum].link}/roster`;

                Axios.get(rosterUrl).then(response => {
                  const curRosterTeamId = response.data.teamId;
                  const batterTeamAbbreviation = curGame.teams.find(team => team.id === curRosterTeamId).abbreviation;
                  let starter = curGame.teams.find(team => team.id !== curRosterTeamId).starter;
                  starter.teamAbbreviation = curGame.teams.find(team => team.id !== curRosterTeamId).abbreviation;

                  response.data.roster.forEach(person => {
                    let batter = {
                      id: person.person.id,
                      name: person.person.fullName,
                      link: person.person.link,
                      position: person.position.abbreviation,
                      teamAbbreviation: batterTeamAbbreviation,
                      alternateTeamAbbreviation: batterTeamAbbreviation
                    };

                    this.addMatchup(batter, starter);
                  });
                });
              }
            })
            .catch(error => {
              // eslint-disable-next-line no-console
              console.log(error);
            });
        }
      })
      .catch(error => {
        // eslint-disable-next-line no-console
        console.log(error);
      });
  }
};
</script>

<style>
.home {
  width: 100%;
  max-width: 1000px;
  margin: 0 auto;
  text-align: center;
}
</style>