<!-- eslint-disable @typescript-eslint/no-explicit-any -->
<script setup lang='ts'>
import { ref, defineProps, withDefaults, onMounted } from 'vue'
import moment from 'moment'
import 'moment/dist/locale/zh-cn'

moment.locale('zh-cn')

const props = withDefaults(defineProps<{
  size: number
}>(), {
  size: 10
})

const ansRes: any = []
const date = new Date()
const today = {
  year: date.getFullYear(),
  month: date.getMonth(),
  day: date.getDate()
}
// const endTime = moment(date, 'YYYY/MM/DD').format('L')
// const startTime = moment((today.year - 1) + '/' + (today.month + 1) + '/' + today.day, 'YYYY/MM/DD').format('L')
const visibleList = ref<Record<string, boolean>>({})
const days = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31]

function isLeapYear(year: number) {
  return (year % 4 === 0 && year % 100 !== 0) || year % 400 === 0;
}

function init() {
  //上一年开始遍历
  //下标从0开始
  for (let month = today.month; month <= 12 + today.month; month++) {
    const year = Math.floor(month / 12); //0->前一年 1->今年
    const m = month % 12;
    let remainDay
    let week
    if (month != today.month && month != 12 + today.month) {
      remainDay = days[m] + (isLeapYear(today.year - (year == 0 ? 1 : 0)) && m == 1 ? 1 : 0) // 闰年补1
      week = new Date(today.year - (year == 0 ? 1 : 0) + '/' + (m + 1) + '/1').getDay()
    } else {
      if (month == today.month) {
        remainDay = days[m] - today.day + 1 + (isLeapYear(today.year - (year == 0 ? 1 : 0)) && m == 1 ? 1 : 0) // 例如 1/1 ~ 1/2 日期相差2天，相减+1
        week = new Date(today.year - (year == 0 ? 1 : 0) + '/' + (m + 1) + '/' + today.day).getDay()
      } else {
        remainDay = today.day + (isLeapYear(today.year - (year == 0 ? 1 : 0)) && m == 1 ? 1 : 0)
        week = new Date(today.year - (year == 0 ? 1 : 0) + '/' + (m + 1) + '/1').getDay()
      }
    }

    ansRes.push({
      year,
      month: m,
      remainDay,
      week,
      rev: month == today.month // 第一个日期必须反转 例如2024/1/20 剩余11天，应该显示2024/1/20 ~ 2024/1/31
    })
  }
}

const viewsList = ref<any>({
  totalCnt: 0,
  views: {
    // '2025/01/20': 10 //格式 —— (key：日期，value：数量)
  },
  colors: {
    // '2025/01/20': '#fff000' //格式 —— (key：日期，value：色值)
  }
})

// 自定义颜色
const colorArr = ["#E0F8E0", "#C6E0C6", "#AEE6AE", "#96EA96", "#7EF07E", "#66F566", "#4EF94E", "#36FD36", "#1EFF1E", "#00CC00"]
// 根据数量获取颜色
function getColorFunc(value: number): string {
  let i = 0
  while (value > 10 && i < colorArr.length) {
    value -= 10
    i += 1
  }
  return colorArr[i]
}

function generateData() {
  // 提交日期及次数
  viewsList.value.views = {
    '2025/01/01': 10,
    '2025/01/02': 10,
    '2025/01/03': 10,
    '2025/01/04': 10,
    '2025/01/05': 10,
    '2025/01/06': 10,
    '2025/01/07': 10,
    '2025/01/08': 10,
  }
  // 总提交次数
  viewsList.value.totalCnt = 80
  // 提交次数对应的颜色
  for (const key in viewsList.value.views) {
    if (Object.prototype.hasOwnProperty.call(viewsList.value.views, key)) {
      const element = viewsList.value.views[key]
      viewsList.value.colors[key] = getColorFunc(element)
    }
  }
}

// 生成数据
// function generateData() {
//   // // startTimeStamp 是一个随机的时间戳，endTimeStamp 是今天的时间戳
//   // const startTimeStamp = new Date(startTime).getTime()
//   // // endTimeStamp 是今天的时间戳
//   // const endTimeStamp = new Date(endTime).getTime()
//   // 随机生成365个数据
//   // for (let i = 0; i < 365; i++) {
//   //   const randomTimeStamp: number = (endTimeStamp - Math.random() * (endTimeStamp - startTimeStamp)) //  随机减一个随机时间戳，相当于在今天的时间戳基础上减
//   //   const dateStr: string = moment(randomTimeStamp).format('YYYY/MM/DD')
//   //   if (!viewsList.value.views[dateStr]) {
//   //     viewsList.value.views[dateStr] = 0
//   //   }
//   //   const curCnt = Math.random() * 100 | 0 // |0去除小数点
//   //   viewsList.value.views[dateStr] += curCnt
//   //   viewsList.value.totalCnt += curCnt
//   //   viewsList.value.colors[dateStr] = getColorFunc(viewsList.value.views[dateStr])
//   // }
// }

const formatDate = (year: number, month: number, day: number) => {
  return moment(today.year + (year == 0 ? -1 : 0) + '/' + (month + 1) + '/' + (day + 1), 'YYYY/MM/DD').format('L')
}

init()
onMounted(() => {
  generateData()
})

</script>

<template>
  <div class="calander_box">
    <p class="view_title">
      近一年提交
      <span style="font-weight: bold;padding: 0 5px;">{{ viewsList?.totalCnt != null ?
        viewsList?.totalCnt : '...' }}
      </span>
      次
    </p>
    <el-scrollbar>
      <div class="mobile_wrap">
        <div class="calander_view_g_wrap">
          <div class="views_wrap" v-for="month in ansRes" :key="month" v-show="month.remainDay > 0">
            <!-- 一排 7个 加边距(20px) -->
            <div class="views_month" :style="{ height: props.size * 7 + 20 + 'px' }">
              <!-- 伪装的格子 -->
              <div class="views_day" :style="{
                width: props.size + 'px',
                height: props.size + 'px'
              }" v-for="_offset in month.week" style="background: transparent; cursor: auto;" :key="_offset">
              </div>
              <!-- 真正显示的格子 -->
              <div v-for="(_day, index) in month.remainDay" :key="index">
                <el-tooltip effect="dark" :visible="visibleList[formatDate(month.year, month.month, index)]"
                  :content="`${formatDate(month.year, month.month, !month.rev ? index : (days[month.month] - (month.remainDay - index)))} 提交 ${viewsList?.views[formatDate(month.year, month.month, index)] || 0} 次`"
                  placement="top-start">
                  <div class="views_day" @mouseenter="visibleList[formatDate(month.year, month.month, index)] = true"
                    @mouseleave="visibleList[formatDate(month.year, month.month, index)] = false" :style="{
                      background: viewsList?.colors[formatDate(month.year, month.month, index)],
                      width: props.size + 'px',
                      height: props.size + 'px'
                    }">
                  </div>
                </el-tooltip>
              </div>
            </div>
            <p style="color: #a2a2a2;">{{ month.month + 1 + '月' }}</p>
          </div>
        </div>
      </div>
    </el-scrollbar>
  </div>
</template>

<style lang='less' scoped>
.calander_box {
  width: 100%;
  padding: 20px;

  .view_title {
    font-size: 18px;
    padding-left: 10px;
    margin-bottom: 20px;
  }

  .mobile_wrap {
    width: fit-content;

    @media screen and (max-width:480px) {
      width: 800px;
      white-space: nowrap;
      overflow-anchor: auto;
    }

    .calander_view_g_wrap {
      display: flex;
      justify-content: space-between;
      // overflow: hidden;

      .views_wrap {
        width: 100%;
        margin-right: 8px;
        margin-left: 8px;

        p {
          text-align: center;
          margin-top: 10px;
        }

        .views_month {
          width: calc(100% / 12);
          height: 90px;
          display: flex;
          flex-direction: column;
          flex-wrap: wrap;

          @media screen and (max-width:1200px) {
            height: 50px;
          }

          .views_day {
            margin: 0 2px 2px 0;
            border-radius: 2px;
            background: #F7F7F8;
            cursor: pointer;

            @media screen and (max-width:1200px) {
              width: 5px;
              height: 5px;
            }
          }
        }
      }
    }
  }

}
</style>
