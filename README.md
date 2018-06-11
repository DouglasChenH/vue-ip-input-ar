# vue-ip-input-ar

This is a forked project from [vue-ip-input-ar](https://github.com/pilevar/vue-ip-input-ar). I redesigned the UI using Bootstrap for personal use. Please go to original project https://github.com/pilevar/vue-ip-input-ar if you are interested.

> An ip input implement by vuejs

## Demo

[Demo](https://lakb248.github.io/vue-ip-input)

## Usage

### Install

```bash
npm install vue-ip-input-ar --save
```

And, if you are using vue 2.0

```bash
npm install vue-ip-input-ar@next --save
``` 

### CommonJS

```javascript
var VueIpInput = require('vue-ip-input-ar');

new Vue({
    components: {
        'vue-ip-input': VueIpInput
    },
    data: function () {
        return {
            ip: '127.0.0.1/16'
        };
    },
    methods: {
        onIpChange: function(ip) {
            console.log('ip input change:', ip);
        },
        onIpBlur: function (ip) {
            console.log('ip input blur:', ip);
        }
    },
    template: '<vue-ip-input :ip="ip" :on-change="onIpChange" :on-blur="onIpBlur" :with-mask="true"></vue-ip-input>'
});
```

### ES6
```javascript
import VueIpInput from 'vue-ip-input-ar';

new Vue({
    components: {
        'vue-ip-input': VueIpInput
    },
    data() {
        return {
            ip: '127.0.0.1/16'
        };
    },
    methods: {
        onIpChange(ip) {
            console.log('ip input change:', ip);
        },
        onIpBlur(ip) {
            console.log('ip input blur:', ip);
        }
    },
    template: '<vue-ip-input :ip="ip" :on-change="onIpChange" :on-blur="onIpBlur" :with-mask="true"></vue-ip-input>'
})
```

### Props
| Property | Description |
|:--|:--|
| ip | the value of ip input |
| onChange | trigger when the ip change |
| onBlur | trigger when the input blur |
| withMask | ‌Boolean variable of show select mask of ip |

## License

[MIT](http://opensource.org/licenses/MIT)
