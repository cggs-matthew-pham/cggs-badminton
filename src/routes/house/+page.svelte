<script>
    import { onMount } from "svelte";

    class House {
        constructor(name) {
            this.name = name;
        }
    }

    class Match {
        constructor(house1, house2, isJunior) {
            this.house1 = house1;
            this.house2 = house2;
            this.isJunior = isJunior;
        }
    }

    class Round {
        constructor(matches, time, umpires) {
            this.matches = matches;
            this.time = time;
            this.umpires = umpires;
        }
    }

    const houseNames = ["Burgmann", "Deakin", "Glebe", "Kilburn", "Robertson", "Waverley"];
    const houses = {};
    houseNames.forEach(name => houses[name] = new House(name));
    function getHouse(name) { return houses[name]; }

    const roundTimes = [
        "9:10 – 9:20 am", "9:20 – 9:30 am", "9:30 – 9:40 am", "9:40 – 9:50 am",
        "9:50 – 10:00 am", "10:00 – 10:10 am", "10:10 – 10:20 am", "10:20 – 10:30 am",
        "10:30 – 10:40 am", "10:40 – 10:50 am", "10:50 – 11:00 am", "11:00 – 11:10 am",
        "11:10 – 11:20 am", "11:20 – 11:30 am", "11:30 – 11:40 am"
    ];

    const roundUmpires = [
        { juniors: "Waverley", seniors: "Robertson" },
        { juniors: "Burgmann", seniors: "Kilburn" },
        { juniors: "Deakin", seniors: "Kilburn" },
        { juniors: "Robertson", seniors: "Deakin" },
        { juniors: "Glebe", seniors: "Burgmann" },
        { juniors: "Kilburn", seniors: "Glebe" },
        { juniors: "Deakin", seniors: "Kilburn" },
        { juniors: "Burgmann", seniors: "Waverley" },
        { juniors: "Glebe", seniors: "Waverley" },
        { juniors: "Kilburn", seniors: "Glebe" },
        { juniors: "Robertson", seniors: "Burgmann" },
        { juniors: "Waverley", seniors: "Robertson" },
        { juniors: "Glebe", seniors: "Waverley" },
        { juniors: "Deakin", seniors: "Burgmann" },
        { juniors: "Robertson", seniors: "Deakin" }
    ];

    // Only one match per division per round
    const roundMatches = [
        {
            juniors: [["Burgmann", "Kilburn"]],
            seniors: [["Deakin", "Glebe"]]
        },
        {
            juniors: [["Deakin", "Glebe"]],
            seniors: [["Robertson", "Waverley"]]
        },
        {
            juniors: [["Robertson", "Waverley"]],
            seniors: [["Burgmann", "Glebe"]]
        },
        {
            juniors: [["Burgmann", "Glebe"]],
            seniors: [["Kilburn", "Waverley"]]
        },
        {
            juniors: [["Kilburn", "Waverley"]],
            seniors: [["Deakin", "Robertson"]]
        },
        {
            juniors: [["Deakin", "Robertson"]],
            seniors: [["Burgmann", "Waverley"]]
        },
        {
            juniors: [["Burgmann", "Waverley"]],
            seniors: [["Glebe", "Kilburn"]]
        },
        {
            juniors: [["Glebe", "Robertson"]],
            seniors: [["Kilburn", "Deakin"]]
        },
        {
            juniors: [["Kilburn", "Deakin"]],
            seniors: [["Burgmann", "Robertson"]]
        },
        {
            juniors: [["Burgmann", "Robertson"]],
            seniors: [["Waverley", "Deakin"]]
        },
        {
            juniors: [["Waverley", "Deakin"]],
            seniors: [["Glebe", "Kilburn"]]
        },
        {
            juniors: [["Glebe", "Kilburn"]],
            seniors: [["Burgmann", "Deakin"]]
        },
        {
            juniors: [["Burgmann", "Deakin"]],
            seniors: [["Robertson", "Kilburn"]]
        },
        {
            juniors: [["Robertson", "Kilburn"]],
            seniors: [["Waverley", "Glebe"]]
        },
        {
            juniors: [["Waverley", "Glebe"]],
            seniors: [["Burgmann", "Kilburn"]]
        }
    ];

    // Score state: [round][match][game][0=house1,1=house2]
    let scores = [];

    // Local storage key
    const SCORES_KEY = "badminton-scores";

    // Load from localStorage or initialize
    function loadScores() {
        const saved = localStorage.getItem(SCORES_KEY);
        if (saved) {
            try {
                const parsed = JSON.parse(saved);
                // Defensive: ensure structure matches
                if (Array.isArray(parsed) && parsed.length === roundMatches.length) {
                    return parsed;
                }
            } catch {}
        }
        // Default structure
        return roundMatches.map(round =>
            [...round.juniors, ...round.seniors].map(() =>
                [[0, 0], [0, 0], [0, 0]]
            )
        );
    }

    // Save to localStorage
    function saveScores() {
        localStorage.setItem(SCORES_KEY, JSON.stringify(scores));
    }

    onMount(() => {
        scores = loadScores();
    });

    // Build rounds
    let combinedRounds = roundMatches.map((round, i) => {
        let matches = [];
        round.juniors.forEach(([h1, h2]) => {
            matches.push(new Match(getHouse(h1), getHouse(h2), true));
        });
        round.seniors.forEach(([h1, h2]) => {
            matches.push(new Match(getHouse(h1), getHouse(h2), false));
        });
        return new Round(matches, roundTimes[i], roundUmpires[i]);
    });

    // Update and persist scores on input
    function updateScore(roundIndex, matchIndex, gameIndex, houseIndex, value) {
        // Defensive deep clone to trigger Svelte reactivity
        scores = scores.map((round, r) =>
            round.map((match, m) =>
                match.map((game, g) =>
                    Array.isArray(game) ? game.slice() : [0, 0]
                )
            )
        );
        scores[roundIndex][matchIndex][gameIndex][houseIndex] = +value;
        saveScores();
        console.log(scores);
    }

    // Helper: get all match results with points
    $: allMatchResults = (() => {
        let results = [];
        combinedRounds.forEach((round, roundIndex) => {
            round.matches.forEach((match, matchIndex) => {
                const games = scores[roundIndex]?.[matchIndex] ?? [[0,0],[0,0],[0,0]];
                const agg1 = games.reduce((sum, g) => sum + (g[0] || 0), 0);
                const agg2 = games.reduce((sum, g) => sum + (g[1] || 0), 0);
                let points1 = 0, points2 = 0;
                if (agg1 === 0 && agg2 === 0) {
                    // Not played
                } else if (agg1 > agg2) {
                    points1 = 3; points2 = 1;
                } else if (agg2 > agg1) {
                    points2 = 3; points1 = 1;
                } else {
                    points1 = 2; points2 = 2;
                }
                results.push({
                    house1: match.house1.name,
                    house2: match.house2.name,
                    points1,
                    points2
                });
            });
        });
        return results;
    })();

    // Calculate house points using match results (reactive)
    $: housePoints = houseNames.map(houseName => {
        let total = 0;
        allMatchResults.forEach(r => {
            if (r.house1 === houseName) total += r.points1;
            if (r.house2 === houseName) total += r.points2;
        });
        return { house: houseName, points: total };
    });

    let selectedHouse = "";
    let selectedCategory = ""; // "Junior", "Senior", or ""

    // Navigation function to scroll to sections
    function scrollToSection(sectionId) {
        document.getElementById(sectionId)?.scrollIntoView({ 
            behavior: 'smooth',
            block: 'start'
        });
    }

    // Export data to JSON file
    function exportData() {
        const data = {
            scores: scores,
            timestamp: new Date().toISOString(),
            version: "1.0"
        };
        
        const blob = new Blob([JSON.stringify(data, null, 2)], { type: 'application/json' });
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = `badminton-scores-${new Date().toISOString().split('T')[0]}.json`;
        document.body.appendChild(a);
        a.click();
        document.body.removeChild(a);
        URL.revokeObjectURL(url);
    }

    // Import data from JSON file
    function importData(event) {
        const file = event.target.files[0];
        if (!file) return;

        const reader = new FileReader();
        reader.onload = (e) => {
            try {
                const data = JSON.parse(e.target.result);
                
                // Validate the imported data structure
                if (data.scores && Array.isArray(data.scores)) {
                    // Additional validation to ensure structure matches
                    if (data.scores.length === roundMatches.length) {
                        scores = data.scores;
                        saveScores();
                        alert('Data imported successfully!');
                    } else {
                        alert('Invalid data structure: Round count mismatch');
                    }
                } else {
                    alert('Invalid JSON format');
                }
            } catch (error) {
                alert('Error parsing JSON file: ' + error.message);
            }
        };
        reader.readAsText(file);
        
        // Clear the input so the same file can be selected again
        event.target.value = '';
    }

    // Clear all scores
    function clearAllScores() {
        if (confirm('Are you sure you want to clear all scores? This action cannot be undone.')) {
            scores = roundMatches.map(round =>
                [...round.juniors, ...round.seniors].map(() =>
                    [[0, 0], [0, 0], [0, 0]]
                )
            );
            saveScores();
        }
    }

    // File input reference
    let fileInput;
</script>

<!-- Sticky Navigation Bar -->
<nav class="sticky-nav">
    <div class="nav-container">
        <h3>CGGS Badminton</h3>
        <div class="nav-links">
            <button on:click={() => scrollToSection('rounds')} class="nav-btn">Rounds</button>
            <button on:click={() => scrollToSection('points')} class="nav-btn">Points</button>
            <button on:click={() => scrollToSection('results')} class="nav-btn">Results</button>
            <div class="nav-divider"></div>
            <button on:click={exportData} class="nav-btn export-btn">Export</button>
            <button on:click={() => fileInput.click()} class="nav-btn import-btn">Import</button>
            <button on:click={clearAllScores} class="nav-btn clear-btn">Clear</button>
        </div>
    </div>
</nav>

<!-- Hidden file input for import -->
<input
    bind:this={fileInput}
    type="file"
    accept=".json"
    on:change={importData}
    style="display: none;"
/>

<div class="content">
    <h1>CGGS House Badminton Competition</h1>

    <!-- Data Management Section -->
    <section id="data-management" class="data-section">
        <details>
            <summary>Data Management</summary>
            <div class="data-controls">
                <button on:click={exportData} class="action-btn export">
                    📥 Export Scores
                </button>
                <button on:click={() => fileInput.click()} class="action-btn import">
                    📤 Import Scores
                </button>
                <button on:click={clearAllScores} class="action-btn clear">
                    🗑️ Clear All Scores
                </button>
                <div class="data-info">
                    <small>
                        Export: Save current scores as JSON file<br>
                        Import: Load scores from JSON file<br>
                        Clear: Reset all scores to zero
                    </small>
                </div>
            </div>
        </details>
    </section>

    <section id="rounds">
        <h2 class="section-title">Tournament Rounds</h2>
        {#each combinedRounds as round, roundIndex}
            <h3>Round {roundIndex + 1} <span style="font-size:1rem;font-weight:normal;">({round.time})</span></h3>
            <div style="margin-bottom:0.5em;">
                <b>Umpires:</b>
                <span>Juniors: {round.umpires.juniors}, Seniors: {round.umpires.seniors}</span>
            </div>
            <table>
                <thead>
                    <tr>
                        <th>Category</th>
                        <th>House 1</th>
                        <th>House 2</th>
                        <th>Game 1</th>
                        <th>Game 2</th>
                        <th>Game 3</th>
                    </tr>
                </thead>
                <tbody>
                    {#each round.matches as match, matchIndex}
                        <tr>
                            <td>{match.isJunior ? 'Junior' : 'Senior'}</td>
                            <td>{match.house1.name}</td>
                            <td>{match.house2.name}</td>
                            {#each [0,1,2] as gameIndex}
                                <td>
                                    <input type="number" min="0" style="width:3em"
                                        value={scores[roundIndex]?.[matchIndex]?.[gameIndex]?.[0] ?? 0}
                                        placeholder={match.house1.name}
                                        on:input={e => updateScore(roundIndex, matchIndex, gameIndex, 0, e.target.value)} />
                                    -
                                    <input type="number" min="0" style="width:3em"
                                        value={scores[roundIndex]?.[matchIndex]?.[gameIndex]?.[1] ?? 0}
                                        placeholder={match.house2.name}
                                        on:input={e => updateScore(roundIndex, matchIndex, gameIndex, 1, e.target.value)} />
                                </td>
                            {/each}
                        </tr>
                    {/each}
                </tbody>
            </table>
        {/each}
    </section>

    <section id="points">
        <h2 class="section-title">House Points</h2>
        <table>
            <thead>
                <tr>
                    <th>House</th>
                    <th>Points</th>
                </tr>
            </thead>
            <tbody>
                {#each housePoints as hp}
                    <tr>
                        <td>{hp.house}</td>
                        <td>{hp.points}</td>
                    </tr>
                {/each}
            </tbody>
        </table>
    </section>

    <section id="results">
        <h2 class="section-title">Results by Round</h2>
        <div class="filters">
            <label>
                Filter by house:
                <select bind:value={selectedHouse}>
                    <option value="">All Houses</option>
                    {#each houseNames as house}
                        <option value={house}>{house}</option>
                    {/each}
                </select>
            </label>
            <label style="margin-left:1em;">
                Filter by category:
                <select bind:value={selectedCategory}>
                    <option value="">All</option>
                    <option value="Junior">Junior</option>
                    <option value="Senior">Senior</option>
                </select>
            </label>
        </div>
        <table>
            <thead>
                <tr>
                    <th>Round</th>
                    <th>Category</th>
                    <th>House 1</th>
                    <th>Aggregate</th>
                    <th>House 2</th>
                    <th>Aggregate</th>
                    <th>Result</th>
                    <th>Points (H1)</th>
                    <th>Points (H2)</th>
                </tr>
            </thead>
            <tbody>
                {#each combinedRounds as round, roundIndex}
                    {#each round.matches as match, matchIndex}
                        {#if
                            (!selectedHouse || match.house1.name === selectedHouse || match.house2.name === selectedHouse)
                            && (!selectedCategory || (selectedCategory === "Junior" ? match.isJunior : !match.isJunior))
                        }
                        <tr>
                            <td>{roundIndex + 1}</td>
                            <td>{match.isJunior ? 'Junior' : 'Senior'}</td>
                            <td>{match.house1.name}</td>
                            <td>{(() => {
                                const games = scores[roundIndex]?.[matchIndex] ?? [[0,0],[0,0],[0,0]];
                                return games.reduce((sum, g) => sum + (g[0] || 0), 0);
                            })()}</td>
                            <td>{match.house2.name}</td>
                            <td>{(() => {
                                const games = scores[roundIndex]?.[matchIndex] ?? [[0,0],[0,0],[0,0]];
                                return games.reduce((sum, g) => sum + (g[1] || 0), 0);
                            })()}</td>
                            <td>{(() => {
                                const games = scores[roundIndex]?.[matchIndex] ?? [[0,0],[0,0],[0,0]];
                                const agg1 = games.reduce((sum, g) => sum + (g[0] || 0), 0);
                                const agg2 = games.reduce((sum, g) => sum + (g[1] || 0), 0);
                                if (agg1 === 0 && agg2 === 0) return "Not played";
                                if (agg1 > agg2) return `${match.house1.name} win`;
                                if (agg2 > agg1) return `${match.house2.name} win`;
                                return "Draw";
                            })()}</td>
                            <td>{(() => {
                                const games = scores[roundIndex]?.[matchIndex] ?? [[0,0],[0,0],[0,0]];
                                const agg1 = games.reduce((sum, g) => sum + (g[0] || 0), 0);
                                const agg2 = games.reduce((sum, g) => sum + (g[1] || 0), 0);
                                return agg1 > agg2 ? 3 : agg1 < agg2 ? 1 : agg1 === 0 && agg2 === 0 ? 0 : 2;
                            })()}</td>
                            <td>{(() => {
                                const games = scores[roundIndex]?.[matchIndex] ?? [[0,0],[0,0],[0,0]];
                                const agg1 = games.reduce((sum, g) => sum + (g[0] || 0), 0);
                                const agg2 = games.reduce((sum, g) => sum + (g[1] || 0), 0);
                                return agg2 > agg1 ? 3 : agg2 < agg1 ? 1 : agg1 === 0 && agg2 === 0 ? 0 : 2;
                            })()}</td>
                        </tr>
                        {/if}
                    {/each}
                {/each}
            </tbody>
        </table>
    </section>
</div>

<style>
    .sticky-nav {
        position: sticky;
        top: 0;
        background: #ffffff;
        border-bottom: 2px solid #ddd;
        z-index: 100;
        box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        margin-bottom: 20px;
    }

    .nav-container {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 10px 20px;
        max-width: 1200px;
        margin: 0 auto;
    }

    .nav-container h3 {
        margin: 0;
        color: #333;
        font-size: 1.2em;
    }

    .nav-links {
        display: flex;
        gap: 15px;
        align-items: center;
    }

    .nav-divider {
        width: 1px;
        height: 20px;
        background: #ddd;
        margin: 0 5px;
    }

    .nav-btn {
        background: #007bff;
        color: white;
        border: none;
        padding: 8px 16px;
        border-radius: 4px;
        cursor: pointer;
        font-size: 14px;
        transition: background-color 0.2s;
    }

    .nav-btn:hover {
        background: #0056b3;
    }

    .export-btn {
        background: #28a745;
    }

    .export-btn:hover {
        background: #218838;
    }

    .import-btn {
        background: #17a2b8;
    }

    .import-btn:hover {
        background: #138496;
    }

    .clear-btn {
        background: #dc3545;
    }

    .clear-btn:hover {
        background: #c82333;
    }

    .content {
        max-width: 1200px;
        margin: 0 auto;
        padding: 0 20px;
    }

    .data-section {
        background: #f8f9fa;
        border-radius: 8px;
        padding: 15px;
        margin-bottom: 30px;
        border: 1px solid #dee2e6;
    }

    .data-section summary {
        cursor: pointer;
        font-weight: bold;
        margin-bottom: 15px;
        color: #495057;
    }

    .data-controls {
        display: flex;
        gap: 15px;
        align-items: flex-start;
        flex-wrap: wrap;
    }

    .action-btn {
        padding: 10px 20px;
        border: none;
        border-radius: 6px;
        cursor: pointer;
        font-size: 14px;
        font-weight: 500;
        transition: all 0.2s;
    }

    .action-btn.export {
        background: #28a745;
        color: white;
    }

    .action-btn.export:hover {
        background: #218838;
    }

    .action-btn.import {
        background: #17a2b8;
        color: white;
    }

    .action-btn.import:hover {
        background: #138496;
    }

    .action-btn.clear {
        background: #dc3545;
        color: white;
    }

    .action-btn.clear:hover {
        background: #c82333;
    }

    .data-info {
        margin-left: auto;
        padding: 10px;
        background: white;
        border-radius: 4px;
        border: 1px solid #dee2e6;
        max-width: 300px;
    }

    section {
        margin-bottom: 40px;
        scroll-margin-top: 80px; /* Offset for sticky nav */
    }

    .section-title {
        border-bottom: 2px solid #007bff;
        padding-bottom: 10px;
        margin-bottom: 20px;
    }

    .filters {
        margin-bottom: 15px;
        padding: 10px;
        background: #f8f9fa;
        border-radius: 4px;
    }

    table {
        width: 100%;
        border-collapse: collapse;
        margin-bottom: 20px;
    }
    
    th, td {
        border: 1px solid #ddd;
        padding: 8px;
        text-align: left;
    }
    
    th {
        background-color: #f2f2f2;
    }
    
    input[type="number"] {
        font-size: 1em;
        padding: 2px 4px;
        margin: 0 2px;
        box-sizing: border-box;
    }

    /* Mobile responsiveness */
    @media (max-width: 768px) {
        .nav-container {
            flex-direction: column;
            gap: 10px;
        }

        .nav-links {
            flex-wrap: wrap;
            justify-content: center;
        }

        .content {
            padding: 0 10px;
        }

        .data-controls {
            flex-direction: column;
            align-items: stretch;
        }

        .data-info {
            margin-left: 0;
            margin-top: 10px;
            max-width: none;
        }
    }
</style>
