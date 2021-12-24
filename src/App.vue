<template>
  <div id="app">
    <!-- <img alt="Vue logo" src="./assets/logo.png"> -->
    <table border="1" cellspacing="0" cellpadding="1">
      <tbody>
        <tr>
          <th></th>
          <th v-for="(month, index) in months" :key="index">{{ month.name }}</th>
          <!-- <td>
          </td> -->
        </tr>
        <tr>
          <td></td>
          <td v-for="(month, index) in months" :key="index" class="date"> </td>
        </tr>
        <tr v-for="(employee, indexEmployee) in employees" :key="employee.id">
          <td>{{employee.person.name}} {{indexEmployee}}</td>
          <td v-for="(month, indexMonth) in months" :key="indexMonth">
            <ul class="weeks">
              <li v-for="week in weeksInMonth(`2021`, indexMonth)" :key="week" class="week" :style="{'width':`${100/weeksInMonth(`2021`, indexMonth)}%`}" :class="{'week_vacation': vacationWeek(week, month, indexMonth, employee.vacations)}">
                {{week}}
                <!-- 1 -->
              </li>
            </ul>
            <!-- {{ indexEmployee }} -->

            <!-- {{  }} -->
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'App',
  data() {
    return {
      employees: {},
      monthNames: [
        'Январь', 'Февраль', 'Март', 'Апрель', 'Май', 'Июнь', 'Июль', 'Август', 'Сентябрь', 'Октябрь', 'Ноябрь', 'Декабрь',
      ],
      currentYear: '2021',
    };
  },
  components: {
  },
  computed: {
    months() {
      const months = [];
      let lastValue = 0;
      for (let i = 0; i < 12; i += 1) {
        months.push({
          name: this.monthNames[i],
          startWeek: lastValue,
          endWeek: this.weeksInMonth(this.currentYear, i) + lastValue - 1,
        });
        if (this.getWeekDay(new Date(`${this.currentYear}-${i + 1}-${this.getLastDayOfMonth(this.currentYear, i)}`)) === 'ВС') {
          lastValue += this.weeksInMonth(this.currentYear, i);
        } else {
          lastValue += this.weeksInMonth(this.currentYear, i) - 1;
        }
      }
      // console.log(this.weekByDate('2021-2-1'));
      return months;
    },
  },
  created() {
    axios.get('vacation.json').then((input) => {
      this.employees = input.data.data;
    });
    // console.log(this.weekByDate('2021-12-27'));
    // const d = new Date();
    // console.log()
  },
  methods: {
    timeToDays(time) {
      return time / 1000 / 60 / 60 / 24;
    },
    weekByDate(date) {
      const dateObj = new Date(date);
      dateObj.setHours(0, 0, 0);
      dateObj.setDate(dateObj.getDate() + 4 - (dateObj.getDay() || 7));
      const yearStart = new Date(dateObj.getFullYear(), 0, 1);
      const weekNumber = Math.floor((this.timeToDays(dateObj - yearStart) + 1) / 7);
      return weekNumber;
    },
    weeksInMonth(year, month) {
      const date = new Date(year, month + 1, 0);
      return Math.ceil((date.getDate() - (date.getDay() ? date.getDay() : 7)) / 7) + 1;
    },
    getLastDayOfMonth(year, month) {
      const date = new Date(year, month + 1, 0);
      return date.getDate();
    },
    getWeekDay(date) {
      const days = ['ВС', 'ПН', 'ВТ', 'СР', 'ЧТ', 'ПТ', 'СБ'];
      // console.log(date);
      return days[date.getDay()];
    },
    vacationWeek(weekNumber, month, indexMonth, vacations) {
      let vacationWeek = false;
      // console.log(month.startWeek + week - 1);
      vacations.forEach((vacation) => {
        let vacationStartWeek;
        if ((new Date(vacation.startDate).getMonth() === 0) && this.weekByDate(vacation.startDate) === 52) {
          vacationStartWeek = 0;
        } else {
          vacationStartWeek = this.weekByDate(vacation.startDate);
        }
        const vacationEndWeek = this.weekByDate(vacation.endDate);

        for (let week = vacationStartWeek; week < vacationEndWeek; week += 1) {
          if (week === (month.startWeek - 1 + weekNumber)) {
            vacationWeek = true;
          }
        }
      });
      return vacationWeek;
    },
  },
};
</script>

<style lang="scss">
* {
  box-sizing: border-box;
}

ul {
  margin: 0;
  padding: 0;
  list-style: none;
}

th {
  width: 100px;
}

td {
  padding: 0;
}

table {
  // border: 1px solid black;
  // cellspacing
}

.date {
  height: 100px;
}

.weeks {
  display: flex;
  justify-content: space-around;
  // width: 100%;
  // flex-direction: row;
}

.week {
  height: 30px;
  text-align: center;
}

.week_vacation {
  background: rgb(0, 166, 255);
}
</style>
