<script>
    import { ref } from 'vue'

    export default {
        setup () {
            const items = ref([
                { id: 0, title: 'X', location: 0 },
                { id: 1, title: 'O', location: 0 },
            ])

            let message = ref('')
            let computerPiece = ''
            let playerPiece = ''
            let gameover = false

            const victoryConditions= [[1,2,3],[4,5,6],[7,8,9],[1,4,7],[2,5,8],[3,6,9],[1,5,9],[3,5,7]]

            const getLocation = (location) => {
                return items.value.filter((item) => item.location == location)
            }

            const startDrag = (event, item) => {
                // specify we are moving, not duplicating an item
                event.dataTransfer.dropEffect = 'copy'
                event.dataTransfer.effectAllowed = 'copy'

                // tell drag and drop API what we're moving
                event.dataTransfer.setData('itemID', item.id)

                // check for new game and reset state if so
                if (gameover) {
                    items.value = [
                        { id: 0, title: 'X', location: 0 },
                        { id: 1, title: 'O', location: 0 },
                    ]
                    computerPiece = ''
                    playerPiece = ''
                    message.value = 'Starting a new game'
                    gameover = false
                }
            }

            const onDrop = (event, location) => {
                // check if location already has content and return if so
                if (items.value.find((item) => item.location == location)) {
                    message.value = 'There is already a piece there'
                    return
                }
                
                // getting the id from the event datatransfer
                const itemID = event.dataTransfer.getData('itemID')

                //finding the item that matches 
                const item = items.value.find((item) => item.id == itemID)
                
                // Set playerPiece and checking player is playing the correct piece
                if (!playerPiece) { 
                    playerPiece = item.title
                } else if (playerPiece != item.title) {
                    message.value = 'That is not your piece'
                    return
                }

                message.value = ''

                // duplicating item
                let newItem = {...item}
                // changing id on duplicated item
                newItem.id = items.value.length
                // pushing duplicated item into items
                items.value.push(newItem)

                // changing the location new item belongs to
                newItem.location = location

                checkVictory(item.title)

                if (item.title === 'X') {
                    computerPiece = 'O'
                } else {
                    computerPiece = 'X'
                }

                computerTurn(computerPiece)
            }

            const checkVictory = ( currentTurnPiece ) => {
                // create an array of the squares where the piece in play sits
                const piecesInPlay = items.value
                    .filter(item => item.title == currentTurnPiece && item.location != 0)
                    .map(item => item.location)
                
                // no victory possible if three pieces haven't been played
                if (piecesInPlay.length < 3) {
                    return
                }

                // check if all three positions of each victory condition appears in the array of pieces in play
                for (let i = 0; i < victoryConditions.length; i++) {
                    const victoryCondition = victoryConditions[i]

                    const victory = piecesInPlay.includes(victoryCondition[0]) && piecesInPlay.includes(victoryCondition[1]) && piecesInPlay.includes(victoryCondition[2])
                
                    if (victory) {
                        // victoryCondition can be used to highlight a win
                        // console.log('Victory! ', victoryCondition)
                        if (computerPiece === currentTurnPiece) {
                            message.value = 'Defeat! The computer won.'
                        } else {
                            message.value = 'Victory! You beat the computer.'
                        }
                        gameover = true
                        return
                    } else {
                        if (items.value.length === 11) {
                            gameover = true
                            message.value = 'Draw!'
                        }
                    }
                }
            }

            // Logic for computer turn
            const findWinningMoves = (currentTurnPiece) => {
                const piecesInPlay = items.value
                    .filter(item => item.title == currentTurnPiece && item.location != 0)
                    .map(item => item.location)
                
                let winningMoves = []

                // Loop over each victory condition
                for (let i = 0; i < victoryConditions.length; i++) {
                    const victoryCondition = victoryConditions[i]
                    
                    // If any pieces are currently in place for that victory condition add them to an array
                    const piecesInPosition = piecesInPlay.filter(piece => {
                        if (piece === victoryCondition[0] || piece === victoryCondition[1] || piece === victoryCondition[2]) {
                            return true
                        } else {
                            return false
                        }
                    })

                    let possibleWinningRow = ''
                    
                    // If two pieces are in position, this victory condition is possible
                    if (piecesInPosition.length === 2) {
                        possibleWinningRow = victoryCondition    
                    } else {
                        continue
                    }

                    let contestedPosition = ''

                    // Find the position where no active pieces are
                    possibleWinningRow.forEach(position => {
                        if (position != piecesInPosition[0] && position != piecesInPosition[1]) {
                            contestedPosition = position
                        }
                    })

                    // Check the items in play to see if an opponent piece is in the contested position
                    let blockingPiece = items.value.filter(item => {
                        if (item.location === contestedPosition) {
                            return true
                        }
                    })

                    // If no pieces are found, this location is a possible winning move
                    if (blockingPiece.length === 0){
                        winningMoves.push(contestedPosition)
                    }   
                }
                return winningMoves
            }

            const findEmptyLocations = () => {
                let emptyLocations = [1,2,3,4,5,6,7,8,9]

                for (let i = 2; i < items.value.length; i++) {
                    emptyLocations.splice(emptyLocations.indexOf(items.value[i].location), 1)
                }

                return emptyLocations
            }

            const makeRandomMove = (possibleMoves) => {
                const move = Math.floor(Math.random() * (possibleMoves.length))

                let newItem = {id: items.value.length, title: computerPiece, location: possibleMoves[move]}

                items.value.push(newItem)

                checkVictory(computerPiece)
            }

            const computerTurn = (computerPiece) => {
                if (gameover) {
                    return
                }

                // Check if you can win
                let possibleMoves = ''
                possibleMoves = findWinningMoves(computerPiece)

                // if you can win, place a piece in a random winning position
                if (possibleMoves.length > 0) {
                    makeRandomMove(possibleMoves)
                    return
                }

                // Find opponent Piece
                let opponentPiece = ''
                
                if (computerPiece === 'X') {
                    opponentPiece = 'O'
                } else {
                    opponentPiece = 'X'
                }

                // Check to see if opponent can win
                possibleMoves = findWinningMoves(opponentPiece)

                // If oppent can win, randomly block one winning move
                if (possibleMoves.length > 0) {
                    makeRandomMove(possibleMoves)
                    return
                }

                // else make a random move
                possibleMoves = findEmptyLocations()

                makeRandomMove(possibleMoves)
            }

            const closeMessageBox = () => {
                message.value = ''
            }

            return {
                getLocation,
                startDrag,
                onDrop,
                closeMessageBox,
                message
            }
        }
    }
</script>

<template>
    <div class="container">
        <div class="piece-container">
            <h1>Grab your piece here</h1>
            <div 
                v-for="item in getLocation(0)" 
                :key="item.id" 
                class="drag-el"
                draggable="true"
                @dragstart="startDrag($event, item)" 
            >
                {{ item.title }}
            </div>
        </div>

        <div class="grid-container">
            <div v-for="n in 9">
                <div
                    class="grid-square"
                    @drop="onDrop($event, n)"
                    @dragenter.prevent
                    @dragover.prevent
                >
                    <div 
                        v-for="item in getLocation(n)" 
                        :key="item.id" 
                        class="drag-el"
                        draggable="false"
                    >
                        {{ item.title }}
                    </div>
                </div>
            </div>
        </div>

        <div v-if="message" class="message-box" @click="closeMessageBox">
            <div class="message">{{ message }} X</div>
        </div>

    </div>
</template>

<style>
.container {
    display: flex;
    flex-direction: column;
    align-items: center;
    border: solid pink;
}

.piece-container {
    border: solid yellow;

    margin-bottom: 1rem;
    display: flex;
    flex-direction: column;
    align-items: center;
}

.grid-container {
    /* border: solid red; */

    display: grid;
    grid-template-columns: repeat(3, 50px);
    grid-template-rows: repeat(3, 50px);
}

.grid-square {
    border: solid;
    /* background-color: lightblue; */
    min-height: 50px;
    min-width: 50px;
    display: flex;
    justify-content: center;
    align-items: center;
    grid-area: square;
}

.message-box {
    display: flex;
    margin: 1rem;
}

.message {
    margin: auto;
    border: solid;
    border-radius: 0.5rem;
    gap: 0.5rem;
    padding: 0.5rem;
}

/* .drag-el{

} */
</style>
