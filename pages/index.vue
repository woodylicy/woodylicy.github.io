<template>
  <div class="container">
    <div>
      <h1>google sheet</h1>
      <div>
        <el-form label-position="left" label-width="80px" :model="googleSheet">
          <el-form-item label="URL">
            <el-input v-model="googleSheet.url"></el-input>
          </el-form-item>
          <el-form-item label="API">
            <el-input v-model="googleSheet.api"></el-input>
          </el-form-item>
        </el-form>
      </div>
      <input type="button" @click="test1()" value="取得資料" />
      <div>
        <div>欄位：{{ googleSheet.colCounts }}</div>
        <div>資料筆數：{{ googleSheet.data.length }}</div>
      </div>
      <div v-for="(filter, index) in filters" :key="index">
        <el-select v-model="filter.col" placeholder="请選擇">
          <el-option
            v-for="(value, key) in googleSheet.columns"
            :key="key"
            :label="value"
            :value="key"
          >
          </el-option>
        </el-select>
        <input type="text" v-model="filter.text" />
      </div>
      <!-- <el-select v-model="num" placeholder="请选择">
        <el-option
          v-for="value in googleSheet.data.length"
          :key="value - 1"
          :label="value"
          :value="value - 1"
        >
        </el-option>
      </el-select> -->
      <!-- <p v-if="googleSheet.data.length">{{ googleSheet.data[num] }}</p> -->
      <p v-if="googleSheet.data.length">共{{ ttt }}筆</p>
      <!-- <p v-if="googleSheet.data.length">{{ googleSheet.data.filter(obj=>{return obj[value].includes(filter)})}}</p> -->
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      googleSheet: {
        
        url: "",
        api: "",
        columns: {},
        colCounts: 0,
        data: [],
      },
      value: "0",
      num: "0",
      filter: "",
      filters: [
        {
          col: "2",
          text: "女",
        },
        {
          col: "4",
          text: "化學",
        },
      ],
    };
  },
  computed: {
    ttt() {
      // return this.googleSheet.data.filter((obj) => {
      //   return obj[this.value].includes(this.filter);
      // }).length;

      return this.filters.reduce((dataAry, filter) => {
        return dataAry.filter((obj) => {
          return obj[filter.col].includes(filter.text);
        });
      }, this.googleSheet.data).length;
    },
  },
  methods: {
    test1() {
      let params = {
        sheetUrl: this.googleSheet.url,
        sheetTag: "工作表1",
        row: 1,
        col: 1,
        endRow: 1,
        endCol: 1000,
      };
      this.$axios
        .get(this.googleSheet.api, { params: params })
        .then((res) => {
          this.googleSheet.colCounts = res.data.split(",").length;
          return Promise.resolve(this.colCounts);
        })
        .then(() => {
          params.endRow = 1000;
          this.$axios
            .get(this.googleSheet.api, { params: params })
            .then((res) => {
              let rawData = res.data.split(",");
              let colCounts = this.googleSheet.colCounts;
              let dataCounts = rawData.length / colCounts;
              let dataAry = [];
              for (let i = 0; i < dataCounts; i++) {
                let dataObj = rawData
                  .slice(i * 73, (i + 1) * 73)
                  .reduce((obj, el, index) => {
                    return { ...obj, [index]: el };
                  }, {});
                dataAry.push(dataObj);
              }
              this.googleSheet.columns = dataAry[0];
              this.googleSheet.data = dataAry.slice(1);
              // console.log(this.googleSheet);
            });
        });
    },
  },
};
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}
</style>
