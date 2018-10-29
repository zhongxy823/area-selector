/* eslint-disable */
<template>
  <div>
    <transition name="addressbox-bounce">
      <div class="address-choose" v-if="visible">
        <div class="title">
          <span v-text="title"></span>
          <i class="el-icon-close" @click="handleAddressClose"></i>
        </div>
        <div class="region__title" @click="handleRegionChange">
          <span step="1" list="provinceList" v-text="regionParams.province.name"></span>
          <span step="2" list="cityList" v-text="regionParams.city.name"></span>
          <span step="3" list="districtList" v-text="regionParams.district.name"></span>
          <span step="4" list="streetList" v-text="regionParams.street.name"></span>
        </div>
        <div class="region__list">
          <div v-if="activeRegion <= getRegionStep.step">
            <p v-for="(item, index) in chooseRegionList" :key="index" @click="handleRegionChoose(item)">
              <span v-text="item.name"></span>
              <i v-if="item.choose" class="el-icon-check f-fr region__icon"></i>
            </p>
          </div>
          <div class="region-prompt" v-else>
            <span v-if="getRegionStep.step < 4">请先选择{{ getRegionStep.cn }}</span>
            <span v-else>您选择的是{{ this.regionResult }}</span>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
import addressList from '@/static/json/data.json'
export default {
  name: '',
  props: {
    title: {
      type: String,
      default: '配送至'
    },
    visible: {
      type: Boolean,
      default: false
    },
    targetId: {
      type: String,
      default: ''
    }
  },
  data: function () {
    return {
      activeRegion: 1,
      chooseRegionList: [],
      provinceList: [],
      cityList: [],
      districtList: [],
      streetList: [],
      regionParams: {
        province: {
          name: '请选择省',
          code: null
        },
        city: {
          name: '请选择市',
          code: null
        },
        district: {
          name: '请选择区',
          code: null
        },
        street: {
          name: '请选择',
          code: null
        }
      },
      regionParamsInit: {
        province: {
          name: '请选择省',
          code: null
        },
        city: {
          name: '请选择市',
          code: null
        },
        district: {
          name: '请选择区',
          code: null
        },
        street: {
          name: '请选择',
          code: null
        }
      },
      sllRegionTextList: [
        {
          cn: '省',
          en: 'province',
          list: 'provinceList',
          nextList: 'cityList',
          step: 1
        }, {
          cn: '市',
          en: 'city',
          list: 'cityList',
          nextList: 'districtList',
          step: 2
        }, {
          cn: '区',
          en: 'district',
          list: 'districtList',
          nextList: 'streetList',
          step: 3
        }, {
          cn: '',
          en: 'street',
          list: 'streetList',
          nextList: '',
          step: 4
        }
      ],
      regionResult: ''
    }
  },
  created () {},
  mounted () {
    // 在样式表内添加样式 addRule 需要添加之前没有的样式，如果在css中存在的样式则无法修改
    document.styleSheets[0].addRule('.region__title::after', `left: 10px`)
    // 进行深拷贝
    this.chooseRegionList = JSON.parse(JSON.stringify(addressList))
    this.provinceList = JSON.parse(JSON.stringify(addressList))
  },
  methods: {
    // 点击请选择切换
    handleRegionChange (e) {
      let ev = e || window.event
      let target = ev.target || ev.srcElement
      if (target.nodeName.toLowerCase() === 'span') {
        this.activeRegion = parseInt(target.getAttribute('step'))
        this.chooseRegionList = this[target.getAttribute('list')]
        document.styleSheets[0].addRule('.region__title::after', `left: ${target.offsetLeft + 10}px`)
      }
    },
    // 选择省市区街道
    handleRegionChoose (item) {
      let ret
      // 当前所在的省或市或区或街道的text信息
      let _current = this.sllRegionTextList[this.activeRegion - 1]
      // 将选择的区域的下一级选出来并将其他曾经选过的区域choose置为false
      for (let i = 0, len = this[_current.list].length; i < len; i++) {
        if (this[_current.list][i].code === item.code) {
          this.$set(this[_current.list][i], 'choose', true)
          ret = item
        } else {
          this.$set(this[_current.list][i], 'choose', false)
        }
      }
      // 当选择到街道前
      if (this.activeRegion < 4) {
        // 将之前选过或未选过的下一级的choose置为false
        this[_current.nextList].forEach((value) => {
          value.choose = false
        })
        // 将之前选过或未选过的下一级所有的的regionParams内容置为默认
        for (let j = this.activeRegion - 1; j < 4; j++) {
          this.regionParams[this.sllRegionTextList[j].en].name = `请选择${this.sllRegionTextList[j].cn}`
          this.regionParams[this.sllRegionTextList[j].en].code = null
        }
        // 移动特效
        document.styleSheets[0].addRule('.region__title::after', `left: ${this.getStepTitleWidth}px`)
        // 将下一级数据提取出来
        this[_current.nextList] = ret.childs
        this.chooseRegionList = ret.childs
        // 将选择的区域填入regionParams中
        this.regionParams[_current.en].name = item.name
        this.regionParams[_current.en].code = item.code
        this.activeRegion = this.activeRegion + 1
      } else {
        // 将选择的区域填入regionParams中
        this.regionParams[_current.en].name = item.name
        this.regionParams[_current.en].code = item.code
        this.activeRegion = this.activeRegion + 1
        this.regionResult = `${this.regionParams.province.name} ${this.regionParams.city.name} ${this.regionParams.district.name} ${this.regionParams.street.name}`
        this.$emit('handleAreaChoose', this.regionResult)
      }
    },
    handleAddressClose () {
      this.$emit('handleAreaClose')
    }
  },
  computed: {
    getRegionStep () {
      if (!this.regionParams.province.code) {
        return this.sllRegionTextList[0]
      } else if (!this.regionParams.city.code) {
        return this.sllRegionTextList[1]
      } else if (!this.regionParams.district.code) {
        return this.sllRegionTextList[2]
      } else {
        return this.sllRegionTextList[3]
      }
    },
    getStepTitleWidth () {
      return (10 + 20 * this.activeRegion) + ((document.body.offsetWidth - 100) / 4) * this.activeRegion
    }
  },
  watch: {
    visible (val) {
      let app = document.getElementById(this.targetId)
      function bodyScroll (event) {
        event.preventDefault()
        event.stopPropagation()
      }
      let child
      if (val) {
        let div = document.createElement('div')
        div.id = 'mask'
        app.appendChild(div)
        child = document.getElementById('mask')
        child.addEventListener('touchmove', bodyScroll)
      } else {
        child = document.getElementById('mask')
        child.parentNode.removeChild(child)
      }
    }
  }
}
</script>

<style scoped>
  .address-choose{
    position: fixed;
    background: #fff;
    bottom: 0;
    left: 50%;
    transform: translate3d(-50%, 0, 0);
    backface-visibility: hidden;
    transition: transform .3s ease-out;
    width: 100%;
    z-index: 666;
  }
  .title{
    text-align: center;
    position: relative;
    height: 40px;
    line-height: 40px;
    font-size: 16px;
  }
  .title span, .title i{
    display: inline-block;
  }
  .title i{
    position: absolute;
    right: 15px;
    top: 12px;
  }
  .region__title{
    margin: 5px 10px 0;
    font-size: 0;
    height: 35px;
    line-height: 35px;
    position: relative;
  }
  .region__title span{
    padding: 0 10px;
    display: inline-block;
    font-size: 15px;
    width: calc((100vw - 100px) / 4);
    text-align: center;
    overflow: hidden;
    text-overflow:ellipsis;
    white-space: nowrap;
  }
  .region__title:after{
    background: #1991EB;
    content: "";
    height: 2px;
    position: absolute;
    bottom: 0;
    width: calc((100vw - 100px) / 4);
    transition:left 0.3s ease-out;
    -webkit-transition:left 0.3s ease-out; /* Safari */
  }
  .region__list{
    height: 240px;
    overflow: auto;
    -webkit-overflow-scrolling: touch;
    overflow-scrolling: touch;
  }
  .region__list p{
    line-height: 40px;
    text-indent: 20px;
    color: #888888;
    font-size: 14px;
    text-align: left;
    margin: 0;
    padding: 0;
  }
  .region-prompt{
    text-align: center;
    line-height: 100px;
    font-size: 18px;
  }
  .region__icon{
    color: #1991EB;
    display: inline-block;
    margin-right: 20px;
    line-height: 40px;
    font-size: 20px;
    font-weight: bold;
  }

  .addressbox-bounce-enter,
  .addressbox-bounce-leave-active {
    transform: translate3d(-50%, 100%, 0);
  }
  /* iphone5 */
  @media screen and (max-width:320px){
    .region__title span {
      font-size: 13px;
    }
  }
</style>
