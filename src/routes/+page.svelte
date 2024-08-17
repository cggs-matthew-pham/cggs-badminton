<script>
    import confetti from 'canvas-confetti';
  
    let team1Details = `CGGS 3
  Coach 1
  1st Singles
  1st Doubles 1
  1st Doubles 2
  2nd Singles
  2nd Doubles 1
  2nd Doubles 2
  3rd Singles`;
  
    let team2Details = `CGS 3
  Coach 1
  1st Singles
  1st Doubles 1
  1st Doubles 2
  2nd Singles
  2nd Doubles 1
  2nd Doubles 2
  3rd Singles`;
  
    $: team1Array = team1Details.split('\n');
    $: team2Array = team2Details.split('\n');
  
    let scores = Array.from({ length: 6 }, () => ({ score1: 0, score2: 0 }));
  
    // State for tracking highlighted players
    let highlighted = Array.from({ length: 6 }, () => ({ player1: false, player2: false }));
  
    function toggleHighlight(index, player) {
      highlighted[index][player] = !highlighted[index][player];
      
      // Trigger confetti when highlighting a player
      if (highlighted[index][player]) {
        confetti({
          particleCount: 100,
          spread: 70,
          origin: { y: 0.6 }
        });
      }
    }

    function resetTeams() {
      team1Details = `CGGS`;
      team2Details = `CGS`;
    }
  
    function resetScores() {
        scores = Array.from({ length: 6 }, () => ({ score1: 0, score2: 0 }));
        highlighted = Array.from({ length: 6 }, () => ({ player1: false, player2: false }));
    }
  </script>
  
  <!-- Remaining code stays the same -->
  
  
  
  <style>

:global(body) {
    background-color: #f0f0ff; 
  }

    .textarea-container {
      display: flex;
      gap: 20px; /* Adds space between the text areas */
      margin-bottom: 10px;
    }

    button {
        font-size: 20px;
    }
  
    textarea {
      width: 100%;
      height: 150px;
      box-sizing: border-box;
      background-color: white;
    }
  
    table {
      width: 100%;
      border-collapse: collapse;
      font-size: 24px;
      background-color: #fffefe;
    }
  
    th, td {
      border: 1px solid black;
      padding: 8px;
      text-align: center;
    }

    th {
        background-color: #d1e2d1;
    }

    h1 {
        font-size: 26px;
    }
  
    .highlight {
      background-color: yellow;
    }
  
    .score-input {
      width: 80px;
      font-size: 30px;
      text-align: center;
      padding: 2px;
      background-color: white;
    }
  
    .score-separator {
      padding: 0 5px;
    }
  </style>
  
  <div class="textarea-container">
    <div>
      <h3>Team 1 Details:</h3>
      <textarea bind:value={team1Details}></textarea>
    </div>
  
    <div>
      <h3>Team 2 Details:</h3>
      <textarea bind:value={team2Details}></textarea>
    </div>

    <div>
        <h1>🏸ACT Girls Interschools Badminton Tournament 2024</h1>  
        <button on:click={resetScores}>Reset Scores</button> 
        
    </div>
    <div>
        <img src="badminton-logo.png" alt="ACT Interschools Logo" width="200" height="200"> 
    </div>
  </div>
  
  <!-- Table and other code remain the same -->
  
  <table>
    <thead>
      <tr>
        <th>Game</th>
        <th>{team1Array[0]}</th> <!-- Team 1 Name -->
        <th>Score</th>
        <th>{team2Array[0]}</th> <!-- Team 2 Name -->
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>Coach</td>
        <td>{team1Array[1]}</td> <!-- Coach 1 -->
        <td></td>
        <td>{team2Array[1]}</td> <!-- Coach 2 -->
      </tr>
      <tr>
        <td>1st Singles</td>
        <td class={highlighted[0].player1 ? 'highlight' : ''} on:click={() => toggleHighlight(0, 'player1')}>{team1Array[2]}</td> <!-- Team 1, 1st Singles -->
        <td>
          <input class="score-input" type="number" bind:value={scores[0].score1}>
          <span class="score-separator">:</span>
          <input class="score-input" type="number" bind:value={scores[0].score2}>
        </td>
        <td class={highlighted[0].player2 ? 'highlight' : ''} on:click={() => toggleHighlight(0, 'player2')}>{team2Array[2]}</td> <!-- Team 2, 1st Singles -->
      </tr>
      <tr>
        <td>1st Doubles</td>
        <td class={highlighted[1].player1 ? 'highlight' : ''} on:click={() => toggleHighlight(1, 'player1')}>{team1Array[3]}<br>{team1Array[4]}</td> <!-- Team 1, 1st Doubles -->
        <td>
          <input class="score-input" type="number" bind:value={scores[1].score1}>
          <span class="score-separator">:</span>
          <input class="score-input" type="number" bind:value={scores[1].score2}>
        </td>
        <td class={highlighted[1].player2 ? 'highlight' : ''} on:click={() => toggleHighlight(1, 'player2')}>{team2Array[3]}<br>{team2Array[4]}</td> <!-- Team 2, 1st Doubles -->
      </tr>
      <tr>
        <td>2nd Singles</td>
        <td class={highlighted[2].player1 ? 'highlight' : ''} on:click={() => toggleHighlight(2, 'player1')}>{team1Array[5]}</td> <!-- Team 1, 2nd Singles -->
        <td>
          <input class="score-input" type="number" bind:value={scores[2].score1}>
          <span class="score-separator">:</span>
          <input class="score-input" type="number" bind:value={scores[2].score2}>
        </td>
        <td class={highlighted[2].player2 ? 'highlight' : ''} on:click={() => toggleHighlight(2, 'player2')}>{team2Array[5]}</td> <!-- Team 2, 2nd Singles -->
      </tr>
      <tr>
        <td>2nd Doubles</td>
        <td class={highlighted[3].player1 ? 'highlight' : ''} on:click={() => toggleHighlight(3, 'player1')}>{team1Array[6]}<br>{team1Array[7]}</td> <!-- Team 1, 2nd Doubles -->
        <td>
          <input class="score-input" type="number" bind:value={scores[3].score1}>
          <span class="score-separator">:</span>
          <input class="score-input" type="number" bind:value={scores[3].score2}>
        </td>
        <td class={highlighted[3].player2 ? 'highlight' : ''} on:click={() => toggleHighlight(3, 'player2')}>{team2Array[6]}<br>{team2Array[7]}</td> <!-- Team 2, 2nd Doubles -->
      </tr>
      <tr>
        <td>3rd Singles</td>
        <td class={highlighted[4].player1 ? 'highlight' : ''} on:click={() => toggleHighlight(4, 'player1')}>{team1Array[8]}</td> <!-- Team 1, 3rd Singles -->
        <td>
          <input class="score-input" type="number" bind:value={scores[4].score1}>
          <span class="score-separator">:</span>
          <input class="score-input" type="number" bind:value={scores[4].score2}>
        </td>
        <td class={highlighted[4].player2 ? 'highlight' : ''} on:click={() => toggleHighlight(4, 'player2')}>{team2Array[8]}</td> <!-- Team 2, 3rd Singles -->
      </tr>
    </tbody>
  </table>
  