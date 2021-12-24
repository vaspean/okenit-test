<template>
  <div id="app">
    <table border="1" cellspacing="0" cellpadding="1">
      <tbody>
        <tr>
          <th></th>
          <th v-for="(month, index) in months" :key="index" class="month__name" @click="openMonth(index)">{{ month.name }}</th>
        </tr>
        <tr>
          <td></td>
          <td v-for="(month, index) in months" :key="index" class="date">
            <ul class="weeks">
              <li v-for="week in weeksInMonth(`2021`, index)" :key="week" :style="{'width':`${100/weeksInMonth(`2021`, index)}%`}" :class="{'openedWeek': openedMonth === index}">
                <span class="week__dates" v-show="openedMonth === index">{{weekRangeDateString(month.startWeek - 1 + week)}}</span>
              </li>
            </ul>
          </td>
        </tr>
        <tr>
          <td></td>
          <td v-for="(month, index) in months" :key="index">
            <ul class="weeks">
              <li v-for="week in weeksInMonth(`2021`, index)" :key="week" :style="{'width':`${100/weeksInMonth(`2021`, index)}%`}" :class="{'openedWeek': openedMonth === index}">
                <span v-show="openedMonth === index">{{month.startWeek - 1 + week}}</span>
              </li>
            </ul>
          </td>
        </tr>
        <tr v-for="employee in employees" :key="employee.id">
          <td>{{employee.person.name}}</td>
          <td v-for="(month, indexMonth) in months" :key="indexMonth">
            <ul class="weeks" :class="{'openedMonth': openedMonth === indexMonth}">
              <li v-for="week in weeksInMonth(`2021`, indexMonth)" :key="week" class="week" :style="{'width':`${100/weeksInMonth(`2021`, indexMonth)}%`}" :class="{'week_vacation': vacationWeek(week, month, employee.vacations).isVacation, 'openedWeek': openedMonth === indexMonth}">
              <span v-show="openedMonth === indexMonth">{{vacationRangeDateString(employee.id, vacationWeek(week, month, employee.vacations).vacationId)}}</span>
              </li>
            </ul>
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
      openedMonth: null,
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
      return months;
    },
  },
  created() {
    axios.get('vacation.json').then((input) => {
      this.employees = input.data.data;
    });
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
      return days[date.getDay()];
    },
    vacationWeek(weekNumber, month, vacations) {
      const vacationWeek = {
        isVacation: false,
        vacationId: undefined,
      };
      vacations.forEach((vacation) => {
        let vacationStartWeek;
        if ((new Date(vacation.startDate).getMonth() === 0) && this.weekByDate(vacation.startDate) === 52) {
          vacationStartWeek = 0;
        } else {
          vacationStartWeek = this.weekByDate(vacation.startDate);
        }
        const vacationEndWeek = this.weekByDate(vacation.endDate);

        for (let week = vacationStartWeek; week <= vacationEndWeek; week += 1) {
          if (week === (month.startWeek - 1 + weekNumber)) {
            vacationWeek.isVacation = true;
            vacationWeek.vacationId = vacation.id;
          }
        }
      });
      return vacationWeek;
    },
    openMonth(index) {
      if (this.openedMonth === index) {
        this.openedMonth = null;
      } else {
        this.openedMonth = index;
      }
    },
    vacationRangeDateString(employeeId, vacationId) {
      let range = '';
      this.employees[employeeId - 1].vacations.forEach((vacation) => {
        if (vacation.id === vacationId) {
          range = `${new Date(vacation.startDate).getDate()}-${new Date(vacation.endDate).getDate()}`;
        }
      });
      return range;
    },
    weekRangeDateString(weekOfYear) {
      let dateFrom = new Date();
      let dateTo = new Date('1970-01-01');
      if (weekOfYear === 0) {
        for (let day = 31; day > 25; day -= 1) {
          if (this.weekByDate(`${this.currentYear - 1}-${12}-${day}`) === 52) {
            const newDateFrom = new Date(`${this.currentYear - 1}-${12}-${day}`);
            if (dateFrom > newDateFrom) {
              dateFrom = newDateFrom;
            }
          }
        }
        for (let day = 1; day <= 8; day += 1) {
          if (this.weekByDate(`${this.currentYear}-${1}-${day}`) === 52) {
            const newDateTo = new Date(`${this.currentYear}-${1}-${day}`);
            if (dateTo < newDateTo) {
              dateTo = newDateTo;
            }
          }
        }
      } else {
        for (let month = 0; month < 12; month += 1) {
          for (let j = 1; j <= this.getLastDayOfMonth(this.currentYear, month); j += 1) {
            if (this.weekByDate(`${this.currentYear}-${month + 1}-${j}`) === weekOfYear) {
              const newDateFrom = new Date(`${this.currentYear}-${month + 1}-${j}`);
              const newDateTo = new Date(`${this.currentYear}-${month + 1}-${j}`);
              if (dateFrom > newDateFrom) {
                dateFrom = newDateFrom;
              }
              if (dateTo < newDateTo) {
                dateTo = newDateTo;
              }
            }
          }
        }
      }
      return `${this.formatDate(dateFrom)}.${this.formatMonth(dateFrom)}-${this.formatDate(dateTo)}.${this.formatMonth(dateTo)}`;
    },
    formatDate(date) {
      return (`0${((date).getDate())}`).slice(-2);
    },
    formatMonth(month) {
      return (`0${((month).getMonth()) + 1}`).slice(-2);
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

.month__name {
  cursor: pointer;
}

.openedMonth {
  width: 250px;
}

.openedWeek {
  position: relative;
  text-align: center;
  border-left: 1px solid black;
  border-right: 1px solid black;
}

.date {
  height: 100px;
}

.weeks {
  height: 100%;
  display: flex;
  justify-content: space-around;
}

.week {
  height: 25px;
}

.week_vacation {
  color: white;
  background: rgb(0, 166, 255);
}

.week__dates {
  width: 100px;
  position: absolute;
  transform: rotate(-90deg);
  transform-origin: left top 0;
  left: 15px;
  position: absolute;
  bottom: -20px;
  font-weight: 800;
}
</style>
