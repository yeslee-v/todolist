<template>
    <div id="app">
        <div class="container">
            <div class="col-md-6 offset-md-3"> 
                <h1 class="text-center mb-4">할 일 목록</h1>
                <!-- 값은 v-model을 통해 입력받고 그 값이 todoInput으로 동기화된다 -->
                <div class="allToDoBox">
                    <div class="setCondition">
                        <input type="text" class="newCategory" v-show="isEdit" v-model="catInput" @keyup.enter="addNewCategory">
                        <select class="setCategory" v-show="!isEdit" v-model="selectCat" @change="onChange($event)">
                            <option value="baseTodo">---------------</option> <!-- option의 value에서 v-model의 값(selectCat)을 고른다 -->
                            <option value="default" type="text">enter 눌러 추가</option>
                            <option v-for="(category, index) in AllCatList" :key="index">
                                {{ category.catName }}
                            </option>
                        </select>
                        <input type="text" class="form-control" v-model="todoInput" @keyup.enter="addNewTodo">
                    </div>

                    <div class="text-right mb-16">
                        <button type="button" class="btn btn-sm m-4" @click="changeCurrentState('active')">할 일</button>
                        <button type="button" class="btn btn-sm" @click="changeCurrentState('done')">한 일</button>
                        <button type="button" class="btn btn-sm m-4" @click="changeCurrentState('all')">전체: {{ todoList.length }}</button>
                    </div>

                    <div class="list-group mb-4">
                        <div class="list-group-item text-left m-2" v-for="(todo, index) in activeTodoList" :key="index">
                            {{ todo.text }}
                            <div class="ToDoProperty">
                                {{ todo.dateTime }}
                                <button class="DoneToDo" @click="toggleTodoState(todo)" v-if="currentState === 'active'">완료</button>
                                <button class="deleteToDo" @click="toggleDoneState(index)" v-if="currentState === 'done'">삭제</button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
// 해당 모듈에는 객체가 하나만(default) 존재한다
export default {
    name: 'app',
    data() {
        return {
            isEdit: false,
            todoInput: '',
            catInput: '',
            selectCat: 'baseTodo',
            catList: [],
            todoList: [],
            currentState: 'active'
        }
    },
    // computed라는 json 객체, 반환값(activeTodoList) 자체는 setter지만, computed안의 activeTodoList는 getter로 동작
    computed: {
        AllCatList() {
            console.log("computed: AllCatList");
            return this.catList;
        },
        activeTodoList() {
            console.log("computed: activeTodoList");
            // return JSON.parse(localStorage.getItem('할 일 리스트'));
            return this.todoList.filter(todo => this.currentState === 'all' || todo.state === this.currentState);
        },
},
    methods: {
        onChange(e) {
            if (e.target.value === "default")
                this.isEdit = true;
        },
        changeCurrentState(state) {
            this.currentState = state;
        },
        addNewCategory() {
            this.catList.push({
                catName: this.catInput
            })
            // 카테고리 리스트에 넣고 나서 다시 false로 바꿔야 select태그가 보인다
            this.isEdit = false;
            this.selectCat = this.catInput;
            this.catInput = '';
        },
        addNewTodo() {
            if (!this.todoInput.trim())
                alert("ERROR: Input right message!");
            else {
                var today = new Date();
                var date = today.getFullYear() + '-' + (today.getMonth()+1) + '-' + today.getDate();
                var time = today.getHours() + ":" + today.getMinutes() + ":" + today.getSeconds();

                this.todoList.push({
                    text: this.todoInput,
                    dateTime: date + ' ' + time,
                    state: 'active',
                });
                localStorage.setItem('할 일 리스트', JSON.stringify(this.todoList));
            }
            this.todoInput = '';
        },
        toggleTodoState(todo) {
            todo.state = (todo.state === 'active') ? 'done' : 'active';
            localStorage.setItem('할 일 리스트', JSON.stringify(this.todoList));
        },
        toggleDoneState(index) {
            this.todoList.splice(index, 1);
            localStorage.setItem('할 일 리스트', JSON.stringify(this.todoList));
        },
    },
    mounted() {
        console.log("mounted");
        // const exists = JSON.parse(localStorage.getItem('할 일 리스트'));
        // this.activeTodoList = exists;
               // if (exists) {
        //     var pv = localStorage.getItem(this.selectCat);
        //     var jsonList = JSON.parse(pv);
        //     console.log("pv:", jsonList, typeof(jsonList));
        //     document.getElementsByClassName("list-group-item").innerHTML = pv;
        // // callPrevTodoList();
        // }
    },
    // 기능 분리
    components: {
    }
}
</script>

<style scoped>
.setCondition {
    width: 33em;
    padding: 0.5em;
    display: flex;
}
.setCategory {
    width: 7em;
}
.newCategory {
    border: solid;
    border-color: blue;
}
.allToDoBox {
    padding: 1em;
    background-color: ivory;
    border-radius: 40px;
    box-shadow: inset -6px -6px 15px rgba(145, 160, 180, 0.45),
        30px 30px 40px rgba(118, 146, 180, 0.18);
    overflow: hidden;
    background-repeat: no-repeat;
    background-position: center center;
    background-size: cover;
    display: flex;
    flex-direction: column;
    align-items: center;
}
.list-group-item {
    display: inline-block;
}
.ToDoProperty {
    font-size: 0.8em;
}
.btn {
    border: none;
    background-color: beige;
    padding: 0.5em 1em 0.5em 1em;
    border-radius: 40px;
    box-shadow: inset -6px -6px 15px rgba(145, 160, 180, 0.45),
        30px 30px 40px rgba(118, 146, 180, 0.18);
    overflow: hidden;
    background-repeat: no-repeat;
    background-position: center center;
    background-size: cover;
    flex-direction: column;
    align-items: center;
}
</style>