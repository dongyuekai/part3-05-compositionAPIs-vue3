<template>
  <section id='app' class='todoapp'>
     <header class="header">
       <h1>todos</h1>
       <input
          class="new-todo"
          placeholder="What needs to be done?"
          autocomplete="off"
          autofocus
          v-model="input"
          @keyup.enter='addTodo'
       >
     </header>
     <section class='main' v-show='count'>
        <input type="checkbox" id="toggle-all" class="toggle-all" v-model="allDone">
        <label for='toggle-all'>Mark all as complete</label>
        <ul class='todo-list'>
          <li v-for='todo in filteredTodos' :key='todo' :class="{ editing: todo === editingTodo, completed: todo.completed }">
             <div class='view'>
                <input class='toggle' type='checkbox' v-model="todo.completed">
                <label @dblclick="editTodo(todo)">{{ todo.text }}</label>
                <button class='destroy' @click="remove(todo)"></button>
             </div>
             <input
              class="edit"
              type="text"
              v-model="todo.text"
              v-editing-focus="todo === editingTodo"
              @keyup.enter="doneEdit(todo)"
              @blur="doneEdit(todo)"
              @keyup.esc="cancelEdit(todo)"
            >
          </li>
        </ul>
     </section>
    <footer class="footer" v-show='count'>
      <span class="todo-count">
        <strong>{{ remainingCount }}</strong> {{ remainingCount > 1 ? 'items' : 'item' }} left
      </span>
      <ul class="filters">
        <li><a href="#/all">All</a></li>
        <li><a href="#/active">Active</a></li>
        <li><a href="#/completed">Completed</a></li>
      </ul>
      <button class="clear-completed" @click="removeCompleted" v-show="count > remainingCount">
        Clear completed
      </button>
    </footer>
  </section>
</template>

<script>
import './assets/index.css'
import useLocalStorage from './utils/useLocalStorage'
import { ref, watchEffect, computed, onMounted,onUnmounted } from 'vue'

const storage = useLocalStorage()

// 1 添加待办事项
const useAdd = todos => {
  const input = ref('')
  const addTodo = () => {
    const text = input.value && input.value.trim()
    if (text.length === 0) return
    todos.value.unshift({
      text,
      completed: false
    })
    input.value = ''
  }
  return {
    input,
    addTodo
  }
}

// 2 删除待办事项
const useRemove = todos => {
  const remove = todo => {
    const index = todos.value.indexOf(todo)
    todos.value.splice(index,1)
  }
  const removeCompleted = () => {
    todos.value = todos.value.filter(todo => !todo.completed)
  }
  return {
    remove,
    removeCompleted
  }
}

// 3 编辑待办项
const useEdit = (remove) => {
  let beforeEditingText = ''
  // editingTodo控制双击文本框状态
  const editingTodo = ref(null)

  const editTodo = todo => {
    beforeEditingText = todo.text
    editingTodo.value = todo
  }
  const doneEdit = todo => {
    if (!editingTodo.value) return
    todo.text = todo.text.trim()
    // todo.text 如果不存在就删除
    todo.text || remove(todo)
    editingTodo.value = null
  }
  const cancelEdit = todo => {
    editingTodo.value = null
    todo.text = beforeEditingText
  }
  return {
    editingTodo,
    editTodo,
    doneEdit,
    cancelEdit
  }
}

// 4 切换待办项完成状态
const useFilter = todos=>{
  const allDone = computed({
    get() {
      return !todos.value.filter(todo => !todo.completed).length
    },
    set(value){
      todos.value.forEach(todo => {
        todo.completed = value
      })
    }
  })
  const filter = {
    all: list => list,
    active: list => list.filter(todo=> !todo.completed),
    completed: list => list.filter(todo=> todo.completed)
  }
  const type = ref('all')

  // 计算属性
  const filteredTodos = computed(()=>filter[type.value](todos.value))

  // 计算属性
  const remainingCount = computed(()=>filter.active(todos.value).length)

  // 计算属性
  const count = computed(()=>todos.value.length)

  const onHashChange = () => {
    const hash = window.location.hash.replace('#/','')
    if (filter[hash]){
       type.value = hash
    } else {
      type.value = 'all'
      window.location.hash = ''
    }
  }
  onMounted(() => {
    window.addEventListener('hashchange', onHashChange)
    onHashChange()
  })

  onUnmounted(() => {
    window.removeEventListener('hashchange', onHashChange)
  })

  return {
    allDone,
    filteredTodos,
    count,
    remainingCount
  }
}

// 5 存储待办事项
const useStorage = () => {
  const KEY = "TODOKEYS"
  const todos = ref(storage.getItem(KEY) || [])
  watchEffect(()=>{
    // 此处作用是在todos有变化的时候都重新赋值localstorage
    storage.setItem(KEY,todos.value)
  })
  return todos
}



export default {
  name: 'App',
  setup(){
   const todos = useStorage()
   const { remove, removeCompleted } = useRemove(todos)
   return {
     todos,
     remove,
     removeCompleted,
     ...useAdd(todos),
     ...useEdit(remove),
     ...useFilter(todos)
   }
  },
  directives:{
    editingFocus: (el,binding)=>{
      binding.value && el.focus()
    }
  }
}
</script>

<style>

</style>
