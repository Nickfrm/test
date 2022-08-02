<template>
  <div class="bottom-sheet">
    <transition name="fade">
      <div
        v-if="yPositionCurrent < yPositionSemiClosed"
        class="bottom-sheet__overlay"
        @click="setPosition(yPositionClosed)"
      ></div>
    </transition>
    <div
      ref="card"
      class="bottom-sheet__card"
      :class="{ transitioned: !isMoving }"
      :style="{ top: `${yPositionCurrent}px` }"
    >
      <div ref="visible-area">
        <div
          class="bottom-sheet__pan-area pan-area"
          @touchstart="(e) => handleTouchStart(e)"
          @touchmove="(e) => handleTouchMove(e)"
          @touchend="() => handleTouchEnd()"
        >
          <div class="pan-area__bar"></div>
        </div>
        <input
          @focus="setPosition(yPositionOpened)"
          class="bottom-sheet__input"
          type="text"
          v-model="query"
          placeholder="Search..."
        />
      </div>
      <div class="bottom-sheet__content">
        <slot />
      </div>
    </div>
  </div>
</template>

<script>
export const BS_CLOSED_STATE = "closed";
export const BS_OPENED_STATE = "opened";
export const BS_SEMICLOSED_STATE = "semi-closed";

export default {
  props: {
    state: {
      type: String,
      default: "",
    },
  },
  data() {
    return {
      isMoving: false,
      yPositionStart: null,
      yPositionCurrent: null,
      height: null,
      visibleAreaHeight: null,
      query: "",
    };
  },
  mounted() {
    window.onresize = () => {
      this.height = this.$refs.card.getBoundingClientRect().height;
      this.setPosition(this.yPositionClosed);
    };
    this.height = this.$refs.card.getBoundingClientRect().height;

    this.visibleAreaHeight =
      this.$refs["visible-area"].getBoundingClientRect().height;

    this.yPositionCurrent = this.yPositionClosed;
  },
  watch: {
    state() {
      this.setPosition(this.yPositionByState);
    },
  },
  computed: {
    yPositionSemiClosed() {
      return (this.height + this.yPositionOpened) * 0.5;
    },
    yPositionClosed() {
      return this.height + this.yPositionOpened - this.visibleAreaHeight;
    },
    yPositionOpened() {
      return 100;
    },
    yPositionByState() {
      switch (this.state) {
        case BS_OPENED_STATE:
          return this.yPositionOpened;
        case BS_SEMICLOSED_STATE:
          return this.yPositionSemiClosed;
        case BS_CLOSED_STATE:
          return this.yPositionClosed;
        default:
          return this.yPositionClosed;
      }
    },
    currentStateByPosition() {
      switch (this.yPositionCurrent) {
        case this.yPositionOpened:
          return BS_OPENED_STATE;
        case this.yPositionSemiClosed:
          return BS_SEMICLOSED_STATE;
        case this.yPositionClosed:
          return BS_CLOSED_STATE;
        default:
          return "";
      }
    },
  },
  beforeDestroy() {
    window.onresize = null;
  },
  methods: {
    handleTouchStart(e) {
      this.isMoving = true;
      this.yPositionStart = e.targetTouches[0].clientY;
    },
    handleTouchMove(e) {
      if (
        e.targetTouches[0].clientY <= this.yPositionOpened ||
        e.targetTouches[0].clientY >= this.yPositionClosed
      )
        return;
      this.yPositionCurrent = e.targetTouches[0].clientY;
    },
    handleTouchEnd() {
      this.isMoving = false;
      const positions = [
        this.yPositionOpened,
        this.yPositionClosed,
        this.yPositionSemiClosed,
      ];
      const closest = positions.reduce((prev, curr) => {
        return Math.abs(curr - this.yPositionCurrent) <
          Math.abs(prev - this.yPositionCurrent)
          ? curr
          : prev;
      });
      this.setPosition(closest);
    },
    setPosition(position) {
      this.yPositionCurrent = position;
      this.$emit("change-state-position", this.currentStateByPosition);
    },
  },
};
</script>

<style lang="scss">
.bottom-sheet {
  .bottom-sheet__overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.25);
  }
  &__card {
    width: 100%;
    height: calc(100vh - 100px);
    position: fixed;
    background: #fff;
    box-shadow: 0 -3px 4px rgba(0, 0, 0, 0.1);
    &.transitioned {
      transition: top 0.3s ease-out;
    }
  }
  &__pan-area {
    padding: 8px 0 16px;
    position: relative;
    .pan-area__bar {
      background-color: $accent-color;
      width: 48px;
      height: 4px;
      border-radius: 4px;
      margin: 0 auto;
    }
    &:after {
      content: "";
      height: 40px;
      position: absolute;
      left: 0;
      right: 0;
      top: 0;
    }
  }
  &__input {
    line-height: normal;
    margin: 0 16px 16px;
    height: 32px;
    width: 90%;
    padding: 0 8px;
    border-radius: 8px;
    border: 1px solid #ccc;
  }
  &__content {
    height: calc(100vh - 178px);
  }
}
</style>
