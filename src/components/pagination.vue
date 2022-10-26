<template>
  <div class="pagination">
    <div class="page">
        <div class="text">Row per page : </div>
        <div class="selection" :class="{'disabled' : disabled}" @click="selectpage = !selectpage">
            <div class="selected">{{ selected }}</div>
            <div class="list" v-if="selectpage">
                <div :class="{'active' : item == selected}" v-for="(item, index) in selection" :key="index" @click="selectPage(item)">{{ item }}</div>
            </div>
        </div>
    </div>
    <div class="pag">
        <div class="arr" :class="{'disabled' : current == 1 || disabled}" @click="prev()"><i class="iconfont">chevron_left</i></div>
        <div class="cur">{{ current }}</div>
        <div class="arr" :class="{'disabled' : current == total || disabled}" @click="next()"><i class="iconfont">navigate_next</i></div>
    </div>
  </div>
</template>

<script>
export default {
    props: {
        'current' : {
            type : Number,
            default : 1
        },
        'disabled' : {
            type: Boolean,
            default: false
        },
        'total' : {
            type: Number,
            default : 1
        }
    },
    watch: {

    },
    data()
    {
        return {
            'selection' : [5, 10, 15, 20],
            'selected'  : 5,
            'selectpage': false
        }
    },
    mounted()
    {
        
    },
    methods:
    {
        selectPage(num)
        {
            this.selected = num;
            this.$emit('selectPage', num);
        },
        prev()
        {
            this.$emit('prev')
        },
        next()
        {
            this.$emit('next')
        }
    }
}
</script>

<style scoped lang="scss">
.pagination
{
    display           : block;
    position          : relative;
    width             : 100%;
    box-sizing        : border-box;
    -webkit-box-sizing: border-box;
    padding           : 20px;
    height            : 74px;
    z-index           : 9;
    margin            : 20px 0px 0px 0px;

    & > .page
    {
        display : block;
        position: relative;
        float   : left;
        height  : 34px;

        & > .text
        {
            display    : block;
            position   : relative;
            float      : left;
            height     : 34px;
            line-height: 34px;
            font-size  : 14px;
            margin     : 0px 10px 0px 0px;
        }
        & > .selection
        {
            display              : block;
            position             : relative;
            float                : left;
            width                : 80px;
            height               : 34px;
            box-sizing           : border-box;
            -webkit-box-sizing   : border-box;
            border               : solid 1px rgba(0,0,0,0.4);
            background           : rgba(0,0,0,0.02);
            border-radius        : 4px;
            -webkit-border-radius: 4px;
            cursor               : pointer;
            transition           : all 0.3s ease;
            -webkit-transition   : all 0.3s ease;
            opacity              : 1;

            &.disabled
            {
                pointer-events: none;
                opacity       : 0.2;
            }
            & > .selected
            {
                display           : block;
                position          : relative;
                width             : 100%;
                font-size         : 14px;
                height            : 32px;
                line-height       : 32px;
                box-sizing        : border-box;
                -webkit-box-sizing: border-box;
                font-weight       : bold;
                padding           : 0px 12px;
            }

            & > .list
            {
                display              : block;
                position             : absolute;
                width                : calc(100% + 2px);
                max-height           : 200px;
                box-sizing           : border-box;
                -webkit-box-sizing   : border-box;
                border-radius        : 4px;
                -webkit-border-radius: 4px;
                border               : solid 1px rgba(0,0,0,0.4);
                background           : #ffffff;
                bottom               : 32px;
                left                 : -1px;
                text-align           : center;

                & div
                {
                    display           : block;
                    position          : relative;
                    width             : 100%;
                    font-size         : 14px;
                    height            : 32px;
                    line-height       : 32px;
                    box-sizing        : border-box;
                    -webkit-box-sizing: border-box;
                    font-weight       : bold;
                    padding           : 0px 12px;
                    border-bottom:solid 1px rgba(0,0,0,0.05);

                    &:hover
                    {
                        background:rgba(0,0,0,0.02);
                    }
                    &.active
                    {
                        background:rgba(0,0,0,0.02);
                    }
                }
            }
        }
    }
    & > .pag
    {
        display : block;
        position: relative;
        float   : right;
        height  : 34px;

        & .arr
        {
            display              : block;
            position             : relative;
            float                : left;
            width                : 34px;
            height               : 34px;
            border-radius        : 4px;
            -webkit-border-radius: 4px;
            box-sizing           : border-box;
            -webkit-box-sizing   : border-box;
            border               : solid 1px rgba(0,0,0,0.4);
            background           : rgba(0,0,0,0.02);
            cursor               : pointer;
            transition           : all 0.3s ease;
            -webkit-transition   : all 0.3s ease;
            opacity              : 1;

            & > i
            {
                display    : block;
                position   : absolute;
                width      : 24px;
                height     : 24px;
                text-align : center;
                font-size  : 24px;
                line-height: 24px;
                top        : calc(50% - 12px);
                left       : calc(50% - 12px);
                font-weight: bold;
            }
            &.disabled
            {
                opacity       : 0.2;
                pointer-events: none;
            }
        }
        & .cur
        {
            display              : block;
            position             : relative;
            float                : left;
            width                : 34px;
            height               : 34px;
            line-height          : 34px;
            font-size            : 14px;
            font-weight          : bold;
            text-align           : center;
            box-sizing           : border-box;
            -webkit-box-sizing   : border-box;
            margin               : 0px 6px;
        }
    }
}

@media screen and (max-width: 624px)
{
    .pagination
    {
        display:none;
    }
}
</style>