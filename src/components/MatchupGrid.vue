<template>
    <div class="matchupGrid">
        <h1 id="matchup-header"> {{title}} </h1>
        <table style="width:100%">
          <thead>
            <tr>
              <th class="heading" @click="sort('batter.name')">Name</th>
              <th class="heading" @click="sort('pitcher.name')">Pitcher</th>
              <th class="heading" @click="sort('matchup.hits')">H/AB</th>
              <th class="heading" @click="sort('matchup.doubles')">2B</th>
              <th class="heading" @click="sort('matchup.triples')">3B</th>
              <th class="heading" @click="sort('matchup.homeRuns')">HR</th>
              <th class="heading" @click="sort('matchup.rbi')">RBI</th>
              <th class="heading" @click="sort('matchup.avg')">AVG</th>
              <th class="heading" @click="sort('matchup.ops')">OPS</th>
              <th class="heading" @click="sort('matchup.totalBases')">Bases</th>
              <th class="heading" @click="sort('matchup.strikeOuts')">Ks</th>
              <th class="heading" @click="sort('matchup.matchup.matchup.strikePercentage')">Strike:Ball</th>
            </tr>
          </thead>
          <tbody>
            <tr :key="`${matchup.batter.id}-${matchup.pitcher.id}`" v-for="matchup in sortedMatchups">
              <td>{{matchup.batter.name}} <span class="batterPosition">({{matchup.batter.position}})</span> <span class="batterTeam">{{matchup.batter.teamAbbreviation}}</span></td>
              <td>{{matchup.pitcher.name}} <span class="pitcherTeam">{{matchup.pitcher.teamAbbreviation}}</span></td>
              <td>{{matchup.matchup.hits}}/{{matchup.matchup.atBats}} </td>
              <td>{{matchup.matchup.doubles}}</td>
              <td>{{matchup.matchup.triples}}</td>
              <td>{{matchup.matchup.homeRuns}}</td>
              <td>{{matchup.matchup.rbi}}</td>
              <td>{{matchup.matchup.avg}}</td>
              <td>{{matchup.matchup.ops}}</td>
              <td>{{matchup.matchup.totalBases}}</td>
              <td>{{matchup.matchup.strikeOuts}}</td>
              <td>{{matchup.matchup.strikes}}:{{matchup.matchup.balls}} ({{matchup.matchup.strikePercentage}})</td>
            </tr>
          </tbody>
        </table>
    </div>
</template>

<script>

export default {
    name: "Matchups",
    props: ["matchups", "title"],
    data() {
      return {
        sortField: 'matchup.ops',
        sortDirection: 'desc'
      }
    },
    methods: {
      sort(s){
        if (s === this.sortField){
          this.sortDirection = this.sortDirection === 'asc' ? 'desc' : 'asc';
        } else{
          this.sortDirection = 'asc';
        }
        this.sortField = s;
      }
    },
    computed:{
      sortedMatchups: function(){
        return this.matchups.slice().sort((a,b) => {

          let prop = this.sortField.split('.');
          var len = prop.length;
          let modifier = 1;
          if(this.sortDirection === 'desc') modifier = -1;

          var i = 0;
          while( i < len ) { a = a[prop[i]]; b = b[prop[i]]; i++; }

          if(a < b) return -1 * modifier;
          if(a > b) return 1 * modifier;
          return 0;
        });
      }
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

tbody tr:nth-child(odd) {
  background: #eee;
}

th{
  font-size: 20px;
  cursor: pointer;
  text-align: left;
}
td{
  font-size: 15px;
  text-align: left;
}
span {
  display:inline;
}
.batterTeam{
  color:steelblue;
  font-size: 12px;
}
.batterPosition{
  font-size: 12px;
}
.pitcherTeam{
  color:steelblue;
  font-size: 12px;
}

</style>