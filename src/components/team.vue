<template>
  <div class="team">
    <div class="elementType">
      <div v-bind:key="player" v-for="player in orderedKeepers">
        {{ player.web_name }}
      </div>
    </div>
    <div class="elementType">
      <div v-bind:key="player" v-for="player in orderedDefenders">
        {{ player.web_name }}
      </div>
    </div>
    <div class="elementType">
      <div v-bind:key="player" v-for="player in orderedMidfielders">
        {{ player.web_name }}
      </div>
    </div>
    <div class="elementType">
      <div v-bind:key="player" v-for="player in orderedForwards">
        {{ player.web_name }}
      </div>
    </div>
    <!-- <div> {{ costOfTeam }} </div> -->
  </div>
</template>

<script>
import { getStatistics } from "../services/statistics.js";

export default {
  name: 'team',
  data() {
    return {
      stats: [],
      rankedPlayers: [],
      team: [],
      costOfTeam: 0,
      attributes: {
        "value_form": null,
        "value_season": null,
        "dreamteam_count": null,
        "form": null,
        "total_points": null,
        "points_per_game": null,
        "minutes": null,
        "bonus": null,
        "bps": null,
        "influence": null,
        "creativity": null,
        "threat": null,
        "ict_index": null,
        "yellow_cards": null,

        "goals_scored": null,
        "assists": null,

        "clean_sheets": null,

        "saves": null,
      }
    }
  },
  methods: {
    orderByRankScore: function(a,b) {
      if (a.rankScore > b.rankScore)
        return -1;
      if (a.rankScore < b.rankScore)
        return 1;
      return 0;
    },
    orderByPosition: function(a,b) {
      if (a.element_type < b.element_type)
        return -1;
      if (a.element_type > b.element_type)
        return 1;
      return 0;
    },
    // sets the current highest score that exists for each attribute
    getAttributeHighScores: function() {
      for (var attribute in this.attributes) {
        if (this.attributes.hasOwnProperty(attribute)) {
          this.stats = _.orderBy(this.stats, attribute, "desc");
          this.attributes[attribute] = this.stats[0][attribute];
        }
      }
    },
    // players' rank scores should be calculated based on position, upcoming schedule, injuries, and attributes above
    // not all attributes should weigh the same for every type of player
    rankPlayers: function() {
      for (var i = 0; i < this.stats.length; i++) {
        this.stats[i].rankScore = 0;
        for (var attribute in this.attributes) {
          if (this.attributes.hasOwnProperty(attribute)) {
            if (attribute == "yellow_cards") {
              this.stats[i].rankScore -= this.stats[i][attribute] / this.attributes[attribute];
            } else {
              this.stats[i].rankScore += this.stats[i][attribute] / this.attributes[attribute];
            }
          }
        }
        if (this.stats[i].web_name == "Arter") {
          console.log(this.stats[i].rankScore);
        }
      }
    },
    makeTeam: function() {
      var keeperCounter = 0;
      var defenderCounter = 0;
      var midfielderCounter = 0;
      var forwardCounter = 0;

      for (var i = 0; i < this.orderedStats.length; i++) {
        if (this.team.length == 15) {
          break;
        } else if (this.orderedStats[i].element_type == 1 && keeperCounter < 2) {
          this.team.push(this.orderedStats[i]);
          keeperCounter += 1;
        } else if (this.orderedStats[i].element_type == 2 && defenderCounter < 5) {
          this.team.push(this.orderedStats[i]);
          defenderCounter += 1;
        } else if (this.orderedStats[i].element_type == 3 && midfielderCounter < 5) {
          this.team.push(this.orderedStats[i]);
          midfielderCounter += 1;
        } else if (this.orderedStats[i].element_type == 4 && forwardCounter < 3) {
          this.team.push(this.orderedStats[i]);
          forwardCounter += 1;
        }
      }
    },
    calculateCostOfTeam: function() {
      for (var i = 0; i < this.team.length; i++) {
        this.costOfTeam += parseFloat(this.team[i].now_cost);
      }
      this.costOfTeam = this.costOfTeam / 10;
    }
  },
  created() {
    getStatistics()
      .then(response => {
        if (response.status == 200) {
          console.log(response.data);
          this.stats = response.data.elements;
          this.getAttributeHighScores();
          this.rankPlayers();
          this.makeTeam();
          this.calculateCostOfTeam();
        }
      });
  },
  computed: {
    orderedStats: function() {
      return this.stats.sort(this.orderByRankScore);
    },
    orderedKeepers: function() {
      return this.stats.sort(this.orderByRankScore);
    },
    orderedDefenders: function() {
      return this.stats.sort(this.orderByRankScore);
    },
    orderedMidfielders: function() {
      return this.stats.sort(this.orderByRankScore);
    },
    orderedForwards: function() {
      return this.stats.sort(this.orderByRankScore);
    },
    orderedTeam: function() {
      return this.team.sort(this.orderByPosition);
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .team {
    display: flex;
    justify-content: center;
  }
  .elementType {
    flex-grow: 1;
  }
</style>
