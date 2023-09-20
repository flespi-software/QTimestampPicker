<template>
  <div class="inline-block fit">
    <q-input
      v-if="withInput"
      outlined
      dense
      v-model="datetime"
      @focus="focusemit"
      @click="show"
      :disable="disable"
      readonly
      :label="label"
      :suffix="suffix"
      :color="color"
      :error="error"
    >
      <template v-slot:append>
        <q-icon name="mdi-calendar" class="cursor-pointer" @click="show" :color="color" />
      </template>
    </q-input>
    <q-btn
      v-else
      flat
      outlined
      dense
      icon="mdi-calendar"
      class="cursor-pointer"
      @click="show"
      :color="color"
    />
    <q-dialog
      v-model="showdialog"
      @show="onDialogShow"
      @hide="onDialogHide"
      :maximized="autoMaximized && $q.platform.is.mobile"
      :content-style="`z-index:${popupZindex || 6000};`"
      no-backdrop-dismiss
      transition-show="scale"
      transition-hide="scale">
      <q-card class="q-pa-none">
        <q-card-section class="q-pa-none">
          <div :class="`bg-${color} absolute-top-left absolute-top-right`" style="min-height:86px" />
          <div class="row items-start">
            <q-carousel
              v-model="slide"
              transition-prev="slide-right"
              transition-next="slide-left"
              swipeable
              animated
              class="fit bg-transparent"
            >
              <q-carousel-slide name="date" class="row justify-center content-center q-pa-none fit">
                <q-date
                  style="max-width:290px"
                  flat
                  square
                  v-model="proxydatetime"
                  :mask="format"
                  @input="slide='time'"
                  :color="color"
                  :options="checkDay"
                  no-unset
                />
              </q-carousel-slide>
              <q-carousel-slide name="time" class="row justify-center content-center q-pa-none fit">
                <q-time
                  style="max-width:290px"
                  flat
                  square
                  v-model="proxydatetime"
                  :mask="format"
                  format24h
                  :with-seconds="withSeconds"
                  :color="color"
                  @click.prevent.capture
                />
              </q-carousel-slide>
            </q-carousel>
          </div>
          <div class="row items-center justify-end q-pa-sm">
            <q-btn-toggle
              dark
              v-model="slide"
              color="grey-6"
              :toggle-color="color"
              flat
              :options="[
                {value: 'date', icon: 'mdi-calendar'},
                {value: 'time', icon: 'mdi-clock-outline'}
              ]"
            />
            <q-space />
            <q-btn label="Cancel" :color="color" flat v-close-popup />
            <q-btn label="OK" :color="color" flat @click="save" v-close-popup />
          </div>
        </q-card-section>
      </q-card>
    </q-dialog>
  </div>
</template>

<script>
import { date } from 'quasar'

export default {
  name: 'QTimestampPicker',
  props: {
    value: {
      type: Number,
      default: Date.now()
    },
    disable: [Boolean],
    label: [String],
    suffix: [String],
    format: {
      type: String,
      default: 'YYYY-MM-DD HH:mm:ss'
    },
    color: {
      type: String,
      default: 'primary'
    },
    error: {
      type: Boolean,
      default: undefined
    },
    // maximized for mobile view
    autoMaximized: {
      type: Boolean,
      default: true
    },
    // filter dates (disabled days)
    dateOptionsFn: [Function],
    // time with seconds
    withSeconds: {
      type: Boolean,
      default: true
    },
    popupZindex: {
      type: Number,
      default: 6000
    },
    withInput: {
      type: Boolean,
      default: true
    }
  },
  data () {
    return {
      datetime: date.formatDate(this.value ? (this.value) : Date.now(), this.format),
      proxydatetime: '',
      showdialog: false,
      slide: 'date'
    }
  },
  methods: {
    show() {
      if (!this.disable) {
        this.slide = 'date'
        this.showdialog = true
        this.updateProxy()
      }
    },
    input (val) {
      this.$emit('input', parseInt(val / 1000))
    },
    focusemit (evt) {
      this.$emit('focus', evt)
    },
    updateProxy () {
      this.proxydatetime = this.datetime
    },
    save () {
      this.datetime = this.proxydatetime
      this.$emit('input', parseInt(date.formatDate(date.extractDate(this.datetime, this.format).getTime(), 'x')))
    },
    // "options" - func for date-picker
    checkDay (d) {
      return this.dateOptionsFn
        ? this.dateOptionsFn(date.extractDate(d, 'YYYY/MM/DD'))
        : true
    },
    // dialog show handler
    onDialogShow () {
      this.$emit('toggle', true)
    },
    // dialog hide handler
    onDialogHide (v) {
      this.$emit('toggle', false)
    }
  }
}
</script>

<style lang="stylus">
</style>
