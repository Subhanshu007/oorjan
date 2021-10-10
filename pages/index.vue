<template>
  <div class="table" v-if="!$fetchState.pending">
    <table>
      <tr class="table__header">
        <th class="table__header-empty">
          <span></span>
        </th>
        <th
          class="table__header-cell"
          v-for="(item, index) in cpyArr[count]"
          :key="`date-${index}`"
        >
          <span>{{ $moment(item.date).calendar() }}</span>
        </th>
      </tr>
      <tr>
        <td>
          <span>open</span>
        </td>
        <td v-for="(item, index) in cpyArr[count]" :key="`open-${index}`">
          <span :class="checkColorOpen(item, index , count)">{{ item.open }}</span>
        </td>
      </tr>
      <tr>
        <td>
          <span >close</span>
        </td>
        <td v-for="(item, index) in cpyArr[count]" :key="`close-${index}`">
          <span :class="[{'close__color--green' : item.close > item.open}, {'close__color--red' : item.close < item.open}]">{{ item.close }}</span>
        </td>
      </tr>
    </table>
    <div class="button__wrapper">
      <button
        class="next cta"
        @click="count < cpyArr.length - 1 ? count++ : (count = cpyArr.length - 1)"
      >
        Next
      </button>
      <button class="prev cta" @click="count > 0 ? count-- : (count = 0)">
        Prev
      </button>
    </div>
  </div>
</template>
<script>
export default {
  async fetch() {
    const { store, error } = this.$nuxt.context;
    try {
      const res = await this.$axios.$get(
        "https://f68370a9-1a80-4b78-b83c-8cb61539ecd6.mock.pstmn.io/api/v1/get_market_data/"
      );
      this.pageData = res.data;
      /* creation of a 2d array */
      let arrayOfArrays = [];
      /* assigning a length equal to the division of chunks that we will get ceil will add an extra length incase if there is a remainder when we divide our actual array */
      let chunkedArray = new Array(
        Math.ceil(this.pageData.length / this.limit)
      );
      /* calculating the remainder in order to check how many addional data needs to be added  */
      let remainder = this.pageData.length % this.limit;
      /* floor value is taken in order to know the exact value of the divided chunks */
      let division = Math.floor(this.pageData.length / this.limit);
      /* this index will help to add the elments inside the second layer of our array*/
      let index = 0;
      /* i is kept less than division in order to create the array size*/
      for (let i = 0; i < division; i++) {
        let index = 0;
      /* this will allocate the space inside the second layer of our array*/
        chunkedArray[i] = new Array(this.limit);
        /* this for loop will add the elements inside the array of our array*/
        for (let j = i * this.limit; j < i * this.limit + this.limit; j++) {
          chunkedArray[i][index] = this.pageData[j];
          /* index needs to be incremented in order to add from 0 to the limit and the loop runs on limit times here the limit is 7 so it will run seven times */
          index ++;
        }
      }
      this.cpyArr = chunkedArray;
      /* adding the remaining element if any to the last index of our 2d array */
      if (remainder > 0) {
        for (let i = this.cpyArr.length; i < this.pageData.length; i++) {
          this.cpyArr[division + 1][i] = this.pageData[i];
        }
      }

    /*color logic */


    } catch (err) {
      err;
    }
  },
  data() {
    return {
      pageData: undefined,
      newPageData: undefined,
      count: 0,
      limit: 7,
      cpyArr: [],
      increment: undefined,
    };
  },
  computed: {
    compareFromPreviousInput(obj, innerlevelIndex, outerLevelIndex ){
      console.log(obj, innerlevelIndex, outerLevelIndex )
    }
  },
  methods: {
    checkColorClose(item, index , count){
      if(item.close > item.open){
        return 'close__color--green';
      }
      else if(item.close < item.open){
        return 'close__color--red';

      }
        else {
        return 'close__color--none';

      }
    },
    checkColorOpen(item, index , count){
      console.log(
        item
      )
      if(count == 0 && index == 0){
        return 'open__color--none';
      }
      if(count > 0 && index == 0){
       let indexNum = count * 7;
        if(item.open > this.pageData[indexNum - 1].close){
          return 'open__color--green';
        }
        else {
           return 'open__color--red';
        }
      }
      else {
       let indexNum = count * 7 + index;
        if(item.open > this.pageData[indexNum - 1].close ){
          return 'open__color--green';

        }
           else {
           return 'open__color--red';
        }
      }
    }
  },
  mounted() {
    // console.log(this.sevenDataArr)
  },
};
</script>
<style lang="css">
  table {
  font-family: arial, sans-serif;
  border-collapse: collapse;
  width: 100%;
}

td, th {
  border: 1px solid #dddddd;
  text-align: left;
  padding: 8px;
}

tr:nth-child(even) {
  background-color: #dddddd;
}
.button__wrapper{
  text-align: center;
  padding: 15px;
}
.cta {
  border: none;
  color: white;
  padding: 10px 25px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  margin: 4px 2px;
  cursor: pointer;
  border-radius: 10px;
}
.next{
  background-color: #4CAF50; /* Green */

}
.prev{
 background-color: #f44336;
}
.open__color--green, .close__color--green{
  color: #4CAF50;
}
.open__color--red, .close__color--red{
  color: #f44336;
}
</style>