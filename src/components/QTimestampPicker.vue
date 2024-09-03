<template>
  <div class="inline-block fit">
    <q-input
      v-if="withInput"
      outlined
      dense
      :value="datetime"
      @change="datetime = $event.target.value"
      @input="applyInput"
      :disable="disable"
      :label="label"
      :suffix="suffix"
      :color="color"
      :error="error"
      @focus="focusemit"
      @blur="(ev) => { parseDate(datetime) }"
      :readonly="!editableInput"
      @click="!editableInput ? show : () => {}"
      :class="!editableInput ? 'cursor-pointer' : ''"
      >
      <template v-slot:append>
        <q-btn flat round dense icon="mdi-calendar" class="cursor-pointer" @click="show" :color="color" />
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
          <div :class="`bg-${color} absolute-top-left`" style="min-height:86px" />
          <div class="row items-start">
            <div :class="`fit row ${splitView?'gt-xs' : ''}`" v-if="splitView">
              <q-date
                class="col-6"
                style="width:270px"
                flat
                square
                v-model="proxydatetime"
                :mask="format"
                @input="slide='time'"
                :color="color"
                :options="checkDay"
                no-unset
              />
              <q-time
                class="col-6"
                style="width:270px"
                flat
                square
                v-model="proxydatetime"
                :mask="format"
                format24h
                :with-seconds="withSeconds"
                :color="color"
                now-btn
                @input="log"
                @click="log"
              />
              <!--
                @click.prevent.capture
               -->
            </div>
            <q-carousel
              v-model="slide"
              transition-prev="slide-right"
              transition-next="slide-left"
              swipeable
              animated
              :class="`fit bg-transparent ${splitView?'lt-sm' : ''}`"
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
                  now-btn
                  @input="log"
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
              :class="splitView?'lt-sm' : ''"
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
import { date, debounce } from 'quasar'


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
    // z-index for popups
    popupZindex: {
      type: Number,
      default: 6000
    },
    // allow to edit time by input
    editableInput: {
      type: Boolean,
      default: false
    },
    // delay to apply value by manual input -1 = disabled
    applyInputDelay: {
      type: Number,
      default: 2000
    },
    // split dialog view for large screens (calendar|time)
    splitView: {
      type: Boolean,
      default: false
    },

    translations: {
      type: Object,
      default: function () {
        return {
          __wrong_format_title: 'Wrong date format!',
          __wrong_format_message: 'Do you want to fix the date and time or set the previous ones?',
          __wrong_format_ok: 'Fix',
          __wrong_format_cancel: 'Previous'
         }
      }
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
      slide: 'date',
      applyInput: () => {},
      whatToDo: false
    }
  },
  watch: {
    datetime (val, old) {
      if (val !== old) {
        this.parseDate(val)
      }
    }
  },
  mounted () {
    let that = this
    if (this.applyInputDelay > -1) {
      this.applyInput = debounce(function () {
        that.parseDate(that.$refs.input.$refs.input.value)
      }, this.applyInputDelay)
    }
  },
  methods: {
    log: console.log,
    parseDate(val) {
        let that = this
        let d = Date.parse(val)
        if (!Number.isNaN(d)) {
          this.datetime = date.formatDate(new Date(d), this.format)
          this.$emit('input', parseInt(date.formatDate(date.extractDate(this.datetime, this.format).getTime(), 'x')))
        } else {
          if (!this.whatToDo && this.$q.dialog) {
            this.whatToDo = true
            this.$q.dialog({
              title: this.translations.__wrong_format_title,
              message: this.translations.__wrong_format_message,
              ok: this.translations.__wrong_format_ok,
              cancel: this.translations.__wrong_format_cancel,
              contentStyle: `z-index:${this.popupZindex}`
            }).onOk(() => {
            }).onCancel(() => {
              this.datetime = date.formatDate(this.value, this.format)
            }).onDismiss(()=>{
              setTimeout(function () { that.$refs.input.focus() }, 100)
              this.whatToDo = false
            })
          }
        }
    },
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
