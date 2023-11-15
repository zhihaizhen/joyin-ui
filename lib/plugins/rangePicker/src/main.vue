/* eslint-disable */
<template>
  <div class="joyin-range-picker-container">
    <div class="main-container" @click.stop="onContainerClick">
      <div>
        <span>{{ rangeDate[0] ? rangeDate[0] : placeholder[0] }}</span>
        <a-icon v-if="isClearStart && rangeDate[0]" type="close-circle" @click.stop="onClearStart"/>
      </div>
      <i>-</i>
      <div>
        <span>{{ rangeDate[1] ? rangeDate[1] : placeholder[1] }}</span>
        <a-icon v-if="isClearEnd && rangeDate[1]" type="close-circle" @click.stop="onClearEnd" />
      </div> 
    </div>
    <a-range-picker
      v-model="rangeDate"
      v-bind="$attrs"
      :open="isOpen"
      :format="format"
      :valueFormat="format"
      @change="onDateChange"
    >
    </a-range-picker>
  </div>
</template>
<script>
export default {
  name: 'JoyinRangePicker',
  props: {
    value: [],
    format: {
      type: String,
      default: 'YYYY-MM-DD'
    },
    isClearStart: {
      type: Boolean,
      default: true
    },
    isClearEnd: {
      type: Boolean,
      default: true
    },
    // 提示文字
    placeholder: {
      type: Array,
      default: () => {
        return ['开始日期', '结束日期']
      }
    }
  },
  data() {
    return {
      isOpen: false,
      rangeDate: []
    }
  },
  watch: {
    value: {
      handler(nVal) {
        const [s, e] = nVal;
        const startDate = typeof s === 'object' && s.isValid() ? s.format(this.format) : s;
        const endDate = typeof e === 'object' && e.isValid() ? e.format(this.format) : e;
        this.rangeDate = [startDate, endDate];
      },
      immediate: true
    }
  },
  created() {
    const evt = e => {
      if (e.target.className && e.target.className.indexOf('ant-calender')) return;
      this.isOpen = false;
    }
    window.addEventListener('click', evt);
    this.$once('hook:beforeDestroy', function() {
      window.removeEventListener('click', evt);
    })
  },
  methods: {
    onClearStart() {
      const [, e] = this.rangeDate;
      this.rangeDate = ['', e];
      this.$emit('input', ['', e]);
      this.$emit('change', ['', e]);
    },
    onClearEnd() {
      const [s] = this.rangeDate;
      this.rangeDate = [s, ''];
      this.$emit('input', [s, '']);
      this.$emit('change', [s, '']);
    },
    onDateChange(val) {
      this.isOpen = false;
      const [s, e] = val;
      this.$emit('input', [s, e]);
      this.$emit('change', val);
    },
    onContainerClick() {
      this.isOpen = true;
    }
  }
}
</script>

<style lang="less" scoped>
  .joyin-range-picker-container {
    position: relative;
    width: 100%;
    min-width: 216px;
    height: 32px;
    background-color: #fff;
    background-image: none;
    border: 1px solid #d9d9d9;
    border-radius: 4px;
    .main-container {
      width: 100%;
      height: 100%;
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 0 10px;
      >div{
        >span {
          margin-right: 5px;
          user-select: none;
        }
        >i {
          display: none;
        }
      }
      &:hover {
        i {
          display: inline;
        }
      }
    }
    /deep/ .ant-calendar-picker {
      visibility: hidden;
      position: absolute;
      top: 0px;
      left: 0px;
      z-index: -1;
    }
  }
</style>