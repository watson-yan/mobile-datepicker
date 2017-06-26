<template>
  <div class="vue-datepicker">
    <div class="vue-datepicker-header">
      <a>取消</a>
      <span>请选择日期</span>
      <a>完成</a>
    </div>
    <div class="vue-datepicker-wrap">
      <div class="vue-datepicker-panel">
        <section class="picker-column">
          <ul ref="year" data-type="year">
            <li v-for="item of yList">{{item}}</li>
          </ul>
        </section>
        <section class="picker-column">
          <ul ref="month" data-type="month">
            <li v-for="item of mList">{{item}}</li>
          </ul>
        </section>
        <section class="picker-column">
          <ul ref="date" data-type="date">
            <li v-for="item of dList">{{item}}</li>
          </ul>
        </section>
      </div>
      <!-- 遮罩层 -->
      <section class="wrap-mask-top"></section>
      <section class="wrap-mask-bottom"></section>
    </div>
  </div>
</template>
<script>
  export default {
    data() {
      return {
        begin: new Date(),
        yList: this.getArray(1900, 2100),
        mList: ['一月', '二月', '三月', '四月', '五月', '六月', '七月', '八月', '九月', '十月', '十一月', '十二月'],
        dList: this.getArray(1, 30),
        start: 0,
        end: 0,
        startTime: 0,
        endTime: 0,
        yTransform: 0,
        mTransform: 0,
        dTransform: 0,
        current: null
      }
    },
    mounted() {
      this.initComponents()
    },
    methods: {
      initComponents() {
        this.transform(this.begin.getFullYear(), 
                      this.begin.getMonth() + 1, 
                      this.begin.getDate())
        this.eventListener([this.$refs.year, this.$refs.month, this.$refs.date])
        this.current = this.begin
      },
      eventListener(ele) {
        if (ele instanceof Array) {
          ele.forEach((item) => {
            this.eventListener(item)
          })
          return
        }
        // 对应滑动块和滑动距离
        const reactive = {
          year: {
            max: this.yList.length * 50,
            transform: this.yTransform
          },
          month: {
            max: 600,
            transform: this.mTransform
          },
          date: {
            max: this.dList.length * 50,
            transform: this.dTransform
          }
        }
        // 记录开始位置
        ele.addEventListener('touchstart', (e) => {
          e.preventDefault()
          this.start = e.touches[0].pageY
          this.startTime = parseInt(new Date().getTime())
        })
        ele.addEventListener('touchmove', (e) => {
          e.preventDefault()
          const distance = this.start - e.touches[0].pageY
          if (e.target.nodeName === 'LI') {
            const parent =  e.target.parentNode
            const type = parent.dataset.type
            const offset = reactive[`${type}`].transform + distance
            // 超过最大或最小值时，阻止偏移
            if (offset + 150 > reactive[`${type}`].max || offset < -100) {
              return
            }
            this.$refs[`${type}`].style.transform = 
                    `translateY(${-offset}px)`
            this.end = e.touches[0].pageY
          }
        })
        ele.addEventListener('touchend', (e) => {
          let distance = this.start - this.end
          this.endTime = parseInt(new Date().getTime())
          if (e.target.nodeName === 'LI') {
            const parent =  e.target.parentNode
            const type = parent.dataset.type
             // 滚动动画
            const duration = this.endTime - this.startTime
            if (duration < 300) {
              const speed = Math.abs(distance) / duration
              let moveTime = duration * speed * 20;// 惯性滚动时间(动画)
              moveTime = moveTime > 2000 ? 2000 : moveTime
              console.warn(`speed: ${speed}`)
              console.warn(`before: ${distance}`)
              distance += distance * speed * 10;// 惯性移动距离
              console.warn(distance)
              reactive[`${type}`].transitionDuration = `${moveTime}ms`
            } else {
              reactive[`${type}`].transitionDuration = '500ms'
            }
            

            // 偏移量
            let offset = reactive[`${type}`].transform + distance
            // 修正偏移量
            if (offset + 150 > reactive[`${type}`].max) {
              offset = reactive[`${type}`].max - 150
            }
            if (offset % 50) {
              const diff = offset % 50
              offset = diff > 25 ? offset + (50 - diff) : offset - diff
            }
            
            reactive[`${type}`].transitionProperty = 'all'
            reactive[`${type}`].transform = offset
            this.$refs[`${type}`].style.transform = 
                    `translateY(-${reactive[`${type}`].transform}px)`
          }
        })
      },
      transform(y, m, d) {
        this.yTransform = (y - this.yList[0]) * 50 - 100
        this.mTransform = (m - 1) * 50 - 100
        this.dTransform = (d - 1) * 50 - 100
        this.$refs.year.style.transform = `translateY(${-this.yTransform}px)`
        this.$refs.month.style.transform = `translateY(${-this.mTransform}px)`
        this.$refs.date.style.transform = `translateY(${-this.dTransform}px)`
      },
      getArray(start, end) {
        const list = []
        for (let i = start; i <= end; i++) {
          list.push(i)
        }
        return list
      }
    }
  }
</script>
<style lang="scss" src="./datepicker.scss"></style>
