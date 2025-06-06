<script>
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
        constructor(matches) {
            this.matches = matches;
        }
    }

    const houses = [
        new House("Burgmann"),
        new House("Deakin"),
        new House("Glebe"),
        new House("Kilburn"),
        new House("Robertson"),
        new House("Waverly")
    ];

    function generateRoundRobin(houses, isJunior) {
        let rounds = [];
        let n = houses.length;
        let matchesPerRound = n / 2;

        for (let i = 0; i < n - 1; i++) {
            let round = [];
            for (let j = 0; j < matchesPerRound; j++) {
                let house1 = houses[j];
                let house2 = houses[n - 1 - j];
                if (j !== 0 || i % 2 === 0) {
                    round.push(new Match(house1, house2, isJunior));
                } else {
                    round.push(new Match(house2, house1, isJunior));
                }
            }
            rounds.push(round);
            houses.splice(1, 0, houses.pop());
        }
        return rounds;
    }

    let seniorRounds = generateRoundRobin([...houses], false);
    let juniorRounds = generateRoundRobin([...houses], true);

    let combinedRounds = [];
    for (let i = 0; i < seniorRounds.length; i++) {
        combinedRounds.push(new Round([...seniorRounds[i], ...juniorRounds[i]]));
    }
</script>

<h1>CGGS House Badminton Competition</h1>

{#each combinedRounds as round, roundIndex}
    <h2>Round {roundIndex + 1}</h2>
    <table>
        <thead>
            <tr>
                <th>Court</th>
                <th>House 1</th>
                <th>House 2</th>
                <th>Category</th>
            </tr>
        </thead>
        <tbody>
            {#each round.matches as match, matchIndex}
                <tr>
                    <td>{matchIndex + 1}</td>
                    <td>{match.house1.name}</td>
                    <td>{match.house2.name}</td>
                    <td>{match.isJunior ? 'Junior' : 'Senior'}</td>
                </tr>
            {/each}
        </tbody>
    </table>
{/each}

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
</style>
