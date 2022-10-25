<template>
  <div class="vander-container">
    <div class="bg"></div>
    <div class="vcover">
      <div class="tabheader">
        <div class="tab" :class="{'active' : selectedTab == tindex}" v-for="(tab, tindex) in tabList" :key="tindex">{{ tab }}</div>
      </div>
      <div class="tabcontent" v-if="selectedTab == 0">
        <div class="summary">
        </div>

        <div class="table">
          <div class="thead">Exchanges Rate</div>
          <div class="tlist">
            <table>
              <tr v-for="(rate, rindex) in ratesData" :key="rindex">
                <td></td>
                <td>{{ rate.name }}</td>
                <td style="text-align:center">{{ rate.unit }}</td>
                <td style="text-align:center">{{ rate.type }}</td>
                <td style="text-align:right">{{ rate.value }}</td>
              </tr>
            </table>
            <!-- <div class="row" v-for="(rate, rindex) in ratesData" :key="rindex">
              <div class="col checkbox"></div>
              <div class="col unit">{{ rate.unit }}</div>
              <div class="col name">{{ rate.name }}</div>
              <div class="col type">{{ rate.type }}</div>
              <div class="col value">{{ rate.value }}</div>
            </div> -->
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
export default {
    data(){
        return {
            'tabList'    : ['Pagination List', 'Lazy Load Scroll', 'View More Button'],
            'selectedTab': 0,
            'ratesData'  : []
        };
    },
    created () {
        
    },
    mounted() {
        this.getData();
    },
    destroyed(){
    },
    methods:{
      getData()
      {
        this.ratesData = [];

        axios
        .get('https://api.coingecko.com/api/v3/exchange_rates')
        .then((res) => 
        {
          if(res.data && res.data.rates)
          {
            let dataArray = Object.entries(res.data.rates).map(([key, obj]) => {
              return obj
            });
            this.ratesData = JSON.parse(JSON.stringify(dataArray));
          }
        })
        .catch(error => console.log(error))
      }
    }
}
</script>

<style scoped lang="scss">

.bg
{
  display   : block;
  position  : absolute;
  height    : 300px;
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
  padding           : 40px 20px;
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

    & > .tabheader
    {
      display : block;
      position: relative;
      width   : 100%;
      overflow: hidden;
      margin  : 0px 0px 20px 0px;

      & .tab
      {
        display              : block;
        position             : relative;
        float                : left;
        height               : 42px;
        line-height          : 40px;
        font-size            : 14px;
        color                : #ffffff;
        background           : rgba(0,0,0,0.1);
        box-sizing           : border-box;
        -webkit-box-sizing   : border-box;
        padding              : 0px 16px;
        border               : solid 1px #ffffff;
        border-radius        : 8px;
        -webkit-border-radius: 8px;
        cursor               : pointer;
        margin               : 0px 16px 0px 0px;
        transition           : all 0.3s ease;
        -webkit-transition   : all 0.3s ease;

        &:last-child
        {
          margin:0px;
        }
        &.active
        {
          pointer-events: none;
          background    : #ffffff;
          color         : #000000;
        }
      }
    }
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
          padding           : 20px;
          font-size         : 18px;
          font-weight       : bold;
        }
        & > .tlist
        {
          display : block;
          position: relative;
          width   : 100%;

          & > table
          {
            width:100%;

            & tr
            {
              background:#ffffff;
              
              &:hover
              {
                background:#f3fafc;
              }
              & td
              {
                padding    : 16px 20px;
                font-size  : 14px;
                line-height: 18px;
              }
            }
          }
          & .row
          {
            display      : block;
            position     : relative;
            width        : 100%;
            overflow     : hidden;
            border-bottom: solid 1px rgba(0,0,0,0.1);

            & .col
            {
              display           : block;
              position          : relative;
              float             : left;
              white-space       : nowrap;
              overflow          : hidden;
              text-overflow     : ellipsis;
              font-size         : 14px;
              line-height       : 18px;
              box-sizing        : border-box;
              -webkit-box-sizing: border-box;
              padding           : 16px 20px;

              &.unit
              {
                width     : 120px;
                text-align: left;
                font-weight:bold;
              }
              &.name
              {
                width:calc(100% - 550px);
              }
              &.type
              {
                width     : 150px;
                text-align: center;
              }
              
              &.value
              {
                width     : 280px;
                text-align: right;
              }
            }
          }
        }
      }
    }
  }
}

</style>
