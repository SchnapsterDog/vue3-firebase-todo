<template>
  <div class="todo-wrapper">
    <div class="title has-text-centered">ToDo</div>
    <form @submit.prevent="addToDo">
      <div class="field is-grouped mb-5">
        <p class="control is-expanded">
          <input
            v-model="newTodoContent"
            class="input"
            type="text"
            placeholder="Add a todo"
          />
        </p>
        <p class="control">
          <button :disabled="!newTodoContent" class="button is-info">
            Add
          </button>
        </p>
      </div>
    </form>
    <div
      v-for="todo in todos"
      :key="todo.id"
      class="card mb-5"
      :class="{ 'has-background-success-light': todo.done }"
    >
      <div class="card-content">
        <div class="content">
          <div class="columns is-vcentered">
            <div
              class="column"
              :class="{ 'has-text-success line-through': todo.done }"
            >
              {{ todo.content }}
            </div>
            <div class="column is-5 has-text-right">
              <button
                class="button"
                :class="todo.done ? 'is-success' : 'is-light'"
                @click="toggleDone(todo.id)"
              >
                &check;
              </button>
              <button
                class="button is-danger ml-2"
                @click="deleteTodo(todo.id)"
              >
                &cross;
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script setup>
import { ref, onMounted } from "vue";
import {
  addDoc,
  collection,
  deleteDoc,
  doc,
  getDocs,
  onSnapshot,
  orderBy,
  query,
  updateDoc,
} from "firebase/firestore";
import { db } from "@/firebase";

const todosCollectionRef = collection(db, "todos");
const todosCollectionQuery = query(todosCollectionRef, orderBy("date", "desc"));

const todos = ref([]);

onMounted(() => {
  onSnapshot(todosCollectionQuery, (querySnapshot) => {
    let fbTodo = [];
    querySnapshot.forEach((doc) => {
      const todo = {
        id: doc.id,
        content: doc.data().content,
        done: doc.data().done,
      };
      fbTodo.push(todo);
      todos.value = fbTodo;
    });
  });
});

const newTodoContent = ref("");

const addToDo = () => {
  addDoc(todosCollectionRef, {
    content: newTodoContent.value,
    done: false,
    date: Date.now(),
  });
  newTodoContent.value = "";
};

const deleteTodo = (id) => {
  deleteDoc(doc(todosCollectionRef, id));
};

const toggleDone = (id) => {
  const index = todos.value.findIndex((todo) => todo.id === id);
  updateDoc(doc(todosCollectionRef, id), {
    done: !todos.value[index].done,
  });
};
</script>
<style>
@import "node_modules\bulma\css\bulma.min.css";

.todo-wrapper {
  max-width: 400px;
  padding: 20px;
  margin: 0 auto;
}

.line-through {
  text-decoration: line-through;
}
</style>