<template>
  <div class="home">
    <PlayerSearch v-bind:players="players" v-on:add-player="addPlayer" />
    <BatterMatchups v-bind:matchups="matchups" />
    <PitcherMatchups v-bind:matchups="matchups" />
    <PlayerList v-bind:players="players" v-on:del-player="deletePlayer" />
    <OtherMatchups v-bind:matchups="matchups" />
  </div>
</template>

<script>
// @ is an alias to /src
import PlayerSearch from "@/components/PlayerSearch.vue";
import PlayerList from "@/components/PlayerList.vue";
import BatterMatchups from "@/components/BatterMatchups.vue";
import PitcherMatchups from "@/components/PitcherMatchups.vue";
import OtherMatchups from "@/components/OtherMatchups.vue";
import Axios from "axios";

const mlbApiBaseRoute = "https://statsapi.mlb.com";

export default {
  name: "home",
  components: {
    PlayerSearch,
    PlayerList,
    BatterMatchups,
    PitcherMatchups,
    OtherMatchups
  },
  data() {
    return {
      games: [],
      players: [],
      matchups: []
    };
  },
  methods: {
    deletePlayer(player) {
      this.players = this.players.filter(p => p.player_id !== player.player_id);

      if (player.position == "P") {
        this.matchups = this.matchups.map(matchup => {
          if (matchup.pitcher.id === player.player_id) {
            return Object.assign({}, matchup, { myPitcher: false });
          }
          return matchup;
        });
      } else {
        this.matchups = this.matchups.map(matchup => {
          if (matchup.batter.id === player.player_id) {
            return Object.assign({}, matchup, { myBatter: false });
          }
          return matchup;
        });
      }
    },
    addPlayer(player) {
      this.players = [...this.players, player];

      if (player.position == "P") {
        this.matchups = this.matchups.map(matchup => {
          if (matchup.pitcher.id === player.player_id) {
            return Object.assign({}, matchup, { myPitcher: true });
          }
          return matchup;
        });
      } else {
        this.matchups = this.matchups.map(matchup => {
          if (matchup.batter.id === player.player_id) {
            return Object.assign({}, matchup, { myBatter: true });
          }
          return matchup;
        });
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
    let date = { params: { date: "09/29/2019" } };

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
                  const curTeam = this.games
                    .find(game =>
                      game.teams.some(team => team.id === curRosterTeamId)
                    )
                    .teams.find(team => team.id === curRosterTeamId);

                  response.data.roster.forEach(person => {
                    let batter = {
                      id: person.person.id,
                      name: person.person.fullName,
                      link: person.person.link,
                      position: person.position.abbreviation,
                      team: curTeam
                    };

                    // TODO: Delete this and just enrich in matchup
                    this.games
                      .find(game =>
                        game.teams.some(team => team.id === curRosterTeamId)
                      )
                      .teams.find(
                        team => team.id === curRosterTeamId
                      ).starter.team = curTeam;

                    this.games
                      .find(game =>
                        game.teams.some(team => team.id === curRosterTeamId)
                      )
                      .teams.find(
                        team => team.id === curRosterTeamId
                      ).batters = [
                      ...this.games
                        .find(game =>
                          game.teams.some(team => team.id === curRosterTeamId)
                        )
                        .teams.find(team => team.id === curRosterTeamId)
                        .batters,
                      batter
                    ];
                    this.addMatchup(
                      batter,
                      this.games
                        .find(game =>
                          game.teams.some(team => team.id === curRosterTeamId)
                        )
                        .teams.find(team => team.id !== curRosterTeamId)
                        .starter
                    );
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