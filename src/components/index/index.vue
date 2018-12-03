<template lang="html">
  <div class="index" ref="wrapper">
    <ul class="city-wrapper" >
      <li v-for="(k,index) in cityData" ref="list">
        <p class="title">{{k.name}}</p>
        <ul class="city-list">
          <li v-for="(j,jindex) in k.cities" class="city-item">{{j.name}}</li>
        </ul>
      </li>
    </ul>
    <ul class="fastSearch" @click="scrollTo">
      <li v-for="(item,index) in fastSearch" class="fastSearch-item" :class="index==currentIndex?'active':''" :data-index="index">{{item}}</li>
    </ul>
    <div class="fixedTitle" ref="fixedTitle">
      {{fixedTitle}}
    </div>
  </div>
</template>

<script>
  import cityData from './cityData'
  import BScroll from 'better-scroll'
  const FIXEDTITLE_HEIGHT = 40
  export default {
    data(){
      return {
        cityData: null,
        fastSearch: null,
        currentIndex: 0,
        heightList: null,
        diff: -1,
        scrollY: -1,
        fixedTitle: null
      }
    },
    created() {
      this.cityData = cityData
      this.format(cityData)
    },
    mounted() {
      setTimeout(() => {
        this._initScroll()
      }, 20)
      this.calcHeight()
    },
    methods: {
      _initScroll() {
        this.$refs.wrapper.style.height = `${document.documentElement.clientHeight}px`
        this.scroll = new BScroll(this.$refs.wrapper, {
          scrollY: true,
          scrollX: false,
          probeType: 2
        })
        this.scroll.on('scroll', pos => {
          this.scrollY = -pos.y
        })
      },
      format(list) {
        let fastSearch = []
        list.forEach((item, index) => {
          if (item.name.toString().length === 1) {
            fastSearch.push(item.name)
          } else {
            fastSearch.push('★')
          }
        })
        this.fastSearch = fastSearch
      },
      scrollTo(e) {
        let index = e.target.dataset.index
        const list = this.$refs.list
        this.currentIndex = index
        this.scroll.scrollToElement(list[index], 400)
        this.fixedTitle = this.fastSearch[index]
      },
      calcHeight() {
        const list = this.$refs.list
        let heightList = [0]
        let height = 0
        list.forEach((item, index) => {
          height += item.clientHeight
          heightList.push(height)
        })
        this.heightList = heightList
      }
    },
    watch: {
      diff(newVal) {
        let fixedTop = newVal > 0 && newVal < FIXEDTITLE_HEIGHT ? newVal - FIXEDTITLE_HEIGHT : 0
        this.$refs.fixedTitle.style['transform'] = `translate3d(0,${fixedTop}px,0)`
      },
      scrollY(newY) {
        const heightList = this.heightList
        for (let i = 0;i<heightList.length;i++) {
          let height1 = heightList[i]
          let height2 = heightList[i+1]
          // 拉到最顶部继续往下拉
          if (newY < 0) {
            this.fixedTitle = null
          } else if (newY >= height1 && newY < height2) {
            this.currentIndex = i
            // diff 下一个高度 - 当前滚动的距离 = 离固定顶部的距离
            this.diff = height2 - newY
            this.fixedTitle = this.fastSearch[i]
            if (i === 0) {
              this.fixedTitle += '热门城市'
            }
            return
          }
        }
      }
    }
  }
</script>

<style lang="stylus" scoped>
  .index
    position:relative
    overflow:hidden
    .fixedTitle
      position:fixed
      top:0
      width:100%
      line-height:40px
      background:#f5f5f5
      padding-left:10px
      color:#666
    .fastSearch
      position: absolute
      right: 10px
      top: 100px
      .fastSearch-item
        color:#666
        font-size:12px
        line-height:20px
        text-align:center
        &.active
          font-weight:900
    .city-wrapper
      .title
        line-height:40px
        background:#f5f5f5
        padding-left:10px
        color:#666
      .city-list
        padding: 0 10px
        .city-item
          height:40px
          line-height:40px
          border-bottom:1px solid #ddd
          color:#333
          font-size:14px
          &:last-child
            border:none
</style>
