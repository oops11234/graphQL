<template>
  <div class="hello">
     <div class="formInput">
        <input type="text" placeholder="輸入代辦事項" v-model="newTodo">
        <input type="text" placeholder="創建人名稱" v-model="assignee">
        <button @click="addTodo" :disabled="newTodo===''|| assignee===''">新增</button>
    </div>
    <div v-if="error">{{error}}</div>
      <div v-for="todo in todo_list" :key="todo.id">
        <div>創建人:{{todo.assignee}}</div>
      <div> id:{{todo.id}}</div>
      <template v-if="edit===todo.id">
        <input type="text" v-model="updateTodo">
        <br>
        <button @click="confirmUpdate(todo.id)" :disabled="updateTodo===''"> 確認修改</button>
      </template>
      <template v-else>
          <div >項目:{{ todo.task }}</div>
          <div>
        <button @click="deleteTodo(todo.id)">刪除</button> 
        <button @click="editTodo(todo.id)">修改</button>
      </div>
      </template>
      <hr /> 
      
    </div>
  </div>
</template>

<script>
import gql from "graphql-tag";
  const add_todo=gql`
        mutation addTodo($todo: String! , , $name: String) {
        insert_todo_list(objects: {task: $todo,, assignee: $name}) {
            returning {
            id
            created_at
            task
            }
        }
        }

    `
    const remove_todo=gql`
    mutation delete($id: Int!) {
    delete_todo_list(where: {id: {_eq: $id}}) {
    affected_rows
  }
}
    `
    const get_todo=gql`
        query todo_list {
          todo_list(order_by: {id: desc}, limit: 20) {
            id
            assignee
            created_at
            task
            updated_at
          }
        }
      `
    const update_todo=gql`
    mutation update($id: Int!, $todo: String!) {
      update_todo_list(where: {id: {_eq: $id}}, _set: {task: $todo}) {
        affected_rows
        returning {
          id
          task
    }
  }
}
    `

export default {
  name: "HelloWorld",
  data() {
    return {
      todo_list: {},
      error:null,
      assignee:"",
      newTodo:'',
      updateTodo:'',
      edit:null,
    };
  },
  apollo: {
    todo_list: {
      query: get_todo,
      error(error){
        this.error= JSON.stringify(error.message)
      },
      loadingKey: "loading",
      result({ data }) {
        this.todo_list = data.todo_list;
      },
    },
  },
  props: {
    msg: String,
  },
  methods: {
        addTodo(){
            this.$apollo.mutate({
                mutation:add_todo,
                variables:{
                    todo: this.newTodo,
                    name:this.assignee
                },
            })
             if(confirm("輸入完成")){
              window.location.reload()
             }
        },

        editTodo(id){
          this.edit=id
        },

        confirmUpdate(id){
        this.$apollo.mutate({
                mutation:update_todo,
                variables:{
                    todo: this.updateTodo,
                    id:id
                },
            })
             if(confirm("輸入完成")){
              window.location.reload()
             }
        },
        deleteTodo(id){
          if(confirm("確認刪除")){
            this.$apollo.mutate({
            mutation:remove_todo,
            variables:{
                    id: id
                },
          })}
         if(confirm("已刪除")){
              window.location.reload()
             }
        }    
        },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.formInput{
  margin-bottom: 30px;
  border-bottom: 1px solid #000;
}
.formInput>input{
  margin: 30px 20px;
}
</style>
