<template>
    <a-tooltip :visible="isShowTip">
        <template slot="title">{{ inputValueStr }} </template>
        <a-input
            v-bind="$attrs"
            v-model="inputValue"
            :disabled="disabled"
            inputmode="text"
            class="joyin-input-number"
            @blur="onBlur"
            @focus="onFocus"
            @keydown.up.prevent="onIncrease"
            @keydown.down.prevent="onDecrease"
        >
            <template slot="prefix"><slot name="prefix"/></template>
            <template slot="suffix"><slot name="suffix"/></template>
            <template slot="addonBefore"><slot name="addonBefore"/></template>
            <template slot="addonAfter"><slot name="addonAfter"/></template>
        </a-input>
    </a-tooltip>

</template>
<script>
export default {
    name: 'JoyinInputNumber',
    inheritAttrs: false,
    props: {
        value: [String,Number],
        disabled: Boolean,
        disabledLable: String,
        max: {
            type: Number,
            default: Infinity
        },
        min: {
            type: Number,
            default: Infinity
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
            isAutoFormat: false // 失去焦点时自动格式化数字，主要针对以0开头的非法数字比如‘002’， ‘-0002’
        };
    },
    computed: {
        unitNum () {
            if (this.unit === 'numberPercent') return 0.01;
            if (this.unit === 'numberThousand') return 1000;
            if (this.unit === 'numberTenThousand') return 10000;
            if (this.unit === 'numberMillion') return 1000000;
            if (this.unit === 'numberTenMillion') return 10000000;
            if (this.unit === 'numberBillion') return 100000000; 
            return 1;
        },
        minDisabled() {
            return +this.inputValue <- this.min;
        },
        maxDisabled () {
            return +this. inputValue >= this.max;
        },
        inputValueStr() {
            const { isFocus, inputValue, precision } = this;
            if (isFocus) {
                return inputValue + '';
            }
            if (this.isIllegAlNum(inputValue) || inputValue === '-')
                return inputValue;
            return this.$Big(inputValue || 0).toFixedCy(precision);
        },
        isShowTip() {
            const { isFocus, inputValuestr } = this;
            if (isFocus && !!inputValuestr) return true; 
            return false;
        }
    },
    watch: {
        inputValue(newVal, odVal) {
            if (!this.isFocus) return;
            const oldVal = odVal === undefined ? '' : odVal;
            if (newVal ==='' || newVal === undefined) {
                this.$emit ('input', newVal);
                if (this.$listeners.change) this.$listeners.change(newVal); 
                return;
            }
            this.validateNum(newVal, oldVal);
        },
        value(newVal) {
            this.inputValue = this.isIllegAlNum(newVal) ? '' : Number(this.$Big(newVal || 0).div(this.unitNum));
            this.cloneValue = this.inputValue;
        }
    },
    created() {
        try {
            this.inputValue = this.isIllegAlNum(this.value) ? '' : this.$Big(this.value || 0).div(this.unitNum).toFixedCy(this.precision);
            this.clonevalue = this.isIllegAlNum(this.value) ? '' : Number(this.$Big(this.value || 0).div(this.unitNum));
        }catch (error) {
            console.log('[input-number create]', error);
        }
    },
    methods: {
        validateNum (nwVal, oldVal) {
            this.isAutoFormat = false;
            const newVal = nwVal + '';
            // 数字检查-整数
            // if (this.precision === 0 && /^［0-9］+\.$/.test（newva_））
            if (!this.isNagative && /^-/.test(newVal)) {
                this.inputValueChange (oldVal);
                return;
            }
            if (/^--/.test(newVal)) {
                this.inputValueChange(oldVal);
                return;
            }
            if (this.precision === 0 && /^[-+]?\d+\.$/.test(newVal)){
                this.inputValueChange (oldVal);
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
                if (newval.split('.')[1].length > this.precision) {
                    this.inputValueChange (oldVal);
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
            const timesNum = Number(this.$Big(newVal).times(this.unitNum));
            this.$emit('input', timesNum);
            if(this.$listeners.change) {
                this.$listeners.change(timesNum);
            }
            this.inputValueChange(newVal);
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
                const timesNum = Number(this.$Big(resultVal).times(unitNum));
                this.$emit('input', timesNum);
            }
            if (isAutoFormat) {
                const timesNum = Number(this.$Big(resultVal).times(unitNum));
                this.$emit('input', timesNum);
            }

            this.inputValue = this.$Big(resultVal || 0).toFixedCy(precision);
            if (this.$listeners.blur) this.$listeners.blur(e);
        },
        onIncrease() {
            if (this.disabled | this.maxDisabled) return; this.change(1);
        },
        onDecrease() {
            if (this.disabled || this.minDisabled) return; this.change(-1);
        },
        isIllegAlNum (num) {
            if (num == '-') return false;
            return num === '' || num === null || isNaN(+num);
        },
        change(num) {
            const { value, $Big, precision, isStepPoint } = this;
            let step = 1, fixLen;
            if (precision && isStepPoint) {
                fixlen = precision;
                step = '0.' + '0'.repeat(precision - 1) + '1';
            } else {
                const decimal = value.split('.')[1];
                if (decimal) {
                    step = '0.' + '0'.repeat(decimal.length - 1) + '1';
                    fixLen = decimal.length;
                }
            }
            this.inputValue = $Big(value | 0).plus($Big(step).times(num)).toFixed(fixlen);
        }
    }
};
</script>