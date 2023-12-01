<script>
export default {
    name: 'JoyinInput',
    props: {
        value: [Number, String],
        maxLength: {
            type: Number,
            default: 1000
        }
    },
    data() {
        return {
            inputVal: ''
        }
    },
    watch: {
        value: {
            handler(nVal) {
                this.inputVal = nVal;
            },
            immediate: true
        }
    },
    render(h) {
        const self = this;
        const attr = {
            class: 'joyin-input',
            attrs: {
                ...self.$attrs,
                value: self.inputVal,
                maxLength: self.maxLength
            },
            on: {
                ...self.$listeners,
                input: e => {
                    self.$emit('input', e.target.value);
                },
                blur: e => {
                    // const value = e.target.value;
                    // self.$emit('input', value.trim());
                    self.$emit('blur', e);
                },
                change: e => {
                    self.$emit('change', e);
                },
                pressEnter: e => {
                    self.$emit('pressEnter', e);
                }
            }
        };
        const {prefix, suffix, addonBefore, addonAfter} = self.$slots;
        return h('a-input', attr, [
            prefix ? h('template', {slot: 'prefix'}, [prefix]) : null,
            suffix ? h('template', {slot: 'suffix'}, [suffix]) : null,
            addonBefore ? h('template', {slot: 'addonBefore'}, [addonBefore]) : null,
            addonAfter ? h('template', {slot: 'addonAfter'}, [addonAfter]) : null
        ]);
    }
}
</script>
