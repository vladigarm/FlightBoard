<template>
   <div id='app'>
      <br>
      <form class = 'direction'>
         <div v-on:click = 'change_direcrion()' class = 'menu'> {{direction_rus}} </div>

      </form>
      <br><br><br><br>
      <form class = search>
         <input v-on:keyup = 'searchText(input)' v-model = 'input' placeholder='Search flight'>
         <div class='menu' @mouseover='listOne = true' @mouseleave='listOne = false'>
            {{filterStatus}}
            <transition name='fade'>
               <ul v-if='listOne' @click='listOne = false'>
                  <li v-on:click = 'filter( allFlights )'>Вcе рейсы</li>
                  <li v-on:click = 'filter( registration )'>Регистрация</li>
                  <li v-on:click = 'filter( inflight )'>В полёте</li>
                  <li v-on:click = 'filter( arrived )'>Прибыл</li>
                  <li v-on:click = 'filter( detained )'>Задержан</li>
                  <li v-on:click = 'filter( canceled )'>Отменен</li>
               </ul>
            </transition>
         </div>
      </form>
      <br>
      <form class = results v-for = 'flight in filteredFlights'>
         <ul class = list> 
            <li >
               <div class = 'box' > 
                  <div class = departure> {{ checkTime(flight.t_st, flight.t_et) }} </div>
                  <div v-if = "direction == 'departure'" class = to> {{flight.mar2.city}} </div>
                  <div v-if = "direction == 'arrived'" class = to> {{flight.mar2.city}} </div>
                  <div class = flightNum> {{flight.co.code}} {{flight.flt}} </div>
                  <div class = gate> {{flight.term_gate}} </div>
                  <div class = status> {{flight.detained ? 'Задержан. ' : ''}} {{ checkStatus(flight.vip_status)}} </div>
               </div>
            </li>
         </ul>
      </form>
   </div>
</template>

<script>
    import axios from 'axios'

    export default {
        data() {
                return {
                    direction_rus: 'Вылет',
                    direction: 'departure',
                    flights: null,
                    filteredFlights: null,
                    url: null,
                    date: '2018-12-09T20:00:00',
                    dateStart: null,
                    dateEnd: null,
                    input: null,
                    listOne: false,
                    filterStatus: 'Все рейсы',
                    allFlights: 'Все рейсы',
                    registration: 'Регистрация',
                    inflight: 'В полете',
                    arrived: 'Прибыл',
                    flyIn: 'Совершил',
                    detained: 'Задержан',
                    canceled: 'Отменен',
                    tempTimeStatus: '',
                }
            },
            created() {
                this.getDate();
                this.generateURL();
                this.giveInfo();
            },
            methods: {
                //Получение текущей даты
                getDate() {
                    let hour = +new Date().toISOString().slice(11, 13) +3; 
                    this.dateStart = new Date().toISOString().slice(0, 11) + hour +':00:00';
                    this.dateEnd = this.dateStart.slice(0, 11) + '23:00:00';
                },
                //Создает URL запроса к данным с аэропорта Шереметьево
                generateURL() {
                    this.url = 'https://www.svo.aero/bitrix/timetable/?direction=' + this.direction + '&dateStart=' + this.dateStart + '%2B03:00&dateEnd=' + this.dateEnd + '%2B03:00&perPage=1000&page=0&locale=en'
                    console.log(this.url);
                },
                //Получает JSON файл с данными о полёте
                giveInfo() {
                    axios.get(this.url).then(response => {
                        this.flights = this.filteredFlights = response.data.items.map( (item) => {
                          item.detained = Date.parse(item.t_et) > Date.parse(item.t_st)
                          return item
                        })
                    }).then(this.filteredFlights = this.flights).catch((err) => console.log('Query error'));
                },
                //Переводит время в читаемый вид
                checkTime( planing, fact ) {
                    if (planing == null) return ''
                    if (fact != null) return fact.slice(11, 16)
                    else return planing.slice(11, 16);
                    return
                },
                //Фильтр рейсов по городу и номеру рейса
                filter( status ) {
                    this.filterStatus = status;
                    this.filteredFlights = this.flights;
                    if (status == 'Все рейсы') return;
                    this.filteredFlights = [];
                    this.flights.forEach((item, i, arr) => {
                        if (status == 'Задержан' && item.detained) {
                          this.filteredFlights.push(item)
                        } 
                        else if (item.vip_status.includes(status)) {
                            this.filteredFlights.push(item)
                        }
                    })
                },
                //Выбор направления полётов
                change_direcrion() {
                    if (this.direction == 'departure') {
                        this.direction = 'arrived';
                        this.direction_rus = 'Прилёт';
                    } else {
                        this.direction = 'departure';
                        this.direction_rus = 'Вылет';
                    }
                    this.filterStatus = 'Все рейсы';
                    this.generateURL();
                    this.giveInfo();
                },
                //Проверка статуса рейса (с опозданием или без)
                checkStatus( status ) {
                    if (this.tempTimeStatus.includes('Задержан')) {
                        status = this.tempTimeStatus + status
                        this.tempTimeStatus = ''
                        return status
                    }
                    this.tempTimeStatus = ''
                    return status
                },
                //Поиск текста для фильтра рейсов
                searchText( input ){
                    if (input == '') return this.filteredFlights = this.flights;
                    this.filteredFlights = this.flights;
                    this.filteredFlights = [];
                    return this.flights.forEach((item, i, arr) => {
                      if (this.direction == 'arrived') {
                      if (item.mar1.city.toLowerCase().includes(input.toLowerCase())  || 
                          item.co.code.toLowerCase().includes(input.toLowerCase())    ||
                          item.flt.toLowerCase().includes(input.toLowerCase()))
                         { 
                          this.filteredFlights.push(item)
                      }} else 
                      if (item.mar2.city.toLowerCase().includes(input.toLowerCase())  || 
                          item.co.code.toLowerCase().includes(input.toLowerCase())    ||
                          item.flt.toLowerCase().includes(input.toLowerCase()))
                          {
                          this.filteredFlights.push(item)
                      }
                    })
                }

            }
    
              
}
</script>

<style>

.list
{
  border-radius: 15px;
  box-shadow: 1px -1px 2px 0 rgba(0,0,0,.1),
 -1px -1px 2px 0 rgba(0,0,0,.1),
 2px 2px 5px 0 rgba(0,0,0,.1),
 -2px 2px 5px 0 rgba(0,0,0,.1);
  font: 22px Arial;
  font-weight: 700;
  letter-spacing: 2px;
  list-style-type: none;
  margin: 7px;
  min-height: 90px;
}
.list li
{
  padding-top: 30px;
}
.direction
{
  display: block;
  margin: 0 auto;
  max-width: 1000px;
  text-align: left;
}
.direction div
{
  font: 20px/1.5 'Open Sans', sans-serif;
  margin-left: 0.5%;
  margin-left: 10px;
}
.checkbox
{
  display: block;
  margin: 0 auto;
  max-width: 1000px;
  text-align: left;  
}
.search
{
  display: block;
  margin: 0 auto;
  max-width: 1000px;
  text-align: left;
}
.search input
{
  height: 35px;
  margin-left: 1%;
  padding: 0 auto;
  width: 78%;
}
.search button
{
  height: 35px;
  width: 2%;
}
.action
{
  color: red;
}
.results
{
  margin: 0 auto;
  max-width: 1000px;
  text-align: left;
}
.box div
{
  display: inline-block;
  word-wrap: break-word;
}
.departure
{
  align: center;
  margin-left: 1%;
  margin-right: 1%;
  padding-right: 1%;
  width: 8%;
}
.to
{
  align: center;
  width: 40%;
}
.flightNum
{
  width: 15%;
}
.gate
{
  width: 3%;
}
.status
{
  display: inline-block;
  font: 16px Arial;
  text-align: center;
  width: 20%;
}
.menu
{
  background: #222;
  color: #fff;
  cursor: pointer;
  display: inline-block;
  float: right;
  font: 14px/1.5 'Open Sans', sans-serif;
  font-weight: 600;
  list-style: none;
  margin: 0 auto;
  max-width: 180px;
  min-width: 100px;
  padding: 10px;
  position: absolute;
  text-align: center;
  width: 20%;
}
.menu li
{
  background: #222;
  display: block;
  float: left;
  min-width: 100px;
  padding: 20px;
  position: relative;
  text-decoration: none;
}
.menu li ul
{
  left: 0;
  margin: 0;
  padding: 0;
  position: absolute;
  top: 40px;
}
.menu li ul li
{
  background: #333;
  transition: background .2s;
}
.menu li ul li:hover
{
  background: #444;
}
.fade-enter-active, .fade-leave-active
{
  transition: opacity .2s;
}
.fade-enter, .fade-leave-active
{
  opacity: 0;
}

</style>
