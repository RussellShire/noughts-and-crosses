<script>
    import { ref } from 'vue'

    export default {
        setup () {
            const items = ref([
                { id: 0, title: 'X', list: 0 },
                { id: 1, title: 'O', list: 0 },
            ])

            const getList = (list) => {
                return items.value.filter((item) => item.list == list)
            }

            const startDrag = (event, item) => {
                // specify we are moving, not duplicating an item
                event.dataTransfer.dropEffect = 'copy'
                event.dataTransfer.effectAllowed = 'copy'

                // tell drag and drop API what we're moving
                event.dataTransfer.setData('itemID', item.id)
            }

            const onDrop = (event, list) => {
                // check if list already has content and return if so
                if (items.value.find((item) => item.list == list)) {
                    alert('there is already a piece here')
                    return
                }
                
                // getting the id from the event datatransfer
                const itemID = event.dataTransfer.getData('itemID')

                //finding the item that matches 
                const item = items.value.find((item) => item.id == itemID)
                
                // duplicating item
                let newItem = {...item}
                // changing id on duplicated item
                newItem.id = items.value.length
                // pushing duplicated into items
                items.value.push(newItem)
                
                // changing the list dragged item belongs to
                item.list = list

                checkVictory(item.title)
            }

            const checkVictory = (piece) => {
                const victoryConditions= [[1,2,3],[4,5,6],[7,8,9],[1,4,7],[2,5,8],[3,6,9],[1,5,9],[3,5,7]]

                // create an array of the squares where the piece in play sits
                const piecesInPlay = items.value
                    .filter(item => item.title == piece && item.list != 0)
                    .map(item => item.list)
                
                // no victory possible if three pieces haven't been played
                if(piecesInPlay.length < 3) {
                    return
                }

                // check if all three positions of each victory condition appears in the array of pieces in play
                for (let i = 0; i < victoryConditions.length; i++) {
                    const victoryCondition = victoryConditions[i]

                    const victory = piecesInPlay.includes(victoryCondition[0]) && piecesInPlay.includes(victoryCondition[1]) && piecesInPlay.includes(victoryCondition[2])
                
                    if (victory) {
                        // victoryCondition can be used to highlight a win
                        console.log(victoryCondition)
                        return
                    }
                }
            }

            return {
                getList,
                startDrag,
                onDrop,
            }
        }
    }
</script>

<template>
    <div class="piece-container">
        <h1>Grab your piece here</h1>
        <div 
            v-for="item in getList(0)" 
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
                    v-for="item in getList(n)" 
                    :key="item.id" 
                    class="drag-el"
                    draggable="false"
                >
                    {{ item.title }}
                </div>
            </div>
        </div>
    </div>
</template>

<style>
.piece-container {
    border: solid yellow;
    margin-bottom: 2rem;
}

.grid-container {
    display: grid;
    grid-template-columns: repeat(3, 50px);
    grid-template-rows: repeat(3, 50px);
    border: solid red;
}

.grid-square {
    border: solid;
    background-color: lightblue;
    min-height: 50px;
    min-width: 50px;
    display: flex;
    justify-content: center;
    align-items: center;
    grid-area: square;
}

/* .drag-el{

} */
</style>
