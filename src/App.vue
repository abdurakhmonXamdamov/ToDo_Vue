<template>
  <div>
    <div class="todo">
        <div class="todo__header">
            <div class="todo__title">
                To<span>Dolist</span>
            </div>
            <div class="todo__box">
               <div class="todo__btns">
                    <button @click="filterTasks('all')">Barchasi</button>
                    <button @click="filterTasks('completed')">Bajarilganlar</button>
                    <button @click="filterTasks('uncompleted')">Bajarilmaganlar</button>
               </div>
            </div>
        </div>

        <div class="todo__cards">
            <div class="card" v-for="tasks of filteredTasks">
                <div id="card__header">
                    <span :style="{
                        textDecoration: tasks.status? 'line-through' : 'none',
                    }">{{ tasks.title }}</span>
                    <div class="card__edit">
                        <span title="edit" :style="{
                            color: true? 'var(--navyBlue)': 'var(--lightOrange)'
                        }" @click="edit( tasks.id )"><i class="bi bi-pencil-square"></i></span>
                        <span title="complete" id="completed__card" @click="isComplete( tasks.id )"><i :style="{
                            color: tasks.status? 'white' : 'var(--darkGrey)',
                        }" class="bi bi-check-circle"></i></span>
                        <span title="remove" @click="removeTask( tasks.id )" id="remove__card"><i class="bi bi-trash"></i></span>
                    </div>
                </div>
                <div id="card__text" :style="{
                        textDecoration: tasks.status? 'line-through' : 'none',
                    }">
                  {{ tasks.text }}
                </div>
                <div id="card__footer">
                    <span>{{ tasks.date }}</span>
                    <span>{{ tasks.member }}</span>
                </div>
            </div>
        </div>

        <div id="todo__modal" :class="{active: addActive}">
            <span id="remove" @:click.self="unTogge()">&times;</span>
            <h3>Yangi topshiriq</h3>
            <form class="todo__form" @submit.prevent="addToDo">
                    <input type="text" v-model="taskObj.title" class="form-control" id="task__title" placeholder="Topshiriq sarlavhasi" required>
                    <input type="text" v-model="taskObj.text" class="form-control" name="task" id="task" placeholder="Topshiriq matni" required>
                    <input type="date" v-model="taskObj.date" class="form-control" id="date" placeholder="Ohirgi muddat" required>
                <select name="member" v-model="taskObj.member" id="group__member" class="form-control" required>
                    <option :disabled="true" value=''>Guruh a'zosi</option>
                    <option v-for="(members, ind) of groupMembers" :key="ind" :value="members"> {{ members }}</option>
                </select>

                <button v-show="showw" id="modal__btn" type="submit">Qo’shish</button>
                <button v-show="!showw" id="modal__btn" type="button" @click="save()">Saqlash</button>
            </form>
        </div>

        <div id="add__task" @click="toggle()">
            <i class="bi bi-plus-lg"></i>
        </div>

        <div id="overlay" @:click.self="unTogge()" :class="{active: addActive}"></div>
    </div> 
</div>
</template>

<script>

import axios from 'axios';

export default {
    data() {
      return {
        addActive: false,
        taskObj:{
            status:false,
        },
        tasksArr: [],
        filteredTasks: [],
        filterType: 'all',
        groupMembers: ['Abdurahmon', 'Alamat', 'Alisher', "Og'abek", 'Maftuna',  'Vasi', 'Bobur', 'Muhammad Ali', 'Madina'],
        url: 'http://localhost:3000/ToDo',
        showw: true,
      }
    },
    computed:{
        arrangeArr(){
            return this.tasksArr.filter(complete => complete.status)
        }
    },
    methods:{
        toggle(){
            this.addActive = !this.addActive
        },

        unTogge(){
            this.addActive = false
            this.taskObj = {}
        },

        addToDo(){
            axios.post(this.url, this.taskObj)
            .then((res) => {
                this.tasksArr = [res.data, ...this.tasksArr]
                this.filteredTasks = [...this.tasksArr]
                this.taskObj = {}
                this.addActive = false
                console.log(res);
            })
            .catch(err => console.error(err))
        },

        removeTask(id){
            axios.delete(`${this.url}/${id}`)
            .then(() => {
                this.tasksArr = this.tasksArr.filter((task) => task.id!== id)
                this.filteredTasks = [...this.tasksArr]
            })
            .catch(err => console.error(err))
        },

        isComplete(idx){
            let changeStatus = this.tasksArr.find(task => task.id === idx)
            if(changeStatus){
                changeStatus.status = !changeStatus.status  
                axios.put(`${this.url}/${changeStatus.id}`, changeStatus)
                    .then()
                    .catch(err => console.error(err))
            }
        },

        edit(id){
            axios.get(`${this.url}/${id}`)
            .then((res) => {
                this.taskObj = {...res.data}
                this.taskObj.id = id
                this.toggle()
                this.showw = false
            })
        },

        save(){
            axios.put(`${this.url}/${this.taskObj.id}`, this.taskObj)
            .then(response => {
                this.tasksArr = this.tasksArr.map(editTask => editTask.id === this.taskObj.id? response.data : editTask);
                this.filteredTasks = [...this.tasksArr]
                this.taskObj = {};
                this.unTogge()
                this.showw = true;
            })
            .catch(error => {console.log(error.message)})
        },

        filterTasks(type){
            // this.filterType = type

            if(type === 'completed'){
                this.filteredTasks = this.tasksArr.filter(t => t.status)
            }else if(type === 'uncompleted'){
                this.filteredTasks = this.tasksArr.filter(t => !t.status)
            }else{
                this.filteredTasks = [...this.tasksArr]
            }
        }
    },
    mounted(){
        axios.get(this.url)
           .then((res) => {
                this.tasksArr = res.data
                this.filteredTasks = [...this.tasksArr]
            })
           .catch(err => console.error(err))
    }
}
</script>

<style>
</style>