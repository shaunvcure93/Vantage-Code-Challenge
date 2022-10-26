<template>
  <div class="vander-container">
    <div class="bg"></div>
    <div class="vcover">
      <div class="tabcontent">
        <div class="summary">
          1. Beware of there's a amount limit for API fetching each minute, if you refresh too many times within a short time, it will stop you from fetching data.
          <br />
          <br />
          2. If you wanna build a list of data from API, it should include page and pagesize for pagination, to avoid heavy data fetching and it can lower the time of fetching data. Yup, There's many way to do it, as the sample below, i create the pagination from frontend part, but i suggest it hanlde by backend.
          <br />
          <br />
          3. It's responsive, you can simply resize it, I replace the load button from the pagination on mobile view
        </div>

        <div class="table">
          <div class="thead">
            <div class="title">Exchanges Rate {{ selectedData.length > 0 ? `(${selectedData.length})` : '' }}</div>
          </div>
          <div class="tnote" v-if="error && tableData && tableData.length > 0">
            <div><i class="iconfont">error_outline</i>There's connection error, below exchanges rates weren't the latest updated data, please check the connection and refresh the page to get the latest rates.</div>
          </div>
          <div class="tlist">
            <div class="loader" v-if="loader && error == ''">
              <i class="iconfont">hourglass_empty</i>
            </div>
            <div class="empty" v-if="error  && tableData.length == 0">
              <div><i class="iconfont">feedback</i><span>{{ error }}</span></div>
            </div>
            <div class="scroll" v-if="error == '' || tableData.length > 0">
              <div class="row head">
                <div class="col checkbox" :class="{'active' : selectAll}" @click="selectAllRates()"><i></i></div>
                <div class="col" :class="col.titleCode" v-for="(col, cindex) in tableContent" :key="cindex">
                  <div :class="{'sort' : col.sort}" @click="sort(cindex)">{{ col.title }}<i class="iconfont" v-if="col.sort">{{ col.status ? 'arrow_downward' : 'arrow_upward' }}</i></div>
                </div>
              </div>
              <div class="row" v-for="(rate, rindex) in tableData" :key="rindex" @click="selectRow(rindex)" :title="rate.name">
                <div class="col checkbox" :class="{'active' : rate.status}"><i></i></div>
                <div class="col" :class="[col.titleCode, {'error' : error && tableData && tableData.length > 0}]" v-for="(col, cindex) in tableContent" :key="cindex">{{ rate[col.titleCode] }}</div>
              </div>
            </div>
          </div>
          <pagination :total="totalPage" :current="currentPage" :disabled="!ratesData || ratesData.length == 0 || loader" @selectPage="selectPage" @prev="prevPage" @next="nextPage"></pagination>
        </div>

        <div class="more">
          <div v-if="!isEnd || error == ''" :class="{'disabled' : loader}" @click="loadMore()">Load More</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import pagination from './components/pagination.vue'
export default {
    data(){
        return {
            'loader'      : false,
            'page'        : 5,
            'pageSize'    : 10,
            'currentPage' : 1,
            'totalPage'   : 1,
            'ratesData'   : [],
            'tableData'   : [],
            'selectAll'   : false,
            'selectedData': [],
            'tableContent': [
              {
                'title'    : 'Name',
                'titleCode': 'name',
                'sort'     : true,
                'status'   : false
              },
              {
                'title'    : 'Unit',
                'titleCode': 'unit',
                'sort'     : false,
                'status'   : false
              },
              {
                'title'    : 'Type',
                'titleCode': 'type',
                'sort'     : false,
                'status'   : false
              },
              {
                'title'    : 'Rate',
                'titleCode': 'value',
                'sort'     : true,
                'status'   : false
              }
            ],
            'isEnd'       : false,
            'error'       : ''
        };
    },
    components: {
      pagination
    },
    created () {
        
    },
    mounted() {
        this.getData();
        window.addEventListener('resize', () =>
        {
          let screenWidth = window.innerWidth || document.documentElement.clientWidth || document.body.clientWidth;
          if(screenWidth <= 624)
          {
            this.selectPage(10);
          }
          else
          {
            this.selectPage(5);
          }
        });
    },
    destroyed()
    {
      window.removeEventListener('resize');
    },
    methods:{
      getData()
      {
        this.loader = true;
        this.error = '';
        this.ratesData = [];

        axios
        .get('https://api.coingecko.com/api/v3/exchange_rates')
        .then((res) => 
        {
          if(res.data && res.data.rates)
          {
            let dataArray = Object.entries(res.data.rates).map(([key, obj]) => {
              let dataObj = JSON.parse(JSON.stringify(obj));
              dataObj.status = false;
              return dataObj
            });

            this.ratesData = JSON.parse(JSON.stringify(dataArray));
            window.localStorage.setItem('rates', JSON.stringify(this.ratesData));
            this.initRateData();
            this.loader = false;

            let swidth = window.innerWidth || document.documentElement.clientWidth || document.body.clientWidth;
            if(swidth <= 624)
            {
              this.selectPage(10)
            }
            else
            {
              this.selectPage(5)
            }
          }
          else
          {
            let d = JSON.parse(window.localStorage.getItem('rates') || '[]');
            if(d && d.length > 0)
            {
              this.ratesData = JSON.parse(JSON.stringify(d));
              let swidth = window.innerWidth || document.documentElement.clientWidth || document.body.clientWidth;
              if(swidth <= 624)
              {
                this.selectPage(10)
              }
              else
              {
                this.selectPage(5)
              }
            }
            this.error = `Seems like there's some issue getting data, please refresh the page and try again.`
          }
        })
        .catch((err) => 
        {
          console.log(err, 'err')
          let d = JSON.parse(window.localStorage.getItem('rates') || '[]');
          if(d && d.length > 0)
          {
            this.ratesData = JSON.parse(JSON.stringify(d));
            let swidth = window.innerWidth || document.documentElement.clientWidth || document.body.clientWidth;
            if(swidth <= 624)
            {
              this.selectPage(10)
            }
            else
            {
              this.selectPage(5)
            }
          }
          this.error = `Seems like there's some issue getting data, please refresh the page and try again.`
        })
      },
      initRateData()
      {
        this.totalPage = Math.ceil(this.ratesData.length / this.page);
        this.loader = true;
        this.tableData = [];
        this.ratesData.forEach((item, index) =>
        {
          if(index >= (this.page * (this.currentPage - 1)) && index <= ((this.page * this.currentPage) - 1))
          {
            this.tableData.push(item);
          }
        })

        this.selectAll = false;
        this.selectedData = [];

        this.tableData.forEach((item) =>
        {
          item.status = false;
        })

        setTimeout(() =>
        {
          this.loader = false;
        }, 1000);
      },
      selectRow(index)
      {
        this.selectedData = [];
        this.tableData[index].status = !this.tableData[index].status;
        this.tableData.forEach((item) =>
        {
          if(item.status)
          {
            this.selectedData.push(item);
          }
        })

        let i = this.tableData.findIndex(x => x.status == false);
        this.selectAll = i == -1;
      },
      selectAllRates()
      {
        this.selectAll = !this.selectAll;
        this.selectedData = [];

        this.tableData.forEach((item) =>
        {
          item.status = this.selectAll;
          if(item.status)
          {
            this.selectedData.push(item);
          }
        })
      },
      selectPage(num)
      {
        this.page = num;
        this.currentPage = 1;
        this.initRateData();
      },
      selectCurrentPage(num)
      {
        this.currentPage = num;
        this.initRateData();
      },
      prevPage()
      {
        if(this.currentPage > 1)
        {
          this.currentPage -= 1;
          this.initRateData();
        }
      },
      nextPage()
      {
        if(this.currentPage < this.totalPage)
        {
          this.currentPage += 1;
          this.initRateData();
        }
      },
      sort(i)
      {
        this.tableContent[i].status = !this.tableContent[i].status;
        let value = this.tableContent[i].titleCode;

        this.tableData.sort((a, b) => 
        {
          if(a[value] < b[value])
          {
            return this.tableContent[i].status ? 1 : -1;
          }
          if(a[value] > b[value])
          {
            return this.tableContent[i].status ? -1 : 1;
          }
          return 0;
        });
      },
      loadMore()
      {
        this.currentPage += 1;
        this.totalPage = Math.ceil(this.ratesData.length / this.page);
        this.loader = true;
        this.isEnd = false;

        setTimeout(() =>
        {
          this.ratesData.forEach((item, index) =>
          {
            if(index >= (this.page * (this.currentPage - 1)) && index <= ((this.page * this.currentPage) - 1))
            {
              this.tableData.push(item);
            }
          })
          this.loader = false;
          if(this.currentPage == this.totalPage)
          {
            this.isEnd = true;
          }
        }, 1500);
      }
    }
}
</script>

<style scoped lang="scss">

.bg
{
  display   : block;
  position  : absolute;
  height    : 400px;
  width     : 100%;
  top       : 0;
  left      : 0;
  background: rgb(106,105,255);
  background: -moz-linear-gradient(48deg, rgba(106,105,255,1) 0%, rgba(226,74,127,1) 100%);
  background: -webkit-linear-gradient(48deg, rgba(106,105,255,1) 0%, rgba(226,74,127,1) 100%);
  background: linear-gradient(48deg, rgba(106,105,255,1) 0%, rgba(226,74,127,1) 100%);
  
}

.vander-container
{
  display           : block;
  position          : relative;
  width             : 100%;
  padding           : 20px;
  box-sizing        : border-box;
  -webkit-box-sizing: border-box;
  z-index           : 1;

  & > .vcover
  {
    display  : block;
    position : relative;
    max-width: 1200px;
    width    : 100%;
    margin   : 0 auto;

    & > .tabcontent
    {
      display : block;
      position: relative;
      width   : 100%;

      & > .summary
      {
        display              : block;
        position             : relative;
        width                : 100%;
        box-sizing           : border-box;
        -webkit-box-sizing   : border-box;
        padding              : 20px;
        background           : rgba(255,255,255,0.1);
        border-radius        : 8px;
        -webkit-border-radius: 8px;
        margin               : 0px 0px 16px 0px;
        font-size            : 14px;
        line-height          : 24px;
        color                : #ffffff;
      }
      & > .table
      {
        display              : block;
        position             : relative;
        width                : 100%;
        box-sizing           : border-box;
        -webkit-box-sizing   : border-box;
        overflow             : hidden;
        background           : #ffffff;
        border-radius        : 8px;
        -webkit-border-radius: 8px;
        box-shadow           : -1px 13px 20px -5px rgba(0,0,0,0.2);
        -webkit-box-shadow   : -1px 13px 20px -5px rgba(0,0,0,0.2);

        & > .thead
        {
          display           : block;
          position          : relative;
          width             : 100%;
          box-sizing        : border-box;
          -webkit-box-sizing: border-box;
          padding           : 24px 20px;
          font-size         : 18px;
          font-weight       : bold;
          border-bottom     : solid 1px rgba(0,0,0,0.1);

          & > .title
          {
            display : inline-block;
            position: relative;
          }
          & > .right
          {
            display : block;
            position: absolute;
            height  : 34px;
            top     : calc(50% - 17px);
            right   : 20px;

            & > .selected
            {
              display    : block;
              position   : relative;
              float      : left;
              height     : 34px;
              line-height: 34px;
              font-size  : 12px;
              font-weight: normal;
              margin     : 0px 10px 0px 0px;
            }
            & > .action
            {
              display : block;
              position: relative;
              float   : left;
              height  : 34px;

              & > div
              {
                display              : block;
                position             : relative;
                box-sizing           : border-box;
                -webkit-box-sizing   : border-box;
                width                : 34px;
                height               : 34px;
                border-radius        : 4px;
                -webkit-border-radius: 4px;
                font-weight          : normal;
                background           : rgba(0,0,0,0.05);
                color                : #000000;
                cursor               : pointer;

                & > i
                {
                  display    : block;
                  position   : absolute;
                  width      : 24px;
                  height     : 24px;
                  top        : calc(50% - 12px);
                  left       : calc(50% - 12px);
                  line-height: 24px;
                  text-align : center;
                  font-size  : 24px;
                }
              }
            }
          }
          & > .selected
          {
            display    : block;
            position   : relative;
            float      : right;
            color      : #333333;
            font-size  : 12px;
            font-weight: normal;
          }
        }
        & > .tnote
        {
          display           : block;
          position          : relative;
          width             : 100%;
          box-sizing        : border-box;
          -webkit-box-sizing: border-box;
          padding           : 20px;

          & > div
          {
            display              : block;
            position             : relative;
            width                : 100%;
            box-sizing           : border-box;
            -webkit-box-sizing   : border-box;
            padding              : 10px 10px 10px 36px;
            background           : #dd5454;
            color                : #ffffff;
            border-radius        : 8px;
            -webkit-border-radius: 8px;
            font-size            : 12px;
            line-height          : 16px;

            & > i
            {
              display  : block;
              position : absolute;
              width    : 18px;
              height   : 18px;
              top      : calc(50% - 9px);
              left     : 10px;
              font-size: 18px;
            }
          }
        }
        & > .tlist
        {
          display      : block;
          position     : relative;
          width        : 100%;
          height       : 365px;
          border-bottom: solid 1px rgba(0,0,0,0.1);

          & > .loader
          {
            display        : flex;
            justify-content: center;
            align-items    : center;
            position       : absolute;
            width          : 100%;
            height         : 100%;
            top            : 0;
            left           : 0;
            background     : rgba(255,255,255,1);
            z-index        : 1;

            & > i
            {
              display          : block;
              position         : absolute;
              width            : 60px;
              height           : 60px;
              line-height      : 60px;
              text-align       : center;
              font-size        : 50px;
              opacity          : 0.1;
              color            : #6f6bfb;
              top              : calc(50% - 30px);
              left             : calc(50% - 30px);
              animation        : loading 0.5s ease infinite;
              -webkit-animation: loading 0.5s ease infinite;
            }
          }
          & > .empty
          {
            display           : flex;
            justify-content   : center;
            align-items       : center;
            position          : absolute;
            width             : 100%;
            height            : 100%;
            top               : 0;
            left              : 0;
            background        : rgba(255,255,255,1);
            z-index           : 1;
            box-sizing        : border-box;
            -webkit-box-sizing: border-box;
            padding           : 0px 20px;

            & > div
            {
              display   : block;
              position  : relative;
              width     : 100%;
              text-align: center;

              & > i
              {
                display    : inline-block;
                position   : relative;
                width      : 24px;
                height     : 24px;
                line-height: 24px;
                text-align : center;
                font-size  : 24px;
                margin     : 0px 0px 10px 0px;
              }
              & > span
              {
                display    : block;
                position   : relative;
                width      : 100%;
                font-size  : 12px;
                line-height: 18px;
              }
            }
          }
          & > .scroll
          {
            display   : block;
            position  : relative;
            width     : 100%;
            height    : 100%;
            overflow-y: auto;
            overflow-x: hidden;
            
            &::-webkit-scrollbar
            {
              width: 4px;
            }
            &::-webkit-scrollbar-track
            {
              background: #ffffff; 
            }
            &::-webkit-scrollbar-thumb 
            {
              background: #14579a; 
            }
            &::-webkit-scrollbar-thumb:hover 
            {
              background: #7068f9; 
            }
            & .row
            {
              display           : block;
              position          : relative;
              width             : 100%;
              overflow          : hidden;
              box-sizing        : border-box;
              -webkit-box-sizing: border-box;
              border-bottom     : solid 1px rgba(0,0,0,0.05);
              cursor            : pointer;
              background        : #ffffff;

              &:hover
              {
                background:#eef3f7 !important;
              }
              &:nth-child(even)
              {
                background:#fafafa;
              }
              &.head
              {
                cursor:default;

                &:hover
                {
                  background:#ffffff !important;
                }
              }
              &:last-child
              {
                border:none;
              }
              &.head
              {
                & .col
                {
                  font-weight: bold;
                  color      : #000000 !important;
                }
              }
              & .col
              {
                display           : block;
                position          : relative;
                float             : left;
                height            : 60px;
                line-height       : 60px;
                font-size         : 14px;
                box-sizing        : border-box;
                -webkit-box-sizing: border-box;
                padding           : 0px 20px;
                white-space       : nowrap;
                text-overflow     : ellipsis;
                overflow          : hidden;

                & > div
                {
                  display           : inline-block;
                  position          : relative;
                  box-sizing        : border-box;
                  -webkit-box-sizing: border-box;
                  padding           : 0px 24px 0px 0px;
                  
                  &.sort
                  {
                    cursor:pointer;
                  }
                  & > i
                  {
                    display    : block;
                    position   : absolute;
                    width      : 18px;
                    height     : 18px;
                    text-align : center;
                    line-height: 18px;
                    font-size  : 18px;
                    top        : calc(50% - 9px);
                    right      : 0px;
                  }
                }
                &.checkbox
                {
                  width : 54px;
                  cursor: pointer;

                  & > i
                  {
                    display              : block;
                    position             : absolute;
                    width                : 18px;
                    height               : 18px;
                    box-sizing           : border-box;
                    -webkit-box-sizing   : border-box;
                    border               : solid 2px #7873f1;
                    top                  : calc(50% - 9px);
                    left                 : 20px;
                    border-radius        : 4px;
                    -webkit-border-radius: 4px;
                    background           : #ffffff;
                  }
                  &.active
                  {
                    & > i
                    {
                      background:#7873f1;
                    }
                  }
                }
                &.name
                {
                  width:calc(100% - 534px);
                }
                &.unit
                {
                  width      : 140px;
                  font-weight: bold;
                  color      : #5883d1;
                }
                &.type
                {
                  width : 140px;
                }
                &.value
                {
                  width     : 200px;
                  text-align: right;

                  &.error
                  {
                    color:#dd5454;
                  }
                }
              }
            }
          }
        }
      }
      & > .more
      {
        display           : none;
        position          : relative;
        width             : 100%;
        box-sizing        : border-box;
        -webkit-box-sizing: border-box;
        padding           : 20px;

        & > div
        {
          display              : block;
          position             : relative;
          max-width            : 200px;
          margin               : 0 auto;
          width                : 100%;
          border-radius        : 8px;
          -webkit-border-radius: 8px;
          height               : 40px;
          line-height          : 40px;
          text-align           : center;
          font-size            : 12px;
          cursor               : pointer;
          text-transform       : uppercase;
          background           : #dd5454;
          color                : #ffffff;
          box-shadow           : -1px 13px 20px -5px rgba(0,0,0,0.2);
          -webkit-box-shadow   : -1px 13px 20px -5px rgba(0,0,0,0.2);
          transition           : all 0.3s ease;
          -webkit-transition   : all 0.3s ease;
          opacity              : 1;

          &.disabled
          {
            pointer-events    : none;
            opacity           : 0.4;
            box-shadow        : none;
            -webkit-box-shadow: none;
          }
        }
      }
    }
  }
}

@keyframes loading {
    from {transform:rotate(0deg);-webkit-transform:rotate(0deg);}
    to {transform:rotate(360deg);-webkit-transform:rotate(360deg);}
}
@-webkit-keyframes loading {
    from {transform:rotate(0deg);-webkit-transform:rotate(0deg);}
    to {transform:rotate(360deg);-webkit-transform:rotate(360deg);}
}

@media screen and (max-width: 624px)
{
  .vander-container
  {
    & > .vcover
    {
      & > .tabcontent
      {
        & > .table
        {
          & > .thead
          {
            font-size:16px;
          }
          & > .tlist
          {
            height:auto;

            & > .loader
            {
              display:none;
            }
            & > .empty
            {
              position: relative;
              height  : 360px;
            }
            & > .scroll
            {
              & .row
              {
                padding:12px 0px 12px 38px;
                
                &.head
                {
                  padding:0px;

                  & .col
                  {
                    float      : none;
                    height     : 54px;
                    line-height: 54px;

                    &.checkbox
                    {
                      position: relative;
                      top     : auto;
                      left    : auto;
                    }
                    &.name, &.unit, &.type, &.value
                    {
                      display:none;
                    }
                  }
                }
                & .col
                {
                  float      : left;
                  line-height: 24px;
                  height     : 24px;

                  &.checkbox
                  {
                    position: absolute;
                    left    : 0px;
                    top     : calc(50% - 27px);
                    height  : 54px;
                  }
                  &.name
                  {
                    width:calc(100% - 120px);
                  }
                  &.unit
                  {
                    width     : 120px;
                    text-align: right;
                  }
                  &.type
                  {
                    width  : 88px;
                    opacity: 0.4;
                    padding: 0px 0px 0px 20px
                  }
                  &.value
                  {
                    width:calc(100% - 88px);
                    text-align:right;
                    padding: 0px 20px 0px 0px;
                  }
                }
              }
            }
          }
        }
        & > .more
        {
          display:block;
        }
      }
    }
  }
}

</style>
