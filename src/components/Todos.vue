<template>
  <div class="container">
    <div class="col-sm-10">
      <h1>Задачи</h1>
      <counter
        :finished="finishedTasks"
        :unfinished="unfnishedTasks"></counter>
      <confirmation
        :message="confirmationMessage"
        :showMsg="showConfirmation"
        v-if="showConfirmation">
      </confirmation>
      <button type="button"
        id="task-add"
        class="btn btn-success btn-sm align-left d-block"
        v-b-modal.todo-modal>
        Добавить задачу
      </button>

      <table class="table table-dark table-stripped table-hover">
        <thead class="thead-light">
          <tr>
            <th>Uid</th>
            <th>Описание</th>
            <th>Выполнена?</th>
            <th></th>
          </tr>
        </thead>

        <tbody>
          <tr v-for="(todo, index) in todos" :key="index">
            <td class="todo-uid">{{ todo.uid }}</td>
            <td>{{ todo.description }}</td>
            <td class="todo-check">
              <span v-if="todo.is_completed">✅</span>
              <span v-else>❌</span>
            </td>
            <td>
              <div class="btn-group" role="group">
                <button type="button"
                    class="btn btn-secondary btn-sm"
                    v-b-modal.todo-modal
                    @click="updateTodo(todo)">
                  Обновить
                </button>
                &nbsp;
                <button type="button"
                    class="btn btn-danger btn-sm"
                    @click="deleteTodo(todo)">
                  X
                </button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>

      <b-modal ref="todoModal"
               id="todo-modal"
               v-bind:title="todoForm.title"
               hide-footer
               @close="resetForm">
        <b-form @submit="onSubmit" @reset="onReset" class="w-100">
        <b-form-group id="form-description-group"
                      label="Описание:"
                      label-for="form-description-input">
          <b-form-input id="form-description-input"
                        type="text"
                        v-model="todoForm.description"
                        required>
          </b-form-input>
        </b-form-group>
        <b-form-checkbox value="true" v-model="todoForm.is_completed">
          Задача выполнена?</b-form-checkbox>
        <b-button-group id="confirm-buttons">
          <b-button type="submit" variant="primary">Ок</b-button>
          <b-button type="reset" variant="danger">Отмена</b-button>
        </b-button-group>
        </b-form>
      </b-modal>

    </div>
  </div>
</template>

<style>
  button#task-add {
    margin-top: 20px;
    margin-bottom: 20px;
  }
  div#confirm-buttons {
    margin-top: 10px;
  }
  h1, td {
    text-align: left;
  }
  .todo-uid, .todo-check {
    text-align: center;
  }
</style>

<script>
import Confirmation from './Confirmation.vue';
import Counter from './Counter.vue';

export default {
  name: 'Todo',
  data() {
    return {
      todos: [],
      todoForm: {
        uid: 0,
        description: '',
        is_completed: false,
        title: 'Добавить',
      },
      confirmationMessage: '',
      showConfirmation: false,
      finishedTasks: 0,
      unfnishedTasks: 0,
    };
  },
  methods: {
    getTodos() {
      if (!localStorage.getItem('todos')) {
        localStorage.setItem('todos', JSON.stringify([]));
      } else {
        this.todos = JSON.parse(localStorage.getItem('todos'));
        this.finishedTasks = 0;
        this.unfnishedTasks = 0;
        this.todos.forEach((todo) => {
          if (todo.is_completed) this.finishedTasks += 1;
          else this.unfnishedTasks += 1;
        });
      }
    },
    resetForm() {
      this.todoForm.uid = 0;
      this.todoForm.description = '';
      this.todoForm.is_completed = false;
      this.todoForm.title = 'Добавить';
      this.getTodos();
    },
    onSubmit(event) {
      event.preventDefault();
      if (this.todoForm.uid === 0) {
        const requestData = {
          description: this.todoForm.description,
          is_completed: this.todoForm.is_completed,
        };
        this.addTodo(requestData);
      } else {
        const requestData = {
          uid: this.todoForm.uid,
          description: this.todoForm.description,
          is_completed: this.todoForm.is_completed,
        };
        this.editTodo(requestData);
      }
      this.$refs.todoModal.hide();
      this.resetForm();
    },
    onReset(event) {
      event.preventDefault();
      this.$refs.todoModal.hide();
      this.resetForm();
    },
    addTodo(data) {
      const currentTodos = JSON.parse(localStorage.getItem('todos'));
      const newUid = (currentTodos.length > 0) ? currentTodos[currentTodos.length - 1].uid + 1 : 1;
      currentTodos.push({
        uid: newUid,
        description: data.description,
        is_completed: data.is_completed,
      });
      localStorage.setItem('todos', JSON.stringify(currentTodos));
      this.confirmationMessage = `Задача "${data.description}" успешно добавлена!`;
      this.showConfirmation = true;
      this.getTodos();
    },
    editTodo(data) {
      const currentTodos = JSON.parse(localStorage.getItem('todos'));
      const modTodos = [];
      currentTodos.forEach((currentTodo) => {
        if (currentTodo.uid === data.uid) {
          modTodos.push({
            uid: data.uid,
            description: data.description,
            is_completed: data.is_completed,
          });
        } else {
          modTodos.push(currentTodo);
        }
      });
      localStorage.setItem('todos', JSON.stringify(modTodos));
      this.confirmationMessage = `Задача "${data.description}" успешно отредактирована!`;
      this.showConfirmation = true;
      this.getTodos();
    },
    updateTodo(todo) {
      this.todoForm = todo;
      this.todoForm.title = 'Редактировать';
    },
    deleteTodo(todo) {
      const currentTodos = JSON.parse(localStorage.getItem('todos'));
      const modTodos = [];
      currentTodos.forEach((currentTodo) => {
        if (currentTodo.uid !== todo.uid) {
          modTodos.push(currentTodo);
        }
      });
      localStorage.setItem('todos', JSON.stringify(modTodos));
      this.confirmationMessage = `Задача "${todo.description}" успешно удалена!`;
      this.showConfirmation = true;
      this.getTodos();
    },
  },
  components: {
    confirmation: Confirmation,
    counter: Counter,
  },
  created() {
    this.getTodos();
  },
};
</script>
