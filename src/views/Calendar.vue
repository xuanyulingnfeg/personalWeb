<template>
  <div class="Calendar">
    <div class="content">
      <div class="leftContent">
        <div class="tip">绝区零9月</div>
        <div class="tip">大事件日历</div>
        <div class="roles">
          <div class="roleItem">
            <div class="roleImg"></div>
            <div class="roleInfo">
              <div class="roleName">简·杜</div>
              <div class="versonIn">1.1下半</div>
              <div class="openTime">调频开启时间：2024/9/4</div>
              <div class="levelIco"></div>
            </div>
          </div>
        </div>
      </div>
      <div class="rightContent">
        <div class="calendarRow rowHeader" key="header">
          <div class="calendarCell" v-for="item in weekStrs" :key="item.value">
            {{ item.label }}
          </div>
        </div>
        <div
          class="calendarRow rowBody"
          v-for="(item, index) in calendarDateFowShow"
          :key="`data_${index}`"
        >
          <div
            class="calendarCell"
            v-for="dateItem in item"
            :key="dateItem.date"
            :class="{ isCurrent: dateItem.isCurrent }"
          >
            <div class="dateStr" v-show="dateItem.isCurrent">
              {{ dateItem.date }}
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="footer"></div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import dayjs from "dayjs";

const weekStrs = ref([
  {
    label: "一",
    value: 1,
  },
  {
    label: "二",
    value: 2,
  },
  {
    label: "三",
    value: 3,
  },
  {
    label: "四",
    value: 4,
  },
  {
    label: "五",
    value: 5,
  },
  {
    label: "六",
    value: 6,
  },
  {
    label: "日",
    value: 0,
  },
]);

const calendarDateFowShow = ref([]);

const eventList = [
  {
    name: "",
  },
];

onMounted(() => {
  setCalendarData();
});

function setCalendarData() {
  let holidayObj = {};
  var holidays = {};

  let thisMonth = dayjs().format("YYYY-MM");
  let dayNum = dayjs().daysInMonth();
  let dayNumForPre = dayjs(`${thisMonth}-01`)
    .subtract(1, "month")
    .daysInMonth();
  let weekDayForFirstDay = dayjs(`${thisMonth}-01`).day();
  weekDayForFirstDay = weekDayForFirstDay === 0 ? 6 : weekDayForFirstDay - 1;
  console.log(dayNum, weekDayForFirstDay);

  let calendarDate = [],
    calendarIndex = 0,
    dateIndex = 1;
  while (dayNum >= dateIndex) {
    calendarDate[calendarIndex] = [];

    for (var k = 0; k < 7; k++) {
      // 首行
      if (calendarIndex === 0) {
        // 上個月的日期
        if (k < weekDayForFirstDay) {
          calendarDate[calendarIndex].push({
            date: dayNumForPre - 5 + k,
            isPreDate: true,
          });
          continue;
        }
      }

      let holidayItem = holidays[dateIndex]
        ? holidayObj[holidays[dateIndex]]
        : null;

      let dateStr = !(dateIndex <= dayNum) ? dateIndex - dayNum : dateIndex;
      dateStr = dateStr > 9 ? dateStr : `0${dateStr}`;
      let dayItem = {
        isNext: !(dateIndex <= dayNum),
        isCurrent: !!(dateIndex <= dayNum),
        date: dateStr,
        fullDate: !!(dateIndex <= dayNum) ? `${thisMonth}-${dateStr}` : null,
        holidayKey: holidayItem ? holidayItem.key : null,
        holidayStr: holidayItem ? holidayItem.label : null,
        holidayColor: holidayItem ? holidayItem.color : null,
      };
      calendarDate[calendarIndex].push(dayItem);

      // 設置日曆假期顏色條位置
      let thisItem =
        calendarDate[calendarIndex][calendarDate[calendarIndex].length - 1];
      if (!!holidayItem) {
        thisItem.holidayRange = 1;
      }
      if (calendarDate[calendarIndex].length > 1) {
        let preItem =
          calendarDate[calendarIndex][calendarDate[calendarIndex].length - 2];
        if (thisItem.holidayKey === preItem.holidayKey) {
          thisItem.holidayRange = preItem.holidayRange + 1;
        } else if (preItem.holidayRange > 0) {
          preItem.isLastestHoliday = true;
        }
      }

      dateIndex++;
    }
    calendarIndex++;

    // 每一行最後一格，如有假日，isLastestHoliday設為true
    if (
      calendarDate[calendarIndex - 1][
        calendarDate[calendarIndex - 1].length - 1
      ].holidayRange > 0
    ) {
      calendarDate[calendarIndex - 1][
        calendarDate[calendarIndex - 1].length - 1
      ].isLastestHoliday = true;
    }
  }
  // isPreDate 是否為 上個月份的日期格子
  // isCurrent 是否為 該月份的日期格子
  // isNext 是否為 下個月份的日期格子
  // isLastestHoliday 是否為 當前行，同類型假期中最後一天

  console.log("calendarDate", calendarDate);
  calendarDateFowShow.value = calendarDate;
  console.log("calendarDateFowShow", calendarDateFowShow.value);
}
</script>

<style lang="scss" scoped>
.Calendar {
  width: 1200px;

  .content {
    display: flex;

    .leftContent {
      background-color: #9f341b;
      color: #9f341b;

      .game,
      .month,
      .tip {
        text-align: center;
        font-size: 40px;
        font-weight: 900;
        color: #3c3c3c;
        position: relative;
        z-index: 0;

        &::before {
          content: "大事件日历";
          top: 0;
          left: 0;
          position: absolute;
          z-index: -1;
          font-size: 40px;
          font-weight: 900;
          -webkit-text-stroke: 7px #fff;
        }

        &:first-child::before {
          content: "";
          left: 9px;
        }
      }
    }

    .rightContent {
      background-color: #f0e6c4;
      .calendarRow {
        display: flex;
        padding: 0 10px;

        &:first-child,
        &:last-child {
          margin-bottom: 10px;
        }

        .calendarCell {
          margin: 5px;
          width: 100px;
          color: #fff199;
          position: relative;
        }

        &.rowHeader {
          background-color: #de7925;
          .calendarCell {
            line-height: 40px;
            text-align: center;
            font-weight: bold;
            font-size: 20px;
          }
        }

        &.rowBody {
          .calendarCell {
            height: 75px;
            border-radius: 20px;
            background-color: gray;
            -webkit-text-stroke: 1px black;
            text-stroke: 1px black;

            &.isCurrent {
              background-color: #ab3c23;
            }

            .dateStr {
              font-size: 36px;
              position: absolute;
              bottom: -7px;
              right: 6px;
              font-weight: 900;
            }
          }
        }
      }
    }
  }
}
</style>
