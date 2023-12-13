<script>
/**
 * 数字输入框 create by zhz
 */
export default {
    name: 'JoyinInputNumber',
    inheritAttrs: false,
    props: {
        value: [String,Number],
        disabled: Boolean,
        disabledLable: String,
        // 是否开启气泡提示
        isAbelTip: {
            type: Boolean,
            default: true
        },
        max: {
            type: Number,
            default: Infinity
        },
        min: {
            type: Number,
            default: -Infinity
        },
        precision: {
            type: Number,
            default: undefined
        },
        // 单步是否是piont
        isStepPoint: Boolean,
        // 是否允许输入负数
        isNagative: {
            type: Boolean,
            default:false
        },
        /**
         * 数字单位
         * number:无单位
         * numberPercent;百分比
         * numberThousand: 千
         * numberTenThousand:万
         * numberMillion:百万
         * numberTenMillion:千万
         * numberBillion: 亿
         */
        unit: {
            type: String,
            default: 'number'
        }
    },
    data() {
        return {
            isFocus: false,
            cloneValue: '',
            inputValue: '',
            inputValueStr: '',
            isAutoFormat: false // 失去焦点时自动格式化数字，主要针对以0开头的非法数字比如‘002’， ‘-0002’
        };
    },
    computed: {
        unitNum() {
            let num = 1;
            switch (this.unit) {
                case 'numberPercent':
                    num = 0.01;
                    break;
                case 'numberThousand':
                    num = 1000;
                    break;
                case 'numberTenThousand':
                    num = 10000;
                    break;
                case 'numberMillion':
                    num = 1000000;
                    break;
                case 'numberTenMillion':
                    num = 10000000;
                    break;
                case 'numberBillion':
                    num = 100000000;
                    break;
                default:
                    num = 1;
                    break;
            }
            return num;
        },
        minDisabled() {
            return +this.inputValue <= this.min;
        },
        maxDisabled () {
            return +this.inputValue >= this.max;
        },
        isShowTip() {
            return this.isFocus && !!this.inputValueStr; 
        } 
    },
    watch: {
        inputValue(newVal, odVal) {
            if (!this.isFocus) return;
            const oldVal = odVal === undefined ? '' : odVal;
            if (newVal ==='' || newVal === undefined) {
                this.$emit('input', newVal);
                this.debounce(this.updateInputValueStr(), 800);
                if (this.$listeners.change) this.$listeners.change(newVal); 
                return;
            }
            this.debounce(this.validateNum(newVal, oldVal), 300);
            this.debounce(this.updateInputValueStr(), 800);
        },
        value(newVal) {
            this.inputValue = this.isIllegAlNum(newVal) ? '' : newVal === '-' ? '-' : Number(this.$Big(newVal).div(this.unitNum));
            this.cloneValue = this.inputValue;
        }
    },
    created() {
        try {
            this.inputValue = this.isIllegAlNum(this.value) ? '' : this.$Big(this.value).div(this.unitNum).toFixedCy(this.precision);
            this.cloneValue = this.isIllegAlNum(this.value) ? '' : Number(this.$Big(this.value).div(this.unitNum));
        }catch (error) {
            console.log('[input-number create]', error);
        }
    },
    render(h) {
        return this.isAbelTip ? this.renderInputInToolTip(h) : this.renderInput(h);
    },
    methods: {
        renderInputInToolTip(h) {
            return h('a-tooltip', {
                attrs: {
                    visible: this.isShowTip
                }
            }, [h('template', {slot: 'title'}, this.inputValueStr), this.renderInput(h)]);
        },
        renderInput(h) {
            const {prefix, suffix, addonBefore, addonAfter} = this.$slots;
            return h('a-input', 
            {
                attrs: {
                    ...this.$attrs,
                    class: 'joyin-input-number',
                    inputmode: 'text',
                    value: this.inputValue,
                    disabled: this.disabled,
                },
                on: {
                    input: e => {
                        this.inputValue = e.target.value;
                    },
                    blur: e => {
                        this.onBlur(e);
                    },
                    focus: e => {
                        this.onFocus(e);
                    },
                    keydown: e => {
                        this.onKeyDown(e);
                    }
                }
            },[
                prefix ? h('template', {slot: 'prefix'}, [prefix]) : null,
                suffix ? h('template', {slot: 'suffix'}, [suffix]) : null,
                addonBefore ? h('template', {slot: 'addonBefore'}, [addonBefore]) : null,
                addonAfter ? h('template', {slot: 'addonAfter'}, [addonAfter]) : null
            ]);
        },
        debounce(func, delay) {
            let timer = null;
            return function (...args) {
                timer && clearTimeout(timer);
                timer = setTimeout(() => {
                    func && func.call(this, ...args);
                }, delay);
            }
        },
        updateInputValueStr() {
            const { isFocus, inputValue, precision } = this;
            if (!isFocus) {
                this.inputValueStr =  inputValue + '';
                return;
            }
            if (this.isIllegAlNum(inputValue) || inputValue === '-'){
                this.inputValueStr = inputValue;
                return;
            }
            
            this.inputValueStr = this.$Big(inputValue).toFixedCy(precision);
        },
        validateNum(nwVal, oldVal) {
            this.isAutoFormat = false;
            const newVal = Object.is(nwVal, -0) ? '-0' : nwVal + '';
            // 数字检查-整数
            // if (this.precision === 0 && /^［0-9］+\.$/.test（newva_））
            if (!this.isNagative && /^-/.test(newVal)) {
                this.inputValueChange(oldVal);
                return;
            }
            if (/^--/.test(newVal)) {
                this.inputValueChange(oldVal);
                return;
            }
            if (this.precision === 0 && /^[-+]?\d+\.$/.test(newVal)){
                this.inputValueChange(oldVal);
                return;
            }
            // 数字检查—小数
            if (!/^[-+]?\d*\.?[0-9]*$/.test(newVal)) {
                this.inputValueChange(oldVal); 
                return;
            }
            // 如果0开头
            if (/^0/.test(newVal) && !/^0(\.|$)/.test(newVal)){
                // this.inputValueChange (oldVal);
                this.isAutoFormat = true;
                return;
            }
            // 如果-0开头
            if (/^-0/.test(newVal) && !/^-0(\.|$)/.test(newVal)){
                // this.inputValueChange (oldVal);
                this.isAutoFormat = true;
                return;
            }
            // 判断有效位数
            if (typeof this.precision === 'number' && /\./.test(newVal)) {
                if (newVal.split('.')[1].length > this.precision) {
                    this.inputValueChange(oldVal);
                    return; 
                }
            }
            // 判断数字是否在有效区间内
            const numValue = Number(newVal);
            const { max,min } = this;
            if (numValue > max) {
                this.inputValueChange(max + ''); 
                return;
            }
            if (numValue < min) {
                // this.inputValueChange(min + '');
                return;
            }
            const timesNum = newVal === '-' ? '-' : Number(this.$Big(newVal).times(this.unitNum));
            if (Object.is(timesNum, -0)) {
                this.isAutoFormat = true;
            } else {
                this.$emit('input', timesNum);
            }
            
            if(this.$listeners.change) {
                this.$listeners.change(timesNum);
            }
            // this.inputValueChange(newVal);
            this.cloneValue = newVal;
        },
        inputValueChange (val) {
            this.inputValue = val;
            this.cloneValue = val;
        },
        onFocus () {
            this.isFocus = true;
            this.inputValue = this.cloneValue;
        },
        onBlur(e) {
            this.isFocus = false;
            const { inputValue, precision, min, unitNum, isAutoFormat } = this;
            this.cloneValue = inputValue;
            if (this.isIllegAlNum(inputValue)) return;
            let resultVal = inputValue;
            if (+inputValue < min) {
                resultVal = min;
                const timesNum = resultVal === '-' ? '-' : Number(this.$Big(resultVal).times(unitNum));
                this.$emit('input', timesNum);
            }
            if (isAutoFormat) {
                const timesNum = resultVal === '-' ? '-' : Number(this.$Big(resultVal).times(unitNum));
                this.$emit('input', timesNum);
            }

            this.inputValue = resultVal === '-' ? '-' : this.$Big(resultVal).toFixedCy(precision);
            if (this.$listeners.blur) this.$listeners.blur(e);
        },
        onKeyDown(e) {
            if (this.disabled | this.maxDisabled) return;
            const { keyCode } = e;
            if (keyCode === 38 || keyCode === 40) {
                e && e.preventDefault();
                this.change(keyCode === 38 ? 1 : -1);
            }
        },
        isIllegAlNum (num) {
            if (num == '-') return false;
            return num === '' || num === null || isNaN(+num);
        },
        change(num) {
            const { inputValue, $Big, precision, isStepPoint } = this;
            let step = 1;
            let fixLen = 0;
            if (precision && isStepPoint) {
                fixLen = precision;
                step = '0.' + '0'.repeat(precision - 1) + '1';
            } else {
                const decimal = inputValue.split('.')[1];
                if (decimal) {
                    step = '0.' + '0'.repeat(decimal.length - 1) + '1';
                    fixLen = decimal.length;
                }
            }
            this.inputValue = $Big(inputValue | 0).plus($Big(step).times(num)).toFixed(fixLen);
        }
    }
};
</script>