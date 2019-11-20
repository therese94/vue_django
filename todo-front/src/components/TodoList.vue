<template>
  <div class="todo-list">
      <div class="card mb-2" v-for="todo in todos" :key="todo.id">
          <div class="card-body d-flex justify-content-between">
              <span>{{ todo.title }}</span>
              <span @click="deleteTodo(todo)">삭제</span>
          </div>
      </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
    name: 'TodoList',
    props: {
        todos: {
            type: Array,
            required : true,
        }
    },
    computed: {
        options() {
            return this.$store.getters.options
        }
    },
    methods: {
        deleteTodo(todo) {
            //요청 보낼 시 헤더에 token실어서 보내야함
            // this.$session.start()
            // const token = this.$session.get('jwt')
            // const options = {
            //     headers: {
            //         Authorization: 'JWT ' + token
            //     }
            // }
            const SERVER_IP = process.env.VUE_APP_SERVER_IP

            axios.delete(`${SERVER_IP}/api/v1/todos/${todo.id}/`, this.options)
                .then(response => {
                    console.log(response)
                    // const targetTodo = this.todos.find(item => {            // namespace겹치니까 todo 로 안하고 item으로 씀
                    //     return item === todo
                    // })
                    const idx = this.todos.indexOf(todo)
                    if (idx > -1) {
                        this.todos.splice(idx,1)            // splice는 배열을 자르겠다는거
                    }

                })
                .catch(error => {
                    console.error(error)
                })
        },
    }
}
</script>

<style>

</style>