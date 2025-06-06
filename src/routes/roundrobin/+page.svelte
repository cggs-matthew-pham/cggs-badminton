<script>
    // Initial list of participants, similar to team1Details/team2Details
    let participantDetails = `Arya / Emily
Henrietta / Kilana
Liz / Louise
Shen / Sydney
Sonia / Sophia
Suha / Maria`;

    // Reactive array of participant names
    $: participants = participantDetails.split('\n').filter(p => p.trim() !== '');

    // Matrix to store match results.
    // results[i][j] stores the result of participants[i] vs participants[j]
    // 'W' for win, 'L' for loss, null for not played/not set.
    // We'll initialize it based on the number of participants.
    let matchResults = [];

    // Function to initialize/reset matchResults when participants change
    $: {
        const numParticipants = participants.length;
        const newMatchResults = Array(numParticipants).fill(null).map(() => Array(numParticipants).fill(null));

        // Preserve existing results if participants array size is the same
        if (matchResults.length === numParticipants) {
            for (let i = 0; i < numParticipants; i++) {
                for (let j = 0; j < numParticipants; j++) {
                    if (i !== j) { // Don't allow self-matches
                        newMatchResults[i][j] = matchResults[i][j];
                    }
                }
            }
        }
        matchResults = newMatchResults;
    }

    // Function to toggle match result
    function toggleMatchResult(player1Index, player2Index) {
        if (player1Index === player2Index) return; // Cannot play against self

        const currentResult = matchResults[player1Index][player2Index];

        let newResult1 = null; // Result for player1 vs player2
        let newResult2 = null; // Result for player2 vs player1

        if (currentResult === 'W') {
            newResult1 = 'L'; // Change W to L
            newResult2 = 'W'; // Correspondingly, L becomes W
        } else if (currentResult === 'L') {
            newResult1 = null; // Change L to null (reset)
            newResult2 = null; // Correspondingly, W becomes null
        } else { // currentResult is null
            newResult1 = 'W'; // Set to W
            newResult2 = 'L'; // Correspondingly, set to L
        }

        // Svelte reactivity requires updating the array directly or reassigning
        matchResults[player1Index][player2Index] = newResult1;
        matchResults[player2Index][player1Index] = newResult2;

        // Trigger reactivity for the entire array
        matchResults = [...matchResults];
    }

    // Computed properties for player stats
    $: playerStats = participants.map((name, index) => {
        let played = 0;
        let wins = 0;
        let losses = 0;
        let points = 0; // 1 point per win, 0 per loss

        for (let j = 0; j < participants.length; j++) {
            if (index === j) continue; // Skip self-match

            const result = matchResults[index][j];
            if (result === 'W') {
                played++;
                wins++;
                points += 1; // 1 point for a win
            } else if (result === 'L') {
                played++;
                losses++;
            }
        }
        return { name, played, wins, losses, points, rank: 0 }; // Rank will be calculated later
    });

    // Calculate ranks after all stats are computed
    $: rankedPlayers = (() => {
        const sorted = [...playerStats].sort((a, b) => {
            // Primary sort: Points (descending)
            if (b.points !== a.points) {
                return b.points - a.points;
            }
            // Secondary sort: Wins (descending)
            if (b.wins !== a.wins) {
                return b.wins - a.wins;
            }
            // Tertiary sort: Losses (ascending)
            if (a.losses !== b.losses) {
                return a.losses - b.losses;
            }
            // Fallback: Alphabetical by name (ascending)
            return a.name.localeCompare(b.name);
        });

        // Assign ranks
        let currentRank = 1;
        for (let i = 0; i < sorted.length; i++) {
            if (i > 0 && (sorted[i].points !== sorted[i-1].points || sorted[i].wins !== sorted[i-1].wins || sorted[i].losses !== sorted[i-1].losses)) {
                currentRank = i + 1;
            }
            sorted[i].rank = currentRank;
        }
        return sorted;
    })();

    // Sorting state for the table
    let sortColumn = 'rank'; // Default sort by rank
    let sortDirection = 'asc'; // Default sort direction for rank

    function handleSort(column) {
        if (sortColumn === column) {
            sortDirection = sortDirection === 'asc' ? 'desc' : 'asc';
        } else {
            sortColumn = column;
            sortDirection = 'asc'; // Default to ascending for new column, except rank which is asc
            if (column === 'points' || column === 'wins') {
                sortDirection = 'desc'; // Points and Wins usually sort descending
            }
        }
    }

    // Final sorted list for rendering the summary table
    $: finalSortedPlayers = (() => {
        const sorted = [...rankedPlayers]; // Start with the pre-ranked list

        sorted.sort((a, b) => {
            let comparison = 0;
            if (sortColumn === 'name') {
                comparison = a.name.localeCompare(b.name);
            } else if (sortColumn === 'played') {
                comparison = a.played - b.played;
            } else if (sortColumn === 'wins') {
                comparison = a.wins - b.wins;
            } else if (sortColumn === 'losses') {
                comparison = a.losses - b.losses;
            } else if (sortColumn === 'points') {
                comparison = a.points - b.points;
            } else if (sortColumn === 'rank') {
                comparison = a.rank - b.rank;
            }

            return sortDirection === 'asc' ? comparison : -comparison;
        });
        return sorted;
    })();

    function resetTournament() {
        participantDetails = `Arya / Emily
Henrietta / Kilana
Liz / Louise
Shen / Sydney
Sonia / Sophia
Suha / Maria`; // Reset to default participants
        // The reactive block for matchResults will re-initialize it
    }

</script>

<style>
    :global(body) {
        font-family: sans-serif;
        background-color: #f0f0ff;
        margin: 20px;
    }

    .container {
        display: flex;
        gap: 20px;
        margin-bottom: 20px;
    }

    .input-section {
        flex: 1;
    }

    .table-section {
        flex: 2;
    }

    textarea {
        width: 100%;
        height: 200px;
        padding: 10px;
        border: 1px solid #ccc;
        border-radius: 4px;
        box-sizing: border-box;
        font-size: 16px;
    }

    h1, h2, h3 {
        color: #333;
    }

    table {
        width: 100%;
        border-collapse: collapse;
        margin-top: 20px;
        background-color: #fffffe;
    }

    th, td {
        border: 1px solid #ddd;
        padding: 8px;
        text-align: center;
    }

    th {
        background-color: #d1e2d1;
        cursor: pointer;
        position: relative;
    }

    th:hover {
        background-color: #c0d1c0;
    }

    th .sort-indicator {
        position: absolute;
        right: 5px;
        top: 50%;
        transform: translateY(-50%);
        font-size: 0.8em;
    }

    .match-cell {
        width: 40px; /* Fixed width for match result cells */
        height: 40px;
        cursor: pointer;
        background-color: #e6ffe6; /* Light green for match cells */
        font-weight: bold;
        display: flex;
        align-items: center;
        justify-content: center;
    }

    .match-cell.win {
        background-color: #aaffaa; /* Darker green for win */
    }

    .match-cell.loss {
        background-color: #ffaaaa; /* Red for loss */
    }

    .match-cell:hover {
        background-color: #d0ffd0;
    }

    .summary-table th {
        background-color: #add8e6; /* Light blue for summary headers */
    }

    .summary-table tr:nth-child(even) {
        background-color: #f2f2f2;
    }

    button {
        padding: 10px 15px;
        font-size: 16px;
        background-color: #007bff;
        color: white;
        border: none;
        border-radius: 5px;
        cursor: pointer;
        margin-top: 10px;
    }

    button:hover {
        background-color: #0056b3;
    }
</style>

<h1>🏸 Badminton Round-Robin Tournament Tracker</h1>

<div class="container">
    <div class="input-section">
        <h3>Participants (one per line):</h3>
        <textarea bind:value={participantDetails}></textarea>
        <button on:click={resetTournament}>Reset Tournament</button>
    </div>

    <div class="table-section">
        <h2>Match Results Matrix</h2>
        <table>
            <thead>
                <tr>
                    <th></th> <!-- Empty corner cell -->
                    {#each participants as participantName, i}
                        <th style="writing-mode: vertical-lr; text-orientation: mixed;">{participantName}</th>
                    {/each}
                </tr>
            </thead>
            <tbody>
                {#each participants as player1Name, i}
                    <tr>
                        <th>{player1Name}</th>
                        {#each participants as player2Name, j}
                            {#if i === j}
                                <td style="background-color: #eee;">-</td> <!-- Self-match -->
                            {:else}
                                <td
                                    class="match-cell"
                                    class:win={matchResults[i][j] === 'W'}
                                    class:loss={matchResults[i][j] === 'L'}
                                    on:click={() => toggleMatchResult(i, j)}
                                >
                                    {matchResults[i][j] || ''}
                                </td>
                            {/if}
                        {/each}
                    </tr>
                {/each}
            </tbody>
        </table>
    </div>
</div>

<h2>Tournament Standings</h2>
<table class="summary-table">
    <thead>
        <tr>
            <th on:click={() => handleSort('rank')}>
                Rank
                {#if sortColumn === 'rank'}
                    <span class="sort-indicator">{sortDirection === 'asc' ? '▲' : '▼'}</span>
                {/if}
            </th>
            <th on:click={() => handleSort('name')}>
                Player/Team
                {#if sortColumn === 'name'}
                    <span class="sort-indicator">{sortDirection === 'asc' ? '▲' : '▼'}</span>
                {/if}
            </th>
            <th on:click={() => handleSort('played')}>
                Played
                {#if sortColumn === 'played'}
                    <span class="sort-indicator">{sortDirection === 'asc' ? '▲' : '▼'}</span>
                {/if}
            </th>
            <th on:click={() => handleSort('wins')}>
                Win
                {#if sortColumn === 'wins'}
                    <span class="sort-indicator">{sortDirection === 'asc' ? '▲' : '▼'}</span>
                {/if}
            </th>
            <th on:click={() => handleSort('losses')}>
                Lose
                {#if sortColumn === 'losses'}
                    <span class="sort-indicator">{sortDirection === 'asc' ? '▲' : '▼'}</span>
                {/if}
            </th>
            <th on:click={() => handleSort('points')}>
                Pts
                {#if sortColumn === 'points'}
                    <span class="sort-indicator">{sortDirection === 'asc' ? '▲' : '▼'}</span>
                {/if}
            </th>
        </tr>
    </thead>
    <tbody>
        {#each finalSortedPlayers as player}
            <tr>
                <td>{player.rank}</td>
                <td style="text-align: left;">{player.name}</td>
                <td>{player.played}</td>
                <td>{player.wins}</td>
                <td>{player.losses}</td>
                <td>{player.points}</td>
            </tr>
        {/each}
    </tbody>
</table> 