<template>
  <div class="view-slider">
    <div class="view-slider-tabs">
      <div
        v-for="(i, idx) in items"
        :key="i.id"
        class="view-slider-tabs__tab"
        :class="{ active: currentIndex === idx }"
        @click="moveTo(idx)"
      >
        {{ i.title }}
      </div>
    </div>
    <div
      ref="touch-container"
      class="view-slider-touch-container"
      @touchstart="(e) => handleTouchStart(e)"
      @touchmove="(e) => handleTouchMove(e)"
      @touchend="() => handleTouchEnd()"
    >
      <div
        ref="items-container"
        class="view-slider-touch-container__items-container"
        :class="transitionClass"
        :style="{ transform: transformStyle }"
      >
        <div
          v-for="i in items"
          :key="i.id"
          class="view-slider-touch-container__item"
          @touchend="(e) => handleExtremePositions(e)"
        >
          <div class="view-slider-touch-container__content">
            {{ i.content }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      currentIndex: 0,
      maxTranslateX: 0,
      translateXAmount: 0,
      isMoving: false,
      startYPosition: null,
      startXPosition: null,
      currentYPosition: null,
      currentXPosition: null,
      maxDeltaY: 0,
      // mock items
      items: [
        {
          id: 1,
          title: "All",
          content:
            "Lorem ipsum dolor sit, amet consectetur adipisicing elit. Molestias neque, perferendis placeat expedita doloremque incidunt quae fuga recusandae voluptatem eveniet officia, mollitia ullam ex. Nemo, aliquam eos! Adipisci, perferendis nihil cumque voluptatum perspiciatis cum accusamus magnam beatae, autem, animi aut veritatis placeat in nemo nobis necessitatibus minima illum voluptas suscipit! Lorem ipsum dolor sit, amet consectetur adipisicing elit. Molestias neque, perferendis placeat expedita doloremque incidunt quae fuga recusandae voluptatem eveniet officia, mollitia ullam ex. Nemo, aliquam eos! Adipisci, perferendis nihil cumque voluptatum perspiciatis cum accusamus magnam beatae, autem, animi aut veritatis placeat in nemo nobis necessitatibus minima illum voluptas suscipit! Lorem ipsum dolor sit, amet consectetur adipisicing elit. Molestias neque, perferendis placeat expedita doloremque incidunt quae fuga recusandae voluptatem eveniet officia, mollitia ullam ex. Nemo, aliquam eos! Adipisci, perferendis nihil cumque voluptatum perspiciatis cum accusamus magnam beatae, autem, animi aut veritatis placeat in nemo nobis necessitatibus minima illum voluptas suscipit! Lorem ipsum dolor sit, amet consectetur adipisicing elit. Molestias neque, perferendis placeat expedita doloremque incidunt quae fuga recusandae voluptatem eveniet officia, mollitia ullam ex. Nemo, aliquam eos! Adipisci, perferendis nihil cumque voluptatum perspiciatis cum accusamus magnam beatae, autem, animi aut veritatis placeat in nemo nobis necessitatibus minima illum voluptas suscipit!",
        },
        {
          id: 2,
          title: "Places",
          content:
            " adipisicing elit. Molestias neque, perferendis placeat expedita doloremque incidunt quae fuga recusandae voluptatem eveniet officia, mollitia ullam ex. Nemo, aliquam eos! Adipisci, perferendis nihil cumque voluptatum perspiciatis cum accusamus magnam beatae, autem, animi aut veritatis placeat in nemo nobis necessitatibus minima illum voluptas suscipit!",
        },
        {
          id: 3,
          title: "Sales",
          content:
            "voluptatem eveniet officia, mollitia ullam ex. Nemo, aliquam eos! Adipisci, perferendis nihil cumque voluptatum perspiciatis cum accusamus magnam beatae, autem, animi aut veritatis placeat in nemo nobis necessitatibus minima illum voluptas suscipit!",
        },
        {
          id: 4,
          title: "Maps",
          content:
            "voluptatem eveniet officia, mollitia ullam ex. Nemo, aliquam eos! Adipisci, perferendis nihil cumque voluptatum perspiciatis cum accusamus magnam beatae, autem, animi aut veritatis placeat in nemo nobis necessitatibus minima illum voluptas suscipit!",
        },
      ],
    };
  },
  computed: {
    transformStyle() {
      return this.isMoving
        ? this.translateXCurrent
        : `translateX(${-this.currentIndex * 100}vw)`;
    },
    translateXCurrent() {
      return `translateX(calc(${-this.currentIndex * 100}vw + ${
        this.translateXAmount
      }px))`;
    },
    transitionClass() {
      return this.isMoving ? "transition-initial" : "transition-item";
    },
    isNextAvailable() {
      return this.currentIndex < this.items.length - 1;
    },
    isPreviousAvailable() {
      return this.currentIndex > 0;
    },
    deltaX() {
      return this.currentXPosition - this.startXPosition;
    },
    deltaY() {
      return this.currentYPosition - this.startYPosition;
    },
  },

  methods: {
    handleTouchStart(e) {
      this.isMoving = true;
      this.startYPosition = e.targetTouches[0].clientY;
      this.startXPosition = e.targetTouches[0].clientX;
    },
    handleTouchMove(e) {
      const touches = e.targetTouches;
      // Update current position
      this.currentYPosition = touches[touches.length - 1].clientY;
      this.currentXPosition = touches[touches.length - 1].clientX;
      const {
        translateXAmount,
        isPreviousAvailable,
        isNextAvailable,
        deltaX,
        deltaY,
      } = this;

      if (Math.abs(deltaY) > Math.abs(this.maxDeltaY)) {
        this.maxDeltaY = deltaY;
      }
      // Break if move more vertical than horizontal
      if (
        (Math.abs(deltaX) < 16 || Math.abs(deltaY) - Math.abs(deltaX) > -1) &&
        !translateXAmount
      )
        return;

      // Break if item is no available
      if (
        (!isPreviousAvailable && deltaX > 0) ||
        (!isNextAvailable && deltaX < 0)
      )
        return;

      // Move logic
      if (Math.abs(deltaX) > Math.abs(this.maxTranslateX)) {
        this.maxTranslateX = deltaX;
      }
      this.translateXAmount = deltaX;
    },
    handleTouchEnd() {
      const { translateXAmount, maxTranslateX, currentIndex } = this;

      if (Math.abs(translateXAmount) - Math.abs(maxTranslateX) < -1) {
        this.isMoving = false;
        this.resetTranslate();
      } else if (translateXAmount > 0) {
        this.moveTo(currentIndex - 1);
      } else if (translateXAmount < 0) {
        this.moveTo(currentIndex + 1);
      }
    },
    moveTo(idx) {
      if (idx < 0 || idx > this.items.length - 1) return;
      this.currentIndex = idx;
      this.resetTranslate();
    },
    resetTranslate() {
      this.isMoving = false;
      this.maxTranslateX = 0;
      this.maxDeltaY = 0;
      this.translateXAmount = 0;
    },
    handleExtremePositions(e) {
      const current = e.currentTarget;
      const child = e.currentTarget.firstChild;
      const { deltaY, deltaX, maxDeltaY } = this;
      if (Math.abs(deltaY) < 16 || Math.abs(deltaX) - Math.abs(deltaY) > -1)
        return;
      if (Math.abs(deltaY) - Math.abs(maxDeltaY) < -1) {
        this.resetTranslate();
        return;
      }
      if (current.scrollTop === 0 && this.deltaY > 0) {
        this.$emit("swipe-on-top");
      }
      if (
        current.offsetHeight + current.scrollTop === child.offsetHeight &&
        this.deltaY < 0
      ) {
        this.$emit("swipe-on-bottom");
      }
    },
  },
};
</script>

<style lang="scss">
.view-slider {
  height: 100%;
  &-tabs {
    display: flex;
    align-items: center;
    justify-content: space-between;
    &__tab {
      font-size: 20px;
      font-weight: 700;
      color: #232323;
      padding: 12px 20px;
      width: 100%;
      border-bottom: 4px solid #f7f7f7;
      transition: all 0.1s ease-out;
      &.active {
        border-color: $accent-color;
        color: $accent-color;
      }
    }
  }

  &-touch-container {
    height: calc(100% - 51px);

    &__items-container {
      display: flex;
      height: 100%;
      min-height: fit-content;
      width: 100vw;
      box-sizing: border-box;
    }
    &__item {
      height: 100%;
      width: 100%;
      min-width: 100%;
      overflow-y: auto;
    }
    &__content {
      font-size: 20px;
      line-height: 1.5;
      padding: 20px 16px;
      touch-action: pan-y;
      min-height: 100%;
    }
  }
}

// Transitions
.transition-initial {
  transition: transform 0s ease;
}

.transition-item {
  transition: transform 0.25s ease;
}
</style>
