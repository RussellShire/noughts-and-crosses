<script>
    import { ref } from 'vue'

    export default {
        setup () {
            const items = ref([
                { id: 0, title: 'Item A', list: 1 },
                { id: 1, title: 'Item B', list: 1 },
                { id: 2, title: 'Item C', list: 2 },
                { id: 3, title: 'Item D', list: 2 },
            ])

            const getList = (list) => {
                return items.value.filter((item) => item.list == list)
            }

            const startDrag = (event, item) => {
                console.log(item)

                // specify we are moving, not duplicating an item
                event.dataTransfer.dropEffect = 'move'
                event.dataTransfer.effectAllowed = 'move'

                // tell drag and drop API what we're moving
                event.dataTransfer.setData('itemID', item.id)
            }

            const onDrop = (event, list) => {
                // getting the id from the event datatransfer
                const itemID = event.dataTransfer.getData('itemID')

                //finding the item that matches 
                const item = items.value.find((item) => item.id = itemID)

                // changing the list it belongs to
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
    <div 
        class="drop-zone"
        @drop="onDrop($event, 1)"
        @dragenter.prevent
        @dragover.prevent
    >
        <div 
            v-for="item in getList(1)" 
            :key="item.id" 
            class="drag-el"
            draggable="true"
            @dragStart="startDrag($event, item)" 
        >
            {{ item.title }}
        </div>
    </div>

    <div 
        class="drop-zone"
        @drop="onDrop($event, 2)"
        @dragenter.prevent
        @dragover.prevent
    >
        <div 
            v-for="item in getList(2)" 
            :key="item.id" 
            class="drag-el"
            draggable="true"
            @dragStart="startDrag($event, item)"
        >
            {{ item.title }}
        </div>
    </div>
</template>

<style>
.drop-zone {
    width: 50%;
    margin: 50px auto;
    background-color: lightblue;
    padding: 10px;
    min-height: 10px;
    text-align: center;
}

.drag-el{
    background-color: #3498db;
    color: white;
    padding: 5px;
    margin-bottom: 10px;
}

.drag-el:nth-last-of-type(1) {
    margin-bottom: 0;
}
</style>
