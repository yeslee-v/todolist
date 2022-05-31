<!-- html -->
<template>
  <div id="app">
    <div class="container">
      <div class="col-md-6 offset-md-3">
        <h1 class="text-center" :style="{ margin: '100px 0px 50px' }">To-Do List</h1>
        <!-- 값은 v-model을 통해 입력받고 그 값이 task으로 동기화된다 -->
        <div class="allToDoBox">
          <div class="setCondition">
            <input
              type="text"
              placeholder="Enter after write"
              class="newCategory"
              v-show="isEdit"
              v-model="catInput"
              @keyup.enter="addNewCategory"
            />
            <select
              class="setCategory"
              v-show="!isEdit"
              v-model="selectCat"
              @change="onChange($event)"
            >
              <option value="default">---------------</option>
              <!-- option의 value에서 v-model의 값(selectCat)을 고른다 -->
              <option value="default" type="text">Add Category</option>
              <option v-for="(category, index) in allCatList" :key="index">
                {{ category.catName }}
              </option>
            </select>

            <input
              type="text"
              class="form-control"
              placeholder="Enter task"
              v-model="task"
              @keyup.enter="submitTodo"
            />
            <button class="btn-writing" @click="submitTodo">submit</button>
          </div>

          <!-- table 태그 사용하는 이유: 딱맞게 나누려고 -->
          <table class="table table-bordered">
            <thead>
              <tr>
                <th scope="col">Category</th>
                <th scope="col">Task</th>
                <th scope="col">Status</th>
                <th scope="col">Created Date</th>
                <th scope="col">Replies</th>
                <th scope="col">Edit</th>
                <th scope="col">Delete</th>
              </tr>
            </thead>
              <tbody v-for="(task, index) in taskList" :key="index">
              <tr>
                <!-- category 추가 -->
                <td>{{ task.cat }}</td>
                <td>
                  <span :class="{ 'done': task.status === 'Done' }">
                   {{ task.todo }}
                  </span>
                </td>
                <td style="width: 120px">
                  <span class="pointer" @click="changeStatus(index)" :class="{'text-danger': task.status === 'To Do', 'text-warning': task.status === 'In Progress', 'text-success': task.status === 'Done'}">
                    {{ firstCharUpper(task.status) }}
                  </span>
                </td>
                <td>{{ customTime(task.dateTime) }}</td>
                  <td>
                    <div class="text-center" @click="isSub = index" >
                      <span class="pointer fa fa-plus"></span>
                    </div>
                  </td>  
                <td>
                  <div class="text-center" @click="editTodo(index)">
                    <span class="pointer fa fa-pen"></span>
                  </div>
                </td>
                <td>
                  <div class="text-center" @click="deleteTodo(index)">
                    <span class="pointer fa fa-trash"></span>
                  </div>
                </td>
              </tr>
            <tr v-show="isSub == index">
            <td colspan="4">
              <input type="text" placeholder="Enter sub task" class="text-center" v-model="subTask" @keyup.enter="addSubTodo(index)">
            </td>
            <td colspan="3">
              <button class="text-center" @click="addSubTodo(index)">submit</button>
            </td>
            </tr>
            <!-- 아예 존재하지 않거나 혹은 존재해도 빈 값인 경우 제외 -->
            <tr v-show="task.subTaskList && task.subTaskList.length" v-for="(subTask, subIndex) of task.subTaskList" :key="subIndex">
              <!-- category 추가 -->
                <td>{{ subTask.cat }}</td>
                <td>
                  <span :class="{ 'done': subTask.status === 'Done' }">
                   {{ subTask.todo }}
                  </span>
                </td>
                <td style="width: 120px">
                  <span class="pointer" @click="changeStatus(index, subIndex)" :class="{'text-danger': subTask.status === 'To Do', 'text-warning': subTask.status === 'In Progress', 'text-success': subTask.status === 'Done'}">
                    {{ firstCharUpper(subTask.status) }}
                  </span>
                </td>
                <td>{{ customTime(subTask.dateTime) }}</td>
                  <td>
                  </td>  
                <td>
                </td>
                <td>
                  <div class="text-center" @click="deleteTodo(index, subIndex)">
                    <span class="pointer fa fa-trash"></span>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<!-- javascript -->
<script>

// 해당 모듈에는 객체가 하나만(default) 존재한다
export default {
  name: "app",
  data() {
    return {
      isEdit: false,
      isSub: null,
      task: "",
      subTask: "",
      editedTask: null,
      catInput: "",
      selectCat: "default",
      catList: [],
      taskList: [],
      subTaskList: [],
      statusList: ["To Do", "In Progress", "Done"],
    };
  },
  // computed라는 json 객체, 반환값(activeTodoList) 자체는 setter지만, computed안의 activeTodoList는 getter로 동작
  computed: {
    allCatList() {
      return this.catList;
    },
  },
  mounted() {
    // localStorage에 저장되어있던 값 불러와서 기존의 taskList, catList에 객체에 저장하는 작업을 해줘야
    this.taskList = JSON.parse(localStorage.getItem("savedTodolist") || "[]");
    this.catList = JSON.parse(localStorage.getItem("savedCategory") || "[]");
  },
  components: {
},
  methods: {
    // 변경됨 감지하는 함수
    onChange(e) {
      if (e.target.value === "default") this.isEdit = true;
    },
    // 카테고리 추가하는 함수: 왼쪽 바에서 카테고리 리스트 보여줌
    addNewCategory() {
      if (!this.catInput.trim())
        alert("ERROR: Input right message!");
      else {
        this.catList.push({
          catName: this.catInput,
        });
        this.isEdit = false; // 카테고리 리스트에 넣고 나서 다시 false로 바꿔야 select태그가 보인다
        this.selectCat = this.catInput;
        this.catInput = "";
      }
    },
    // 시간 custom 출력하는 함수
    customTime(today) {
      // today는 string이므로 object로 바꿔준다(day)
      const day = new Date(today);
      const date =
        day.getFullYear() + "-" + (day.getMonth() + 1) + "-" + day.getDate();
      const time =
        day.getHours() + ":" + day.getMinutes() + ":" + day.getSeconds();
      return date + " " + time;
    },
    // 할 일 추가하는 함수
    submitTodo() {
      console.log("submitTodo")
      if (!this.task.trim()) {
        alert("ERROR: Input right message!");
      } else {
        var today = new Date();
        if (this.editedTask === null) {
          this.taskList.push({
            cat: this.selectCat,
            todo: this.task,
            dateTime: today,
            status: this.statusList[0],
          });
        } else {
          this.taskList[this.editedTask].todo = this.task;
          this.editedTask = null;
        }
        localStorage.setItem("savedTodolist", JSON.stringify(this.taskList));
      }
      this.task = "";
    },
    addSubTodo(index) {
      // subTaskList가 없을 때만 새로 만들어 준다
      if (!this.taskList[index].subTaskList)
        this.taskList[index].subTaskList = [];

      this.isSub = true;
      // 카테고리는 동일
      // 클릭하면 인풋 받음
      // 값 엔터하면 push를 하기
      console.log("idx>>>>>>>>>>>>", index, this.selectCat);
      console.log("addSubTodo");
      if (!this.subTask.trim()) {
        alert("ERROR: Input right message!");
      } else {
        var today = new Date();
        if (this.editedTask === null) {
          console.log("push sub tasks");

          // object 넣을 때는 {}
          this.taskList[index].subTaskList.push({
            cat: "↪ " + this.selectCat,
            todo: this.subTask,
            dateTime: today,
            status: this.statusList[0],
          });
        } else {
          console.log("push this task");
          this.taskList.subTaskList[this.editedTask].todo = this.subTask;
          this.editedTask = null;
        }
        console.log("taskList>>>>", this.taskList);

        localStorage.setItem("savedTodolist", JSON.stringify(this.taskList));
      };
      this.subTask = "";
      this.isSub = null;
    },
    editTodo(index) {
      this.task = this.taskList[index].todo;
      this.editedTask = index;
    },
    deleteTodo(index, subIndex = -1) {
      if (subIndex === -1 ) {
        this.taskList.splice(index, 1);
        localStorage.setItem("savedTodolist", JSON.stringify(this.taskList));
      } else {
        this.taskList[index].subTaskList.splice(subIndex, 1);
        localStorage.setItem("savedTodolist", JSON.stringify(this.taskList));
      }
    },
    editSubTodo(index, subIndex) {
      this.task = this.taskList[index].subTaskList[subIndex].todo;
      this.editedTask = subIndex;
    },
    changeStatus(index, subIndex = -1) {
      if (subIndex === -1) {
        let newIndex = this.statusList.indexOf(this.taskList[index].status);
        if (++newIndex > 2) newIndex = 0;
        this.taskList[index].status = this.statusList[newIndex];
        localStorage.setItem("savedTodolist", JSON.stringify(this.taskList));
      } else {
        let newIndex = this.statusList.indexOf(this.taskList[index].subTaskList[subIndex].status);
        if (++newIndex > 2) newIndex = 0;
        this.taskList[index].subTaskList[subIndex].status = this.statusList[newIndex];
        localStorage.setItem("savedTodolist", JSON.stringify(this.taskList));
      }
    },
    firstCharUpper(str) {
      return str.charAt(0).toUpperCase() + str.slice(1);
    }
  },
};
</script>

<!-- css -->
<style scoped>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
.router-wrapper {
  min-height: 60vh;
}
.setCondition {
  display: flex;
}
.setCategory {
  width: 7em;
}
.newCategory {
  border: solid;
  border-color: blue;
}
.list-group-item {
  display: inline-block;
}
.ToDoProperty {
  font-size: 0.8em;
}
.pointer {
  cursor: pointer;
}
.done {
  text-decoration: line-through;
}
</style>
