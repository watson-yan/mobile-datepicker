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
            <li></li>
            <li></li>
            <li v-for="item of yList">{{item}}</li>
            <li></li>
            <li></li>
          </ul>
        </section>
        <section class="picker-column">
          <ul ref="month" data-type="month">
            <li></li>
            <li></li>
            <li v-for="item of mList">{{item}}</li>
            <li></li>
            <li></li>
          </ul>
        </section>
        <section class="picker-column">
          <ul ref="date" data-type="date">
            <li></li>
            <li></li>
            <li v-for="item of dList">{{item}}</li>
            <li></li>
            <li></li>
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
    props: {
      datetime: {
        type: Date,
        default() {
          return new Date()
        }
      }
    },
    data() {
      return {
        yList: this.getArray(1900, 2100),
        mList: ['一月', '二月', '三月', '四月', '五月', '六月', '七月', '八月', '九月', '十月', '十一月', '十二月'],
        dList: this.getArray(1, 30),
        start: 0,
        startTime: 0,
        endTime: 0,
        transform: {
          year: 0,
          month: 0,
          date: 0
        },
        current: null
      }
    },
    // Todo
    // computed: {
    //   dList() {

    //   }
    // },
    mounted() {
      this.initComponents()
    },
    methods: {
      initComponents() {
        this.initTransform(this.datetime.getFullYear(), 
                      this.datetime.getMonth() + 1, 
                      this.datetime.getDate())
        this.eventListener([this.$refs.year, this.$refs.month, this.$refs.date])
        this.current = this.datetime
      },
      eventListener(ele) {
        if (ele instanceof Array) {
          ele.forEach((item) => {
            this.eventListener(item)
          })
          return
        }
        // 对应滑动块和滑动距离
        const limit = {
          year: -((this.yList.length + 4) * 50) + 250,
          month: -550,
          date: -((this.dList.length + 4) * 50) + 250
        }
        console.warn(limit)
        // 记录开始位置
        ele.addEventListener('touchstart', (e) => {
          e.preventDefault()
          this.start = e.touches[0].pageY
          this.startTime = parseInt(new Date().getTime())
        })
        // 滑动过程中事件
        ele.addEventListener('touchmove', (e) => {
          e.preventDefault()
          const distance = e.touches[0].pageY - this.start
          if (e.target.nodeName === 'LI') {
            const parent =  e.target.parentNode
            const type = parent.dataset.type
            let offset = this.transform[type] + distance
            console.warn(`distance: ${distance}`)
            console.warn(`offset${offset}`)
            // 超过最大或最小值时，阻止偏移
            if (offset < limit[`${type}`]) {
              offset = limit[`${type}`]
            }
            if ( offset > 0) {
              offset = 0
            }
            this.$refs[`${type}`].style.transform = `translateY(${offset}px)`
          }
        })
        ele.addEventListener('touchend', (e) => {
          const touch = e.changedTouches ? e.changedTouches[0] : e
          let distance = touch.pageY - this.start
          this.endTime = parseInt(new Date().getTime())
          const duration = this.endTime - this.startTime
          if (e.target.nodeName === 'LI') {
            const parent =  e.target.parentNode
            const type = parent.dataset.type
            // this.transform[type] = this.transform[type] + distance
            let offset = 0
            if (duration < 200) {
              const speed = Math.abs(distance) / duration
              offset = distance * speed * 10
            } else {
              offset = distance
            }
            let movePageY = this.transform[type] + offset
            console.warn(`movePageY:${movePageY}`)
            movePageY = movePageY < limit[type] ? limit[type] : movePageY
            movePageY = movePageY > 0 ? 0 : movePageY
            // this.transform[type] = movePageY
            this.transform[type] = movePageY
            console.warn(this.transform[type])
            this.setTransform(this.$refs[`${type}`], this.transform[type], 500)
            console.warn(`transform[${type}]: ${this.transform[type]}`)
          }
        })
      },
      initTransform(y, m, d) {
        this.transform.year = -(y - this.yList[0]) * 50
        this.transform.month = -50 * (m - 1)
        this.transform.date = -50 * (d - 1)
        this.$refs.year.style.transform = `translateY(${this.transform.year}px)`
        this.$refs.month.style.transform = `translateY(${this.transform.month}px)`
        this.$refs.date.style.transform = `translateY(${this.transform.date}px)`
      },
      setTransform(ele, distance, moveTime) {
        ele.style.transitionProperty = 'all'
        ele.style.transitionTimingFunction = 'cubic-bezier(0.1, 0.57, 0.1, 1)'
        ele.style.transitionDuration = `${moveTime}ms`
        ele.style.transform = `translate(0, ${distance}px, 0)`
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
