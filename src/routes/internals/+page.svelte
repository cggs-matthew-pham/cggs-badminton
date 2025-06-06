<script>
    let allText = 
`Player 1
Player 2
Player 3
Player 4
Player 5
Player 6`;
  
    let homeText = 
`Player 1
Player 2
Player 3`;
  
    let awayText = 
`Player 4
Player 5
Player 6`;

  $: playerStats = allPlayers.map(player => {
  let played = 0;
  let wins = 0;
  let pointsFor = 0;
  let pointsAgainst = 0;

  for (const round of roundResults) {
    const result = round[player];
    if (result) {
      played += 1;

      const match = result.match(/^(\d+)\s*-\s*(\d+)\s*vs\s*(.+)$/);
      if (match) {
        const [_, forScoreStr, againstScoreStr] = match;
        const forScore = parseInt(forScoreStr);
        const againstScore = parseInt(againstScoreStr);
        pointsFor += forScore;
        pointsAgainst += againstScore;
        if (forScore > againstScore) {
          wins += 1;
        }
      }
    }
  }

  return { player, played, wins, pointsFor, pointsAgainst };
});

  
    $: allPlayers = allText.split('\n').map(name => name.trim()).filter(name => name !== '');
    $: homePlayers = homeText.split('\n').map(name => name.trim()).filter(name => name !== '');
    $: awayPlayers = awayText.split('\n').map(name => name.trim()).filter(name => name !== '');
    $: maxRows = Math.max(homePlayers.length, awayPlayers.length);
    $: scores = Array.from({ length: maxRows }, (_, i) => scores?.[i] ?? { home: 0, away: 0 });
  
    let selectedRound = 1;
    let roundResults = Array.from({ length: 5 }, () => ({})); // 5 rounds
  
    function saveCurrentRound() {
      const results = {};
      for (let i = 0; i < maxRows; i++) {
        const home = homePlayers[i];
        const away = awayPlayers[i];
        const homeScore = scores[i].home;
        const awayScore = scores[i].away;
  
        if (home) {
          results[home] = `${homeScore} - ${awayScore} vs ${away}`;
        }
        if (away) {
          results[away] = `${awayScore} - ${homeScore} vs ${home}`;
        }
      }
      roundResults[selectedRound - 1] = results;
    }

    function getSortedPlayerStats() {
  return [...playerStats].sort((a, b) => {
    if (b.wins !== a.wins) return b.wins - a.wins;
    if (b.pointsFor !== a.pointsFor) return b.pointsFor - a.pointsFor;
    return a.pointsAgainst - b.pointsAgainst;
  });



}

let sortByStats = true;

function toggleSort() {
  sortByStats = !sortByStats;
}

  </script>
  
  
  <style>
    .container {
      max-width: 1000px;
      margin: auto;
      padding: 30px;
      background: #fff;
      border-radius: 10px;
      font-family: sans-serif;
    }
  
    textarea {
      width: 100%;
      height: 120px;
      font-size: 16px;
      margin-bottom: 20px;
      padding: 10px;
      border-radius: 6px;
      border: 1px solid #ccc;
      background: white;
    }
  
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 20px;
      font-size: 18px;
    }
  
    th, td {
      border: 1px solid #ccc;
      padding: 8px;
      text-align: left;
    }
  
    th {
      background-color: #d1e2d1;
    }
  
    input[type="number"] {
      width: 80px;
      font-size: 16px;
      text-align: center;
    }
  
    .button-save {
      margin-top: 10px;
      padding: 10px 20px;
      font-size: 16px;
    }
  
    .textarea-columns {
      display: flex;
      gap: 20px;
      margin-bottom: 20px;
    }
  
    .textarea-columns > div {
      flex: 1;
    }
  
    h1 {
      font-size: 26px;
      margin-bottom: 20px;
    }
  
    .label {
      font-weight: bold;
      margin-bottom: 5px;
    }
  </style>
  
  <div class="container">
    <h1>🏸 CGGS Internal Badminton Match Tracker</h1>
  
    <div class="label">All Players</div>
    <textarea bind:value={allText}></textarea>
  
    <div class="textarea-columns">
      <div>
        <div class="label">Home Players</div>
        <textarea bind:value={homeText}></textarea>
      </div>
      <div>
        <div class="label">Away Players</div>
        <textarea bind:value={awayText}></textarea>
      </div>
    </div>
  
    <table>
      <thead>
        <tr>
          <th>Home Player</th>
          <th>Home Score</th>
          <th>Away Player</th>
          <th>Away Score</th>
        </tr>
      </thead>
      <tbody>
        {#each Array(maxRows) as _, i}
          <tr>
            <td>{homePlayers[i] || '-'}</td>
            <td><input type="number" bind:value={scores[i].home} min="0" /></td>
            <td>{awayPlayers[i] || '-'}</td>
            <td><input type="number" bind:value={scores[i].away} min="0" /></td>
          </tr>
        {/each}
      </tbody>
    </table>

    <div style="margin-top: 20px;">
        <label for="round-select"><strong>Select Round:</strong></label>
        <select id="round-select" bind:value={selectedRound}>
          {#each Array(5) as _, i}
            <option value={i + 1}>Round {i + 1}</option>
          {/each}
        </select>
      </div>
      
  
      <button class="button-save" on:click={saveCurrentRound}>
        Save Round {selectedRound} Results
      </button>

      <button class="button-save" on:click={toggleSort}>
        {sortByStats ? 'Show Original Order' : 'Sort by Stats'}
      </button>
      
      
  
      <h2>Results Table</h2>
<table>
  <thead>
    <tr>
      <th>Player</th>
      {#each Array(5) as _, i}
        <th>R{i + 1}</th>
      {/each}
      <th>Played</th>
      <th>Wins</th>
      <th>For</th>
      <th>Against</th>
    </tr>
  </thead>
  <tbody>
    {#each (sortByStats ? getSortedPlayerStats() : playerStats) as stat}
  <tr>
    <td>{stat.player}</td>
    {#each roundResults as result}
      <td>{result[stat.player] || ''}</td>
    {/each}
    <td>{stat.played}</td>
    <td>{stat.wins}</td>
    <td>{stat.pointsFor}</td>
    <td>{stat.pointsAgainst}</td>
  </tr>
{/each}

  </tbody>
</table>


      </div>
      
  