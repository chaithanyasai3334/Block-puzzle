# Block-puzzle
/*HTML*/
<!DOCTYPE html>
<html>
<head>
    <title>Block Puzzle</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <h1>Candy Crush Lite</h1>
    <div id="game-container">
        <div id="score-display">Score: 0</div>
        <div id="game-grid"></div>
    </div>
    <script src="script.js"></script>
</body>
</html>

/*CSS*/
body {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    margin: 0;
    background-color:grey;
    font-family: 'Arial', sans-serif;
    color: black;
}

h1 {
    color:black;
    margin-bottom: 25px;
}

#game-container {
    background-color:black;
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 0 20px rgba(0, 0, 0, 0.5);
    text-align: center;
}

#score-display {
    font-size: 1.5em;
    margin-bottom: 15px;
    font-weight: bold;
    color: red;
}

#game-grid {
    display: grid;
    border: 3px solid black;
}

.candy {
    width: 50px;
    height: 50px;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 2em;
    cursor: pointer;
    border: 2px solid #7f8c8d;
    box-sizing: border-box;
    transition: transform 0.1s ease-out, border 0.1s ease-out;
}

.candy.selected {
    border: 4px solid black;
    transform: scale(1.05);
}

.candy.moving {
    transition: top 0.2s ease-out, left 0.2s ease-out;
}

.candy-type-0 { background-color: green; } 
.candy-type-1 { background-color: black; } 
.candy-type-2 { background-color: purple; }
.candy-type-3 { background-color: grey; } 
.candy-type-4 { background-color: yellow; } 
.candy-type-5 { background-color: white; } 

/*JAVA SCRIPR*/
const gameGrid = document.getElementById('game-grid');
const scoreDisplay = document.getElementById('score-display');

const gridSize = 8;
const numCandyTypes = 6;
const candies = [];
let score = 0;

let firstSelectedCandy = null;
let isAnimating = false;

function createCandyElement(type, row, col) {
    const candy = document.createElement('div');
    candy.classList.add('candy', `candy-type-${type}`);
    candy.dataset.type = type;
    candy.dataset.row = row;
    candy.dataset.col = col;
    candy.addEventListener('click', handleCandyClick);
    return candy;
}

function initializeGame() {
    gameGrid.style.gridTemplateColumns = `repeat(${gridSize}, 1fr)`;
    gameGrid.style.gridTemplateRows = `repeat(${gridSize}, 1fr)`;

    for (let r = 0; r < gridSize; r++) {
        candies[r] = [];
        for (let c = 0; c < gridSize; c++) {
            let type;
            do {
                type = Math.floor(Math.random() * numCandyTypes);
            } while (hasInitialMatch(r, c, type));
            candies[r][c] = type;
            gameGrid.appendChild(createCandyElement(type, r, c));
        }
    }
    updateScoreDisplay();
}

function hasInitialMatch(row, col, type) {
    if (col >= 2 && candies[row][col - 1] === type && candies[row][col - 2] === type) return true;
    if (row >= 2 && candies[row - 1][col] === type && candies[row - 2][col] === type) return true;
    return false;
}

function updateGameGridUI() {
    gameGrid.innerHTML = '';
    for (let r = 0; r < gridSize; r++) {
        for (let c = 0; c < gridSize; c++) {
            const candy = createCandyElement(candies[r][c], r, c);
            gameGrid.appendChild(candy);
        }
    }
}

function updateScoreDisplay() {
    scoreDisplay.textContent = `Score: ${score}`;
}

function handleCandyClick(event) {
    if (isAnimating) return;

    const clickedCandy = event.target;
    const row = parseInt(clickedCandy.dataset.row);
    const col = parseInt(clickedCandy.dataset.col);

    if (!firstSelectedCandy) {
        firstSelectedCandy = { element: clickedCandy, row, col, type: parseInt(clickedCandy.dataset.type) };
        clickedCandy.classList.add('selected');
    } else {
        const secondSelectedCandy = { element: clickedCandy, row, col, type: parseInt(clickedCandy.dataset.type) };

        if (isValidSwap(firstSelectedCandy, secondSelectedCandy)) {
            performSwap(firstSelectedCandy, secondSelectedCandy);
            firstSelectedCandy.element.classList.remove('selected');
            firstSelectedCandy = null;
            
            setTimeout(() => {
                findAndClearMatches();
            }, 300);

        } else {
            firstSelectedCandy.element.classList.remove('selected');
            firstSelectedCandy = null;
        }
    }
}

function isValidSwap(candy1, candy2) {
    const rowDiff = Math.abs(candy1.row - candy2.row);
    const colDiff = Math.abs(candy1.col - candy2.col);
    return (rowDiff === 1 && colDiff === 0) || (rowDiff === 0 && colDiff === 1);
}

function performSwap(candy1, candy2) {
    isAnimating = true;

    [candies[candy1.row][candy1.col], candies[candy2.row][candy2.col]] =
    [candies[candy2.row][candy2.col], candies[candy1.row][candy1.col]];

    updateGameGridUI();

    const currentMatches = getMatches();
    if (currentMatches.length === 0) {
        setTimeout(() => {
            [candies[candy1.row][candy1.col], candies[candy2.row][candy2.col]] =
            [candies[candy2.row][candy2.col], candies[candy1.row][candy1.col]];
            updateGameGridUI();
            isAnimating = false;
        }, 300);
    } else {
        isAnimating = false;
    }
}

function getMatches() {
    const matches = new Set();

    for (let r = 0; r < gridSize; r++) {
        for (let c = 0; c < gridSize; c++) {
            const type = candies[r][c];

            if (type === undefined) continue;

            const horizontalMatch = [];
            for (let i = c; i < gridSize && candies[r][i] === type; i++) {
                horizontalMatch.push(`${r}-${i}`);
            }
            if (horizontalMatch.length >= 3) {
                horizontalMatch.forEach(m => matches.add(m));
            }

            const verticalMatch = [];
            for (let i = r; i < gridSize && candies[i][c] === type; i++) {
                verticalMatch.push(`${i}-${c}`);
            }
            if (verticalMatch.length >= 3) {
                verticalMatch.forEach(m => matches.add(m));
            }
        }
    }
    return Array.from(matches).map(coords => {
        const [row, col] = coords.split('-').map(Number);
        return { row, col };
    });
}

function findAndClearMatches() {
    isAnimating = true;
    let matchesFound = false;

    const matches = getMatches();
    if (matches.length > 0) {
        matchesFound = true;
        score += matches.length * 10;
        updateScoreDisplay();

        matches.forEach(({ row, col }) => {
            candies[row][col] = undefined;
            const element = gameGrid.querySelector(`[data-row="${row}"][data-col="${col}"]`);
            if (element) {
                element.style.opacity = '0';
                element.style.transform = 'scale(0.5)';
            }
        });

        setTimeout(() => {
            dropCandies();
            setTimeout(() => {
                fillNewCandies();
                setTimeout(() => {
                    if (getMatches().length > 0) {
                        findAndClearMatches();
                    } else {
                        isAnimating = false;
                    }
                }, 400);
            }, 300);
        }, 400);
    } else {
        isAnimating = false;
    }
}

function dropCandies() {
    for (let c = 0; c < gridSize; c++) {
        let emptySpaces = 0;
        for (let r = gridSize - 1; r >= 0; r--) {
            if (candies[r][c] === undefined) {
                emptySpaces++;
            } else if (emptySpaces > 0) {
                candies[r + emptySpaces][c] = candies[r][c];
                candies[r][c] = undefined;
            }
        }
    }
    updateGameGridUI();
}

function fillNewCandies() {
    for (let r = 0; r < gridSize; r++) {
        for (let c = 0; c < gridSize; c++) {
            if (candies[r][c] === undefined) {
                candies[r][c] = Math.floor(Math.random() * numCandyTypes);
            }
        }
    }
    updateGameGridUI();
}

initializeGame();
