<script setup>
import { ref, watch } from "vue"

const homeTeam = ref("")
const awayTeam = ref("")
const games = ref([])
const finishedGames = ref([])

const isDisabled = () =>
  !homeTeam.value ||
  !awayTeam.value ||
  homeTeam.value === awayTeam.value ||
  isOneTeamPlaying(homeTeam.value, awayTeam.value)

const isOneTeamPlaying = (team1, team2) => {
  return isPlaying(team1) || isPlaying(team2)
}

const isPlaying = (team) => {
  return games.value.filter(t => t.homeTeam === team || t.awayTeam === team).length > 0
}

const addGame = () => {
  games.value.push({
    id: Date.now(),
    homeTeam: homeTeam.value,
    homeScore: 0,
    awayTeam: awayTeam.value,
    awayScore: 0
  })
  homeTeam.value = ""
  awayTeam.value = ""
}

const finishGame = (game) => {
  const finishedGame = games.value.splice(game, 1)[0]
  finishedGame.finishedAt = getFormattedDate(new Date())
  finishedGames.value.push(finishedGame)
}

const getFormattedDate = (date) => {
  var day = addLeadingZeroes(date.getDay() + "")
  var month = addLeadingZeroes((date.getMonth() + 1) + "")
  var year = date.getFullYear()
  var hour = addLeadingZeroes(date.getHours() + "")
  var minutes = addLeadingZeroes(date.getMinutes() + "")
  var seconds = addLeadingZeroes(date.getSeconds() + "")
  return day + '/' + month + '/' + year + ' ' + hour + ':' + minutes + ':' + seconds
}

const addLeadingZeroes = (param) => {
  if (param.length == 1) {
    param = "0" + param;
  }
  return param;
}

watch(games.value, () => {
  games.value.sort((game1, game2) => {
    const totalScoreGame1 = getTotalScore(game1)
    const totalScoreGame2 = getTotalScore(game2)
    return totalScoreGame2 - totalScoreGame1
  })
})

const getTotalScore = (game) => game.homeScore + game.awayScore
</script>

<template>
  <div class="container">
    <h1>Welcome to the live score board !</h1>
    <div class="new-game">
      <fieldset>
        <legend>Please add here your new game</legend>
        <div>
          <label for="homeTeam">Team 1:</label>
          <input id="homeTeam" v-model.trim="homeTeam" placeholder="Add home team here">
        </div>
        <div>
          <label for="awayTeam">Team 2:</label>
          <input id="awayTeam" v-model.trim="awayTeam" placeholder="Add away team there">
        </div>
        <button id="newGameButton" :disabled=isDisabled() @click="addGame()">Add game</button>
        <div class="constraints">
          <ul>
            <li :class="{ error: !homeTeam }">Home team should not be null</li>
            <li :class="{ error: !awayTeam }">Away team should not be null</li>
            <li :class="{ error: homeTeam === awayTeam }">Home and away team should not be the same</li>
            <li :class="{ error: isOneTeamPlaying(homeTeam, awayTeam) }">Home and/or away team should not be already
              playing
            </li>
          </ul>
        </div>
      </fieldset>
    </div>
    <div class="games-container">
      <div class="games">
        <table>
          <caption>Ongoing games</caption>
          <tr v-for="(game, index) in games" :key="game.id">
            <td><button @click="game.homeScore++">Goal</button></td>
            <td>{{ game.homeTeam }}</td>
            <td>{{ game.homeScore }}</td>
            <td>-</td>
            <td>{{ game.awayScore }}</td>
            <td>{{ game.awayTeam }}</td>
            <td><button @click="game.awayScore++">Goal</button></td>
            <td><button @click="finishGame(index)">Finish this game</button></td>
          </tr>
        </table>
      </div>
      <div class="games">
        <table>
          <caption>Finished games</caption>
          <tbody>
            <tr v-for="game in finishedGames" :key="game.id">
              <td>{{ game.homeTeam }}</td>
              <td>{{ game.homeScore }}</td>
              <td>-</td>
              <td>{{ game.awayScore }}</td>
              <td>{{ game.awayTeam }}</td>
              <td>{{ game.finishedAt }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<style scoped>
.container {
  font-family: monospace;
  text-align: center;
}

.new-game {
  margin: auto;
  width: 40%;
}

input,
table {
  margin: 10px;
}

caption {
  border: 2px solid;
  font-size: 1.2rem;
  font-weight: 900;
}

button {
  font-size: 1.1rem;
  font-family: inherit;
}

li {
  color: forestgreen;
  text-align: left;
}

table {
  min-width: 500px;
}

.error {
  color: red;
}

.games-container {
  display: flex;
  justify-content: space-around;
  flex-wrap: wrap;
}
</style>
