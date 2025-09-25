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
            seniors: [["Burgmann", "Kilburn"]]
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
            seniors: [["Burgmann", "Robertson"]]
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
</script>

<h1>CGGS House Badminton Competition</h1>

{#each combinedRounds as round, roundIndex}
    <h2>Round {roundIndex + 1} <span style="font-size:1rem;font-weight:normal;">({round.time})</span></h2>
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

<h2>House Points</h2>
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

<h2>Results by Round</h2>
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

<style>
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
</style>
