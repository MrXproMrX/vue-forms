<template>
  <div class="container">
    <app-alert  :alert="alert" @close="alert = null"></app-alert>
    <form class="card" @submit.prevent="createPerson">
      <h2>Работа с базой данных</h2>

      <div class="form-control">
        <label for="name">Введите имя</label>
        <input type="text" id="name" v-model.trim="name">
      </div>

      <button class="btn primary" :disabled="name.length === 0">Создать человека</button>
    </form>

    <app-loader v-if="loader"></app-loader>

    <app-people-list v-else :people="people" @load="LoadPeople" @remove="removePerson"></app-people-list>
  </div>
</template>

<script>
import AppPeopleList from '@/AppPeopleList.vue';
import axios from 'axios';
import AppAlert from '@/AppAlert.vue';
import AppLoader from '@/AppLoader.vue';
  export default {
    data(){
      return{ 
        name:'',
        people:[],
        alert:null,
        loader: false
      }
    },

    mounted(){
      this.LoadPeople()
    },

    components:{
      AppPeopleList, 
      AppAlert,
      AppLoader,
    },

    // Запрос api

    methods:{
      async createPerson() {
      const response = await fetch('https://vue-with-ht-default-rtdb.firebaseio.com/people.json', {
        method: 'POST',

        headers: {
          'Content-Type': 'application/json'
        },

        body: JSON.stringify({
          firstName: this.name
        })
      })

      const firebaseData = await response.json()

      this.people.push({
        firstName: this.name,
        id: firebaseData.name
      })

      this.name = ''
    },

    
    // Список ползволтел api


      async LoadPeople(){
        try{
          this.loader = true
          const {data} = await axios.get('https://vue-with-ht-default-rtdb.firebaseio.com/people.json')
          if(!data){
            throw new Error('Список людей пуст')
          }
          
          this.people  = Object.keys(data).map(key=>{
          return{
            id: key,
           ...data[key]
          }
          })

        this.loader = false
        }catch(e){
          this.alert = {
            type:'danger',
            title:'Ошибка!',
            text: e.message
          } 
          this.loader = false
          console.log(e.message);
        }
      },

      // Удалит ползволтел

      async removePerson(id){
        try{
          const name = this.people.find(person => person.id === id).firstName
          await axios.delete(`https://vue-with-ht-default-rtdb.firebaseio.com/people/${id}.json`)
          this.people = this.people.filter(people => people.id !== id)
          this.alert = {
            type:'primary',
            title: 'Успешно!',
            text: `Пользвател с именем "${name}" был удалеон`
          }
        }catch(e){

        }
      }
    }
  }
</script>

<style>

</style>
