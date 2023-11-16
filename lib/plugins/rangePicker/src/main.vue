<script>
export default {
  name: 'JoyinRangePicker',
  props: {
    value: [],
    // 日期格式
    format: {
      type: String,
      default: 'YYYY-MM-DD'
    },
    // 清除开始日期
    clearStart: {
      type: Boolean,
      default: true
    },
    // 清除结束日期
    clearEnd: {
      type: Boolean,
      default: true
    },
    // 清除所有
    clearAll: {
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
        if (!Array.isArray(nVal)) {
          this.rangeDate = [];
          return;
        }
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
    onClearByIndex(i) {
      let changeDate = [];
      if (!this.clearAll) {
        const [s, e] = this.rangeDate;
        changeDate = i === 0 ? ['', e] : [s, ''];
      }
      this.rangeDate = changeDate;
      this.$emit('input', changeDate);
      this.$emit('change', changeDate);
    },

    onDateChange(val) {
      this.isOpen = false;
      const [s, e] = val;
      this.$emit('input', [s, e]);
      this.$emit('change', val);
    },

    onContainerClick() {
      this.isOpen = true;
    },

    renderItem(h, i) {
      const spanEl = h(
        'span', 
        { class: !this.rangeDate[i] && 'placeholder'}, 
        this.rangeDate[i] ? this.rangeDate[i] : this.placeholder[i]
      );
      const iconEl = this.clearStart && this.rangeDate[i] && 
        h('a-icon', 
          { 
            attrs: {
              type: 'close-circle'
            },
            on: {
              click: e => {
                e && e.stopPropagation();
                this.onClearByIndex(i);
              }
            }
          });
      return h('div', null, [spanEl, this.clearAll && !i ? '' : iconEl]);
    },

    renderMain(h) {
      return h('div', 
        {
          class: 'main-container',
          on: {
            click: e => {
              e && e.stopPropagation();
              this.onContainerClick();
            }
          }
        }, 
        [this.renderItem(h, 0), h('i',null,'~'), this.renderItem(h, 1)]);
    },

    renderDate(h) {
      return h('a-range-picker',
      {
        attrs: {
          ...this.$attrs,
          value: this.value,
          fomat: this.format,
          valueFormat: this.format,
          open: this.isOpen
        },
        on: {
          change: e => {
            this.onDateChange(e);
          },
          ok: e => {
            this.$emit('ok', e);
          }
        }
      })
    }
  },
  render(h) {
    return h('div', { class: 'joyin-range-picker-container' }, [this.renderMain(h), this.renderDate(h)]);
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
      color: #242933;
      >div{
        >span {
          margin-right: 5px;
          user-select: none;
        }
        >i {
          display: none;
          color: #bbbbbb;
          &:hover {
            color: #242933;
          }
        }
        .placeholder{
          color: #bbbbbb;
        }
      }
      &:hover {
        i {
          display: inline !important;
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