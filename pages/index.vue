<template>
  <div class="container">
    <div>
      <h1>Google Sheet Analysis</h1>
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
      <input type="button" @click="getGoogleSheetData()" value="取得資料" />
      <div v-if="googleSheet.colCounts">
        <div>欄位：{{ googleSheet.colCounts }}</div>
        <div>資料筆數：{{ googleSheet.data.length }}</div>
      </div>
      <div v-if="googleSheet.colCounts">
        <div
          v-for="(filter, index) in filters"
          :key="index"
          style="display: flex"
        >
          <el-select v-model="filter.col" placeholder="请選擇">
            <el-option
              v-for="(value, key) in googleSheet.columns"
              :key="key"
              :label="value"
              :value="key"
            >
            </el-option>
          </el-select>
          <el-input type="text" v-model="filter.text" />
          <el-button @click.native="checkFilterData(filter)">確認資料</el-button>
          <el-checkbox v-model="filter.reg">正規表示</el-checkbox>
        </div>
      </div>
      <el-drawer :title="drawer.title" :visible.sync="drawer.isOpen">
        <div>
          <span v-for="key in drawerDataKeysAry" :key="key">{{ key }} : {{ drawer.data[key] }}</span>
        </div>
      </el-drawer>
      <p v-if="googleSheet.data.length">共{{ statistics }}筆</p>
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
          reg: false,
        },
        {
          col: "4",
          text: "化學",
          reg: true,
        },
      ],
      drawer: {
        isOpen: false,
        title: "",
        data: {},
      },
    };
  },
  computed: {
    statistics() {
      return this.filters.reduce((dataAry, filter) => {
        return dataAry.filter((obj) => {
          if (filter.reg) {
            return new RegExp(filter.text.trim()).test(obj[filter.col]);
          } else {
            return obj[filter.col].includes(filter.text);
          }
        });
      }, this.googleSheet.data).length;
    },
    drawerDataKeysAry(){
      return Object.keys(this.drawer.data).sort((a,b)=>a.length-b.length)
    }
  },
  methods: {
    getGoogleSheetData() {
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
            });
        });
    },
    checkFilterData(filter) {

      let data = this.googleSheet.data.filter(obj=>{
        if (filter.reg) {
            return new RegExp(filter.text.trim()).test(obj[filter.col]);
          } else {
            return obj[filter.col].includes(filter.text);
          }
      })

      let dataCount = data.reduce((optObj, dataObj) => {
        dataObj[filter.col].split("||").map((el) => {
          el = el.trim();
          if (optObj[el]) {
            optObj[el] += 1;
          } else {
            optObj[el] = 1;
          }
        });
        return optObj;
      }, {});

      this.drawer.isOpen = true;
      this.drawer.title = this.googleSheet.columns[filter.col];
      this.drawer.data = dataCount;
    },
  },
};
</script>

<style lang="scss">
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}
.el-drawer__body {
  span {
    display: block;
    margin: 0px 5px 4px 0px;
    padding: 2px;
    background-color: #ccc;
  }
}
</style>
