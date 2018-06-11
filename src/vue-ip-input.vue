<template>
    <form class='form-inline' >
        <span class="form-group" v-for="segment in segments" track-by="$index">
            <label class="sr-only">{{segment}}</label>
            <input type="text" maxlength="3" class="form-control" style='width:50px' :value="segment"
                v-on:keydown="onInputKeydown($event, $index)"
                v-on:input="onInput($event, $index)"
                v-on:blur="onInputBlur()"
                v-on:paste="onPaste($event, $index)">
            <i v-show="$index != segments.length - 1">.</i>
        </span>
        <select class="mask-list" v-model="mask" v-if="withMask">
            <option v-for="m in masks" :value="m">/ {{ m }}</option>
        </select>
    </form>
</template>

<script>
/* global document*/
/**
 * get the cursor position of the element
 * @param  {Element} el the element
 * @return {Integer}    the position fo the cursor
 */
function getRange(el) {
    var cuRange;
    var tbRange;
    var headRange;
    var range;
    var dupRange;
    var ret = {};
    if (el.setSelectionRange) {
        // standard
        ret.begin = el.selectionStart;
        ret.end = el.selectionEnd;
        ret.result = el.value.substring(ret.begin, ret.end);
    } else if (document.selection) {
        // ie
        if (el.tagName.toLowerCase() === 'input') {
            cuRange = document.selection.createRange();
            tbRange = el.createTextRange();
            tbRange.collapse(true);
            tbRange.select();
            headRange = document.selection.createRange();
            headRange.setEndPoint('EndToEnd', cuRange);
            ret.begin = headRange.text.length - cuRange.text.length;
            ret.end = headRange.text.length;
            ret.result = cuRange.text;
            cuRange.select();
        } else if (el.tagName.toLowerCase() === 'textarea') {
            range = document.selection.createRange();
            dupRange = range.duplicate();
            dupRange.moveToElementText(el);
            dupRange.setEndPoint('EndToEnd', range);
            ret.begin = dupRange.text.length - range.text.length;
            ret.end = dupRange.text.length;
            ret.result = range.text;
        }
    }
    el.focus();
    return ret;
}
export default {
    props: {
        ip: {
            type: String,
            required: true
        },
        withMask: Boolean,
        onChange: Function,
        onBlur: Function
    },
    data() {
        return {
            segments: ['', '', '', ''],
            mask: '32',
            masks: ['32', '31', '30', '29', '28', '27', '26', '25', '24', '23', '22', '21', '20', '19', '18', '17', '16', '15', '14', '13', '12', '11', '10', '9', '8', '7', '6', '5', '4', '3', '2', '1']
        };
    },
    watch:{
        ip: function(){ this.onReady() },
        mask: function(){
            this.ip = this.segments.join('.') + (this.withMask ? `/${this.mask}`:'');
        }
    },
    methods: {
        onInputKeydown(event, index) {
            var keyCode = event.keyCode || event.which;
            var value = event.target.value;
            if (keyCode === 8 || keyCode === 37) {
                // move the cursor to previous input if backspace and left arrow is pressed at the begin of one input
                if ((value.length === 0 || getRange(event.target).end === 0) &&
                    index > 0) {
                    this.$el.getElementsByTagName('input')[index - 1].focus();
                }
            } else if (keyCode === 39) {
                if (getRange(event.target).end === value.length &&
                    index < 3) {
                    // move to cursor to the next input if right arrow is pressed at the end of one input
                    this.$el.getElementsByTagName('input')[index + 1].focus();
                }
            }
        },
        onInput(event, index) {
            var value = event.target.value;
            event.target.value = this.segments[index];
            var segment = Number(value);
            if (isNaN(segment)) {
                return;
            } else if (segment > 255 || segment < 0) {
                // set the segment to 255 if out of ip range
                this.segments.$set(index, 255);
            } else if (segment === 0) {
                this.segments.$set(index, '');
            } else {
                this.segments.$set(index, segment);
            }
            // jump to next input
            if (value.length === 3 && index < 3) {
                this.$el.getElementsByTagName('input')[index + 1].focus();
            }
        },
        onInputBlur() {
            setTimeout(() => {
                var className = document.activeElement.className;
                if (className.indexOf('ip-segment-input') === -1) {
                    if (this.onBlur) {
                        this.onBlur(this.segments.join('.'));
                    }
                }
            }, 50);
        },
        onPaste(e, index) {
            var pasteText = e.clipboardData.getData('text/plain');
            var segments = pasteText.split('.');
            segments.forEach((segment, i) => {
                var value = Number(segment);
                if (index + i < 4 && !isNaN(value) &&
                value >= 0 && value <= 255) {
                    this.segments.$set(index + i, value);
                }
            });
            e.preventDefault();
        },
        onReady() {
            var ip = this.ip;
            this.mask = '32';
            if (this.withMask) {
                if (ip.indexOf('/') !== -1) {
                    let exp = this.ip.split('/');
                    this.mask = exp[1];
                    ip = exp[0];
                }
            }else if(ip.indexOf('/') !== -1) {
                let exp = this.ip.split('/');
                ip = exp[0];
            }
            if (ip && ip.indexOf('.') !== -1) {
                ip.split('.').map((segment, index) => {
                    segment = Number(segment);
                    if (isNaN(segment) || segment < 0 || segment > 255) {
                        segment = 255;
                    }
                    this.segments.$set(index, segment);
                    return segment;
                });
            }
            this.$watch(() => {
                 return this.segments.join('.') + (this.withMask ? `/${this.mask}`:'');
            }, (val, oldValue) => {
                if (val !== oldValue) {
                    if (val === '...') {
                        val = '';
                    }
                    this.ip = val;
                    if (this.onChange) {
                        this.onChange(val);
                    }
                }
            });
        }
    },
    ready() {
        this.onReady();
    }
};
</script>

<style scoped>
.form-inline {
    border: 1px solid #ccc;
    border-radius: 4px;
}  
.form-control {
    border: none;
}
@media (max-width: 767px) {
    .form-inline .form-control {
        display: inline-table;
        width: auto;
        vertical-align: middle;
    }
}
@media (max-width: 767px) {
    .form-inline .form-group {
        display: inline-table;
        margin-bottom: 0;
        vertical-align: middle;
    }
}
.mask-list{
    width: 50px;
    height: 26px;
    line-height: normal;
    border: none;
    outline: none;
    text-align: center;
    text-indent: 0px;
    margin: 0px;
    padding: 0px;
    background-color: transparent;
}
.mask-slash, .mask-list{
    display: inline-block;
}
</style>
