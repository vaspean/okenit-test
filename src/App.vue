<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <table>
      <tbody>
        <tr>
          <td>
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
    };
  },
  components: {
  },
  created() {
    axios.get('vacation.json').then((input) => {
      this.employees = input.data.data;
    });
  },
};
</script>

<style lang="scss">
</style>
