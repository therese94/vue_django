<template>
  <div class="home">
    <h1>Todo</h1>
    <TodoInput @createTodo="createTodo"/>
    <TodoList :todos="todos" />
  </div>
</template>

<script>
import axios from 'axios'
import jwtDecode from 'jwt-decode' // JWT의 payload값을 해석해서 보여주는 library
import { mapGetters} from 'vuex' // import vuex from 'vuex'; vuex.mapGetters
import TodoList from '@/components/TodoList' //호출
import TodoInput from '@/components/TodoInput'  //호출
import router from '@/router'

export default {
  name: 'Home',
  data() {
    return {
      todos: [],
    }
  },
  computed: {
    ...mapGetters([ 
      'isLoggedIn',
      'options',
      'userId'
    ])
  },
  components: {
    TodoInput,
    TodoList,
  },
  methods: {
    // 사용자 로그인 유무를 확인하여 로그인되어있지 않을 시 로그인 페이지로 보내겠다.
    checkLoggedIn() {
      // 1. 세션을 시작해서
      this.$session.start()
      // 2. 'jwt' 가 있는지 확인하겠다.
      if(!this.$session.has('jwt')){
        // 로그인 페이지로 보내겠다.
        router.push('/login')
      }
    },

    // 우리가 만든 django API 서버로 todos를 달라는 요청을 보낸 뒤
    // todos data 에 할당하는 함수
    getTodo() {
     this.$session.start()
     const token = this.$session.get('jwt')
     const userId = jwtDecode(token).user_id
     const SERVER_IP = process.env.VUE_APP_SERVER_IP

     const options = {
       headers: {
         Authorization: 'JWT ' + token
       }
     }
     axios.get(`${SERVER_IP}/api/v1/users/${userId}/`, options)
       .then(response => {
         console.log(token)
         this.todos = response.data.todo_set
       })
       .catch(error => {
         console.error(error)
       })
   },
  createTodo(title) {
    this.$session.start()
    const token = this.$session.get('jwt')
    const userId = jwtDecode(token).user_id
    const SERVER_IP = process.env.VUE_APP_SERVER_IP

    const options = {
      headers: {
        Authorization: 'JWT ' + token
      }
    }

    const data = {
      title,
      user: userId
    }

    axios.post(`${SERVER_IP}/api/v1/todos/`, data, options)
      .then(response => {
        this.todos.push(response.data)
      })
      .catch(error => {
         console.error(error)
       })

    
  },
 },

  // Vue 가 화면에 그려지면 실행하는 함수
  mounted() {
    if(this.isLoggedIn) {
      this.checkLoggedIn()
      this.getTodo()
    }
    
  },
  watch: {
    isLoggedIn() {
      this.checkLoggedIn()
      this.getTodo()
    }
  }
}
</script>

<style>

</style>