<script>
import { myMixin } from './mixin.js';
export default {
  mixins: [myMixin],
  name: 'AjaxButton',
  props: {
    // 点击执行的方法 必传
    click: {
      type: Function,
      default: null,
      required: true
    },
    size: {
      type: String,
      default: 'small'
    },
    type: {
      type: String
    },
    disabled: {
      type: Boolean,
      default: false
    },
    plain: {
      type: Boolean,
      default: false
    },
    round: {
      type: Boolean,
      default: false
    },
    circle: {
      type: Boolean,
      default: false
    },
    icon: {
      type: String,
      default: null
    },
    autofocus: {
      type: Boolean,
      default: false
    },
    nativeType: {
      type: String,
      default: 'button'
    },
    // 权限校验参数
    authList: {
      type: Array,
      default: null
    },
    authMark: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      ajaxing: false
    };
  },
  computed: {
    // 显示loading
    isDisabled() {
      return this.ajaxing || this.disabled;
    },
    isVisible() {
      // 没有权限控制，默认该按钮显示状态
      if (!this.authList) return true;
      return this.authList.includes(this.authMark);
    }
  },
  methods: {
    async sleep(timeLen) {
      return new Promise(resolve => {
        setTimeout(resolve, timeLen);
      });
    },
    async clickHandle() {
      this.ajaxing = true;
      try {
        await this.click();
      } catch (err) {
        console.log(err);
      }
      await this.sleep(100);
      this.ajaxing = false;
    }
  },
  render() {
    const {
      $props,
      $listeners,
      $attrs,
      $slots,
      ajaxing,
      isDisabled,
      isVisible
    } = this;
    const wrapProps = {
      key: 'ajax-btn',
      props: {
        ...$props,
        loading: ajaxing,
        disabled: isDisabled
      },
      attrs: { ...$attrs },
      on: {
        ...$listeners,
        click: this.clickHandle
      }
    };
    return isVisible ? (
      <el-button {...wrapProps}> {$slots['default']} </el-button>
    ) : null;
  }
};
</script>
