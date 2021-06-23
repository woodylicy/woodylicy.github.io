<template>
  <div class="container">
    <div
      v-loading.fullscreen.lock="fullScreenLoading"
      element-loading-text="取得資料中"
      element-loading-spinner="el-icon-loading"
      element-loading-background="rgba(0, 0, 0, 0.6)"
    ></div>
    <div>
      <h1>Google Sheet Analysis</h1>
      <div style="text-align: right">
        <el-link type="primary" @click.native="dialogVisible = true"
          >使用說明</el-link
        >
      </div>
      <el-drawer title="使用說明" :visible.sync="dialogVisible" size="75%">
        <div>
          <ul style="text-align: left">
            <li>
              <p>
                需修改原始資料分隔符號，建議保留原始檔案，複製一份新試算表再行操作
              </p>
              <ol>
                <li>
                  <p>打開試算表，點選[編輯]>[尋找與取代]</p>
                </li>
                <li>
                  <p>
                    尋找[,]全部取代為[||]，原始資料欄位或選項不可使用[,]避免錯誤
                  </p>
                  <img src="~/assets/使用說明/05.png" />
                </li>
              </ol>
            </li>
            <li>
              <p>URL: 設定權限如下圖後，複製連結</p>
              <img src="~/assets/使用說明/02.png" />
            </li>
            <li>
              <p>API: 依以下步驟取得</p>
              <ol>
                <li>
                  <p>點選[工具]>[指令碼編輯器]</p>
                  <img src="~/assets/使用說明/03.png" />
                </li>
                <li>
                  <p>複製以下程式碼貼上</p>
                  <p>
                    function doGet(e) {<br />
                    var params = e.parameter;<br />
                    var sheetUrl = params.sheetUrl;<br />
                    var sheetTag = params.sheetTag;<br />
                    var row = params.row;<br />
                    var col = params.col;<br />
                    var endRow = params.endRow;<br />
                    var endCol = params.endCol;<br />
                    var rowRange = endRow - row + 1;<br />
                    var colRange = endCol - col + 1;<br />

                    var SpreadSheet = SpreadsheetApp.openByUrl(sheetUrl);<br />
                    var Sheet = SpreadSheet.getSheetByName(sheetTag);<br />

                    var lastRow = Sheet.getLastRow();<br />
                    var lastCol = Sheet.getLastColumn();<br />

                    if(rowRange>lastRow){<br />
                    rowRange = lastRow;<br />
                    }<br />

                    if(colRange>lastCol){<br />
                    colRange = lastCol;<br />
                    }<br />

                    var data = Sheet.getSheetValues(row, col,
                    rowRange,colRange);<br />

                    return ContentService.createTextOutput(data);<br />
                    }
                  </p>
                </li>
                <li>
                  <p>
                    點選[部署]>[新增部署作業]，點選[部署]按鈕，即可取得網頁應用程式網址（API網址）
                  </p>
                  <img src="~/assets/使用說明/04.png" />
                </li>
              </ol>
            </li>
            <li>
              <p>工作表名稱: 試算表下方分頁名稱</p>
              <img src="~/assets/使用說明/06.png" />
            </li>
          </ul>
        </div>
        <span slot="footer" class="dialog-footer"> </span>
      </el-drawer>
      <div class="wrap">
        <el-form
          label-position="right"
          label-width="100px"
          :model="googleSheet"
        >
          <el-form-item label="URL : ">
            <el-input v-model="googleSheet.url"></el-input>
          </el-form-item>
          <el-form-item label="API : ">
            <el-input v-model="googleSheet.api"></el-input>
          </el-form-item>
          <el-form-item label="工作表名稱 : ">
            <el-input v-model="googleSheet.sheetTag"></el-input>
          </el-form-item>
          <!-- <el-form-item> -->
          <el-button @click.native="getGoogleSheetData()">取得資料</el-button>
          <!-- </el-form-item> -->
        </el-form>
      </div>
      <div class="sheet-info wrap" v-if="googleSheet.colCounts">
        <h4>Google Sheet資訊</h4>
        <div>欄位：{{ googleSheet.colCounts }}</div>
        <div>資料筆數：{{ googleSheet.data.length }}</div>
      </div>
      <div class="filter-data wrap" v-if="googleSheet.data.length">
        <h4>
          篩選資料
          <i class="el-icon-circle-plus-outline" @click="addFilter()"></i>
        </h4>
        <div v-for="(filter, index) in filters" :key="index">
          <i class="el-icon-circle-close" @click="delFilter(index)"></i>
          <el-select v-model="filter.col" placeholder="請選擇">
            <el-option
              v-for="(value, key) in googleSheet.columns"
              :key="key"
              :label="value"
              :value="key"
            >
            </el-option>
          </el-select>
          <el-input
            type="text"
            v-model="filter.text"
            placeholder="輸入篩選條件"
          />
          <el-button @click.native="checkFilterData(filter)">檢視</el-button>
          <el-checkbox v-model="filter.reg">正規表示</el-checkbox>
        </div>
      </div>
      <el-drawer :title="drawer.title" :visible.sync="drawer.isOpen" size="50%">
        <div>
          <span v-for="key in drawerDataKeysAry" :key="key"
            >{{ key }} : {{ drawer.data[key] }}</span
          >
        </div>
      </el-drawer>
      <div class="statistic wrap" v-if="googleSheet.data.length">
        <!-- <el-divider></el-divider> -->
        <div>
          <span>共{{ filterDataAry.length }}筆，</span>
          <el-select v-model="filterData.selectCol" placeholder="請選擇">
            <el-option
              v-for="(value, key) in googleSheet.columns"
              :key="key"
              :label="value"
              :value="key"
            >
            </el-option>
          </el-select>
        </div>
        <div v-if="filterData.selectCol">
          <dl>
            <dt>{{ googleSheet.columns[filterData.selectCol] }}</dt>
            <dd v-for="key in dataStatisticsKeysAry" :key="key">
              {{ key }} : {{ dataStatistics[key] }}
            </dd>
          </dl>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      fullScreenLoading: false,
      dialogVisible: false,
      googleSheet: {
        url: "",
        api: "",
        sheetTag: "",
        columns: {},
        colCounts: 0,
        data: [],
      },
      filters: [
        {
          col: "0",
          text: "",
          reg: false,
        },
      ],
      drawer: {
        isOpen: false,
        title: "",
        data: {},
      },
      filterData: {
        selectCol: "",
      },
    };
  },
  mounted() {
    // this.getGoogleSheetData()
  },
  computed: {
    filterDataAry() {
      return this.filters.reduce((dataAry, filter) => {
        return dataAry.filter((obj) => {
          if (filter.reg) {
            return new RegExp(filter.text.trim()).test(obj[filter.col]);
          } else {
            return obj[filter.col].includes(filter.text);
          }
        });
      }, this.googleSheet.data);
    },
    dataStatistics() {
      return this.filterDataAry.reduce((optObj, dataObj) => {
        dataObj[this.filterData.selectCol].split("||").map((el) => {
          el = el.trim().toLocaleUpperCase();
          if (optObj[el]) {
            optObj[el] += 1;
          } else {
            optObj[el] = 1;
          }
        });
        return optObj;
      }, {});
    },
    dataStatisticsKeysAry() {
      return Object.keys(this.dataStatistics).sort(
        (a, b) => a.length - b.length
      );
    },
    drawerDataKeysAry() {
      return Object.keys(this.drawer.data).sort((a, b) => a.length - b.length);
    },
  },
  methods: {
    checkInput() {
      if (
        this.googleSheet.url.trim()==='' ||
        this.googleSheet.api.trim()==='' ||
        this.googleSheet.sheetTag.trim()===''
      ) {
        return false;
      }
      return true
    },
    getGoogleSheetData() {
      if(!this.checkInput()){this.alertMessage("請填入試算表資訊", "error");return }
      this.fullScreenLoading = true;
      let params = {
        sheetUrl: this.googleSheet.url,
        sheetTag: this.googleSheet.sheetTag,
        row: 1,
        col: 1,
        endRow: 1,
        endCol: 1000,
      };
      this.alertMessage("開始取得欄位數量", "info");
      //axios
      // this.$axios
      //   .get(this.googleSheet.api, { params })
      //   .then((res) => {
      //     this.googleSheet.colCounts = res.data.split(",").length;
      //     this.alertMessage(`共 ${this.googleSheet.colCounts} 欄`, "success");
      //     return Promise.resolve(this.googleSheet.colCounts);
      //   })
      //   .then(() => {
      //     params.endRow = 1000;
      //     this.alertMessage("開始取得所有資料", "info");
      //     this.$axios.get(this.googleSheet.api, { params }).then((res) => {
      //       let rawData = res.data.split(",");
      //       let colCounts = this.googleSheet.colCounts;
      //       let dataCounts = rawData.length / colCounts;
      //       let dataAry = [];
      //       for (let i = 0; i < dataCounts; i++) {
      //         let dataObj = rawData
      //           .slice(i * 73, (i + 1) * 73)
      //           .reduce((obj, el, index) => {
      //             return { ...obj, [index]: el };
      //           }, {});
      //         dataAry.push(dataObj);
      //       }
      //       this.googleSheet.columns = dataAry[0];
      //       this.googleSheet.data = dataAry.slice(1);
      //       this.alertMessage(
      //         `共 ${this.googleSheet.data.length} 筆`,
      //         "success"
      //       );
      //       this.fullScreenLoading = false;
      //     });
      //   })
      //   .catch(() => {
      //     this.alertMessage("取得資料失敗", "error");
      //     this.fullScreenLoading = false;
      //   });
      //jquery
      $.get(this.googleSheet.api, params, (data) => {
        this.googleSheet.colCounts = data.split(",").length;
      })
        .done(() => {
          this.alertMessage(`共 ${this.googleSheet.colCounts} 欄`, "success");
          params.endRow = 1000;
          $.get(this.googleSheet.api, params, (data) => {
            let rawData = data.split(",");
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
            this.alertMessage(
              `共 ${this.googleSheet.data.length} 筆`,
              "success"
            );
            this.fullScreenLoading = false;
          }).fail(() => {
            this.alertMessage("取得資料失敗，請確認填入的資料是否正確", "error");
            this.fullScreenLoading = false;
          });
        })
        .fail(() => {
          this.alertMessage("取得資料失敗，請確認填入的資料是否正確", "error");
          this.fullScreenLoading = false;
        });
    },
    checkFilterData(filter) {
      let data = this.googleSheet.data.filter((obj) => {
        if (filter.reg) {
          return new RegExp(filter.text.trim().toLocaleUpperCase()).test(
            obj[filter.col]
          );
        } else {
          return obj[filter.col].includes(
            filter.text.trim().toLocaleUpperCase()
          );
        }
      });

      let dataCount = data.reduce((optObj, dataObj) => {
        dataObj[filter.col].split("||").map((el) => {
          el = el.trim().toLocaleUpperCase();
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
    addFilter() {
      this.filters.push({
        col: "0",
        text: "",
        reg: false,
      });
    },
    delFilter(index) {
      this.filters.splice(index, 1);
    },
    alertMessage(message, type) {
      this.$message({
        message,
        type,
      });
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
    margin: 0 0 5px;
    background-color: #ccc;
  }
}
.filter-data {
  margin-top: 20px;
  h4 {
    text-align: left;
  }
  .el-input {
    width: auto;
  }
  + div {
    display: flex;
  }
  i {
    cursor: pointer;
    font-size: 18px;
    margin-right: 3px;
  }
  > div {
    display: flex;
    justify-content: center;
    align-items: center;
  }
}
.sheet-info,
.statistic {
  text-align: left;
  margin-top: 20px;
}

.wrap {
  background-color: rgb(215, 236, 219);
  border-radius: 8px;
  box-shadow: 5px 5px 5px rgba(0, 0, 0, 0.7);
  padding: 10px;
}

.el-dialog__wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
}

.el-dialog {
  width: auto;
}
</style>
