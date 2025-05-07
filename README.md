<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>5-a-Side Tournament Manager</title>
  <style>
    body { font-family: Arial; margin: 0; padding: 0; }
    .tabs { display: flex; background: #333; }
    .tabs button {
      flex: 1;
      padding: 15px;
      background: #444;
      color: white;
      border: none;
      cursor: pointer;
      font-size: 16px;
    }
    .tabs button.active { background: #007bff; }
    .tab-content {
      display: none;
      padding: 20px;
    }
    .tab-content.active { display: block; }
    input, select { margin: 5px 0; padding: 5px; }
    table, th, td {
      border: 1px solid black;
      border-collapse: collapse;
      padding: 5px;
    }
    table { margin-top: 10px; width: 100%; }
  </style>
</head>
<body>

<div class="tabs">
  <button class="tab-button active" onclick="openTab('teams')">Teams</button>
  <button class="tab-button" onclick="openTab('matches')">Matches</button>
  <button class="tab-button" onclick="openTab('playerStats')">Player Stats</button>
  <button class="tab-button" onclick="openTab('leaderboard')">Leaderboard</button>
</div>

<div id="teams" class="tab-content active">
  <h2>Enter Teams</h2>
  <div id="teamForms"></div>
  <button onclick="saveTeams()">Save Teams</button>
</div>

<div id="matches" class="tab-content">
  <h2>Enter Match Stats</h2>
  <form onsubmit="saveMatch(event)">
    <label>Team A:
      <select id="teamASelect"></select>
    </label>
    <label>Team B:
      <select id="teamBSelect"></select>
    </label>
    <div id="matchStats"></div>
    <button type="submit">Save Match</button>
  </form>
</div>

<div id="playerStats" class="tab-content">
  <h2>Player Stats</h2>
  <div id="playerStatsContainer"></div>
</div>

<div id="leaderboard" class="tab-content">
  <h2>Leaderboard</h2>
  <div id="leaderboardContainer"></div>
</div>

<script>
let teams = JSON.parse(localStorage.getItem('teams')) || {};
let matches = JSON.parse(localStorage.getItem('matches')) || [];

function openTab(tabName) {
  document.querySelectorAll('.tab-content').forEach(t => t.classList.remove('active'));
  document.querySelectorAll('.tab-button').forEach(b => b.classList.remove('active'));
  document.getElementById(tabName).classList.add('active');
  document.querySelector(`[onclick="openTab('${tabName}')"]`).classList.add('active');
  if (tabName === 'matches') loadMatchForm();
  if (tabName === 'playerStats') showPlayerStats();
  if (tabName === 'leaderboard') showLeaderboard();
}

// TEAMS
function renderTeamForms() {
  const container = document.getElementById('teamForms');
  container.innerHTML = '';
  for (let i = 1; i <= 5; i++) {
    container.innerHTML += `<div><h3>Team ${i}</h3>
      <input type="text" id="teamName${i}" placeholder="Team Name" required /><br>`;
    for (let j = 1; j <= 5; j++) {
      container.innerHTML += `<input type="text" id="team${i}Player${j}" placeholder="Player ${j}" required /><br>`;
    }
    container.innerHTML += '</div><hr>';
  }
}

function saveTeams() {
  teams = {};
  for (let i = 1; i <= 5; i++) {
    const name = document.getElementById(`teamName${i}`).value.trim();
    if (!name) continue;
    teams[name] = [];
    for (let j = 1; j <= 5; j++) {
      const player = document.getElementById(`team${i}Player${j}`).value.trim();
      if (player) teams[name].push(player);
    }
  }
  localStorage.setItem('teams', JSON.stringify(teams));
  alert('Teams saved!');
}
renderTeamForms();

// MATCHES
function loadMatchForm() {
  const teamASelect = document.getElementById('teamASelect');
  const teamBSelect = document.getElementById('teamBSelect');
  const matchStats = document.getElementById('matchStats');
  teamASelect.innerHTML = teamBSelect.innerHTML = '';
  Object.keys(teams).forEach(team => {
    teamASelect.innerHTML += `<option value="${team}">${team}</option>`;
    teamBSelect.innerHTML += `<option value="${team}">${team}</option>`;
  });
  teamBSelect.selectedIndex = 1;
  updateMatchStats();
}

document.getElementById('teamASelect').onchange = updateMatchStats;
document.getElementById('teamBSelect').onchange = updateMatchStats;

function updateMatchStats() {
  const teamA = document.getElementById('teamASelect').value;
  const teamB = document.getElementById('teamBSelect').value;
  const matchStats = document.getElementById('matchStats');
  if (teamA === teamB) {
    matchStats.innerHTML = '<p>Select two different teams.</p>';
    return;
  }

  const rows = (team, side) => teams[team].map(player => `
    <tr>
      <td>${player}</td>
      <td><input type="number" name="${side}_${player}_goals" value="0" /></td>
      <td><input type="number" name="${side}_${player}_assists" value="0" /></td>
      <td><input type="number" name="${side}_${player}_saves" value="0" /></td>
    </tr>`).join('');

  matchStats.innerHTML = `
    <h3>${teamA}</h3>
    <table><tr><th>Player</th><th>Goals</th><th>Assists</th><th>Saves</th></tr>
    ${rows(teamA, 'A')}</table>
    <h3>${teamB}</h3>
    <table><tr><th>Player</th><th>Goals</th><th>Assists</th><th>Saves</th></tr>
    ${rows(teamB, 'B')}</table>
  `;
}

function saveMatch(e) {
  e.preventDefault();
  const teamA = document.getElementById('teamASelect').value;
  const teamB = document.getElementById('teamBSelect').value;
  const inputs = document.querySelectorAll('#matchStats input');
  const stats = { teamA, teamB, players: {}, date: new Date().toISOString() };
  let teamAGoals = 0, teamBGoals = 0;

  inputs.forEach(input => {
    const [side, player, stat] = input.name.split('_');
    const key = `${side}_${player}`;
    stats.players[key] = stats.players[key] || { goals: 0, assists: 0, saves: 0 };
    stats.players[key][stat] = Number(input.value);
    if (stat === 'goals') {
      if (side === 'A') teamAGoals += Number(input.value);
      else teamBGoals += Number(input.value);
    }
  });

  stats.result = { [teamA]: teamAGoals, [teamB]: teamBGoals };
  matches.push(stats);
  localStorage.setItem('matches', JSON.stringify(matches));
  alert("Match saved!");
}

// PLAYER STATS
function showPlayerStats() {
  const container = document.getElementById('playerStatsContainer');
  container.innerHTML = '';
  const playerStats = {};

  matches.forEach(match => {
    for (const [key, value] of Object.entries(match.players)) {
      const player = key.split('_')[1];
      if (!playerStats[player]) playerStats[player] = { goals: 0, assists: 0, saves: 0 };
      playerStats[player].goals += value.goals;
      playerStats[player].assists += value.assists;
      playerStats[player].saves += value.saves;
    }
  });

  container.innerHTML = `<table><tr><th>Player</th><th>Goals</th><th>Assists</th><th>Saves</th></tr>
    ${Object.entries(playerStats).map(([player, stats]) =>
      `<tr><td>${player}</td><td>${stats.goals}</td><td>${stats.assists}</td><td>${stats.saves}</td></tr>`
    ).join('')}</table>`;
}

// LEADERBOARD
function showLeaderboard() {
  const container = document.getElementById('leaderboardContainer');
  const standings = {};

  Object.keys(teams).forEach(team => {
    standings[team] = { points: 0, played: 0, goals: 0, wins: 0, draws: 0, losses: 0 };
  });

  matches.forEach(match => {
    const [teamA, teamB] = [match.teamA, match.teamB];
    const [goalsA, goalsB] = [match.result[teamA], match.result[teamB]];
    standings[teamA].played++; standings[teamB].played++;
    standings[teamA].goals += goalsA;
    standings[teamB].goals += goalsB;

    if (goalsA > goalsB) {
      standings[teamA].points += 3;
      standings[teamA].wins++;
      standings[teamB].losses++;
      standings[teamB].points += 1;
    } else if (goalsA < goalsB) {
      standings[teamB].points += 3;
      standings[teamB].wins++;
      standings[teamA].losses++;
      standings[teamA].points += 1;
    } else {
      standings[teamA].points += 2;
      standings[teamB].points += 2;
      standings[teamA].draws++;
      standings[teamB].draws++;
    }
  });

  const sorted = Object.entries(standings).sort((a, b) => b[1].points - a[1].points);

  container.innerHTML = `<table>
    <tr><th>Team</th><th>Points</th><th>Played</th><th>Goals</th><th>Wins</th><th>Draws</th><th>Losses</th></tr>
    ${sorted.map(([team, s]) => 
      `<tr><td>${team}</td><td>${s.points}</td><td>${s.played}</td><td>${s.goals}</td><td>${s.wins}</td><td>${s.draws}</td><td>${s.losses}</td></tr>`
    ).join('')}</table>`;
}
</script>

</body>
</html>
