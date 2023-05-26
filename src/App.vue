<script>
    import Draggable from "vue3-draggable";

    export default {
        components: {
            Draggable,
        },
        data() {
            return {
                items: [
                    {
                        "id": 1,
                        "title": One
                    },
                    {
                        "id": 2,
                        "title": Two
                    },
                    {
                        "id": 3,
                        "title": Three
                    },
                ]
            }
            
        }
    };
</script>
// test comment
<template>
    <draggable v-model="items">
        <template v-slot:item="{item}">
            <!-- example -->
            <div>
                {{item.title}}
            </div>
            <!-- or your own template -->
        </template>
    </draggable>
<div></div>
</template>

<style scoped>
/*  */
</style>
