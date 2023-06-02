<script>
    import { ref } from 'vue'

    export default {
        setup () {
            const items = ref([
                { id: 0, title: 'X', list: 1, draggble: true },
                { id: 1, title: 'O', list: 1, draggble: true },
            ])

            const getList = (list) => {
                return items.value.filter((item) => item.list == list)
            }

            const startDrag = (event, item) => {
                // console.log(item)

                // specify we are moving, not duplicating an item
                event.dataTransfer.dropEffect = 'copy'
                event.dataTransfer.effectAllowed = 'copy'

                // tell drag and drop API what we're moving
                event.dataTransfer.setData('itemID', item.id)
            }

            const onDrop = (event, list) => {
                // check if list already has content and return if so
                // ADD CHECK HERE
                if (items.value.find((item) => item.list == list)) {
                    console.log('there is already a piece here')
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
            v-for="item in getList(1)" 
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
                @drop="onDrop($event, n + 1)"
                @dragenter.prevent
                @dragover.prevent
            >
                <div 
                    v-for="item in getList(n + 1)" 
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
    /* background-color: aquamarine; */
    border: solid;
    margin-bottom: 2rem;
}

.grid-container {
    display: grid;
    grid-template-columns: repeat(3, 50px);
    grid-template-rows: repeat(3, 50px);
    border: solid red;
}

.grid-square {
    /* width: 50%; */
    border: solid;
    background-color: lightblue;
    /* padding: 100px; */
    min-height: 50px;
    min-width: 50px;
    text-align: center;
    grid-area: square;
}

.drag-el{
    /* background-color: #3498db; */
    /* color: white; */
    padding: 5px;

}

</style>
