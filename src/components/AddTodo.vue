<template>
        
    <div class="formInput">
        <input type="text" placeholder="輸入代辦事項" v-model="newTodo">
        <input type="text" placeholder="簽名" v-model="name">
        <button @click="addTodo">新增</button>
    </div>
</template>

<script>
import gql from 'graphql-tag'
const add_todo=gql`
        mutation addTodo($todo: String!) {
        insert_todo_list(objects: {task: $todo}) {
            returning {
            id
            created_at
            task
            }
        }
        }

    `
export default {
    props:['type'],
    data() {
        return {
            newTodo: '',
            name:"",
        };
    },
    methods: {
        addTodo:function(){
            let task= this.newTodo
            
            this.$apollo.mutate({
                mutation:add_todo,
                variable:{
                    task: task,
                }
            })
        }
    },
}
</script>