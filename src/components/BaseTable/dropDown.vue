<script>
export default {
  name: 'DropDown',
  props: {
    visible: {
      type: Boolean,
      default: false
    },
    placement: {
      type: String,
      default: 'right'
    },
    offsetLeft: {
      type: Number,
      default: 0
    },
    boundariesElement: {
      type: null
    },
    containerStyle: {
      type: Object,
      default: () => ({})
    }
  },
  data() {
    return {
      offsetX: 0,
      minHeight: 90
    };
  },
  watch: {
    visible(val) {
      if (!val) return;
      this.$nextTick(() => this.calcPanelOffset());
    }
  },
  methods: {
    getElementWidth(el) {
      return el ? el.offsetWidth : 0;
    },
    calcPanelOffset() {
      const res = this.offsetLeft + this.getElementWidth(this.$refs['panel']) - this.getElementWidth(this.boundariesElement);
      if (res > 0) {
        this.offsetX = res + 2;
      } else {
        this.offsetX = 0;
      }
    },
    calcPanelMaxHeight() {
      if (!this.boundariesElement) {
        return this.minHeight;
      }
      const res = this.boundariesElement.querySelector('.el-table').offsetHeight - 42;
      return res > this.minHeight ? res : this.minHeight;
    }
  },
  render() {
    const { $slots, visible, minHeight, offsetX, placement, containerStyle } = this;
    const boxStyle = {
      ...containerStyle,
      [placement]: 0,
      marginLeft: `${-1 * offsetX}px`,
      minHeight: `${minHeight}px`,
      maxHeight: `${this.calcPanelMaxHeight()}px`
    };
    return (
      <div class="wrapper">
        {$slots['reference']}
        <transition name="el-zoom-in-top">
          <div ref="panel" v-show={visible} class="content" style={boxStyle} onClick={ev => ev.stopPropagation()} onMousedown={ev => ev.stopPropagation()}>
            {$slots['content']}
          </div>
        </transition>
      </div>
    );
  }
};
</script>

<style lang="less" scoped>
.wrapper {
  display: inline-flex;
  align-items: center;
  vertical-align: middle;
  position: relative;
  height: 34px;
  padding: 0;
  overflow: visible;
  z-index: 9;
  .content {
    display: flex;
    flex-direction: column;
    position: absolute;
    top: 34px;
    background-color: rgba(255, 255, 255, 1);
    border: 1px solid @borderColorSecondary;
    border-radius: @borderRadius;
    box-shadow: @boxShadow;
    cursor: default;
  }
}
</style>
