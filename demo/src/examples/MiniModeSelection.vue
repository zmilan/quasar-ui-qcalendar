<template>
  <div class="row justify-center" style="max-width: 800px; width: 100%; overflow: hidden;">
    <div class="q-gutter-sm">
      <q-checkbox v-model="mobile" label="Use Touch (set if on mobile)" />
    </div>
    <q-separator class="full-width" />
    <div class="row justify-center" style="max-width: 800px; width: 100%; overflow: hidden;">
      <q-calendar
        ref="calendar"
        v-model="selectedDate"
        view="month"
        locale="en-us"
        mini-mode
        :selected-start-end-dates="startEndDates"
        :day-class="classDay"
        @mousedown:day="onMouseDownDay"
        @mouseup:day="onMouseUpDay"
        @mousemove:day="onMouseMoveDay"
        style="max-width: 300px;"
      />
    </div>
  </div>
</template>

<script>
// normally you would not import "all" of QCalendar, but is needed for this example to work with UMD (codepen)
import QCalendar from 'ui' // ui is aliased from '@quasar/quasar-ui-qcalendar'

function leftClick (e) {
  return e.button === 0
}

export default {
  data () {
    return {
      splitterModel: 90, // start at 90%
      selectedDate: '',
      miniMode: false,
      anchorTimestamp: '',
      otherTimestamp: '',
      mouseDown: false,
      mobile: false
    }
  },

  computed: {
    startEndDates () {
      const dates = []
      if (this.anchorDayIdentifier !== false && this.otherDayIdentifier !== false) {
        if (this.anchorDayIdentifier <= this.otherDayIdentifier) {
          dates.push(this.anchorTimestamp.date, this.otherTimestamp.date)
        } else {
          dates.push(this.otherTimestamp.date, this.anchorTimestamp.date)
        }
      }
      return dates
    },

    anchorDayIdentifier () {
      if (this.anchorTimestamp !== '') {
        return QCalendar.getDayIdentifier(this.anchorTimestamp)
      }
      return false
    },

    otherDayIdentifier () {
      if (this.otherTimestamp !== '') {
        return QCalendar.getDayIdentifier(this.otherTimestamp)
      }
      return false
    },

    lowIdentifier () {
      // returns lowest of the two values
      return Math.min(this.anchorDayIdentifier, this.otherDayIdentifier)
    },

    highIdentifier () {
      // returns highest of the two values
      return Math.max(this.anchorDayIdentifier, this.otherDayIdentifier)
    }
  },

  methods: {
    calendarNext () {
      this.$refs.calendar.next()
    },

    calendarPrev () {
      this.$refs.calendar.prev()
    },

    classDay (timestamp) {
      if (this.anchorDayIdentifier !== false && this.otherDayIdentifier !== false) {
        return this.getBetween(timestamp)
      }
    },

    getBetween (timestamp) {
      const nowIdentifier = QCalendar.getDayIdentifier(timestamp)
      return {
        'q-selected-day-first': this.lowIdentifier === nowIdentifier,
        'q-selected-day': this.lowIdentifier <= nowIdentifier && this.highIdentifier >= nowIdentifier,
        'q-selected-day-last': this.highIdentifier === nowIdentifier
      }
    },

    onMouseDownDay ({ scope, event }) {
      if (leftClick(event)) {
        if (this.mobile === true &&
          this.anchorTimestamp !== null &&
          this.otherTimestamp !== null &&
          this.anchorTimestamp.date === this.otherTimestamp.date) {
          this.otherTimestamp = scope
          this.mouseDown = false
          return
        }
        // mouse is down, start selection and capture current
        this.mouseDown = true
        this.anchorTimestamp = scope
        this.otherTimestamp = scope
      }
    },

    onMouseUpDay ({ scope, event }) {
      if (leftClick(event)) {
        // mouse is up, capture last and cancel selection
        this.otherTimestamp = scope
        this.mouseDown = false
      }
    },

    onMouseMoveDay ({ scope, event }) {
      if (this.mouseDown === true) {
        this.otherTimestamp = scope
      }
    }
  }
}
</script>
