<template>
  <div
    class="q-slider non-selectable"
    :class="classes"
    @click="__click"
    v-touch-pan.horizontal="__pan"
  >
    <div ref="handle" class="q-slider-handle-container">
      <div class="q-slider-track"></div>
      <div
        v-if="markers"
        class="q-slider-mark"
        v-for="n in ((max - min) / step + 1)"
        :style="{left: (n - 1) * 100 * step / (max - min) + '%'}"
      ></div>
      <div
        class="q-slider-track active-track"
        :style="{width: percentage}"
        :class="{'no-transition': dragging, 'handle-at-minimum': value === min}"
      ></div>
      <div
        class="q-slider-handle"
        :style="{left: percentage, borderRadius: square ? '0' : '50%'}"
        :class="{dragging: dragging, 'handle-at-minimum': value === min}"
      >
        <q-chip
          pointing="down"
          square
          :color="labelColor"
          class="q-slider-label no-pointer-events"
          :class="{'label-always': labelAlways}"
          v-if="label || labelAlways"
        >
          {{ displayValue }}
        </q-chip>

        <div v-if="$q.theme !== 'ios'" class="q-slider-ring"></div>
      </div>
    </div>
  </div>
</template>

<script>
import {
  getModel,
  getPercentage,
  notDivides,
  mixin
} from './slider-utils'
import TouchPan from '../../directives/touch-pan'

export default {
  name: 'q-slider',
  directives: {
    TouchPan
  },
  mixins: [mixin],
  props: {
    value: {
      type: Number,
      required: true
    },
    labelValue: String
  },
  data () {
    return {
      dragging: false,
      currentPercentage: (this.value - this.min) / (this.max - this.min)
    }
  },
  computed: {
    percentage () {
      if (this.snap) {
        return (this.value - this.min) / (this.max - this.min) * 100 + '%'
      }
      return 100 * this.currentPercentage + '%'
    },
    displayValue () {
      return this.labelValue !== void 0
        ? this.labelValue
        : this.value
    }
  },
  watch: {
    value (value) {
      if (this.dragging) {
        return
      }
      this.currentPercentage = (value - this.min) / (this.max - this.min)
    },
    min (value) {
      if (this.value < value) {
        this.value = value
        return
      }
      this.$nextTick(this.__validateProps)
    },
    max (value) {
      if (this.value > value) {
        this.value = value
        return
      }
      this.$nextTick(this.__validateProps)
    },
    step () {
      this.$nextTick(this.__validateProps)
    }
  },
  methods: {
    __setActive (event) {
      let container = this.$refs.handle

      this.dragging = {
        left: container.getBoundingClientRect().left,
        width: container.offsetWidth
      }
      this.__update(event)
    },
    __update (event) {
      let
        percentage = getPercentage(event, this.dragging),
        model = getModel(percentage, this.min, this.max, this.step, this.decimals)

      this.currentPercentage = percentage
      if (model !== this.value) {
        this.$emit('input', model)
        this.$emit('change', model)
      }
    },
    __end () {
      this.dragging = false
      this.currentPercentage = (this.value - this.min) / (this.max - this.min)
    },
    __validateProps () {
      if (this.min >= this.max) {
        console.error('Range error: min >= max', this.$el, this.min, this.max)
      }
      else if (notDivides((this.max - this.min) / this.step, this.decimals)) {
        console.error('Range error: step must be a divisor of max - min', this.min, this.max, this.step, this.decimals)
      }
    }
  }
}
</script>
