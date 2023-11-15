<template>
  <div style="height: 100%">
    <el-container>
      <el-aside :width="sidebarWidth" class="aside" :style="customAsideStyle">
        <custom-sidebar
          :isCollapse="collapseState"
          :isPhone="isPhone"
        ></custom-sidebar>
      </el-aside>
      <el-container>
        <el-header class="header">
          <div class="left">
            <div class="operate" ref="operate">
              <i
                class="iconfont icon-fold"
                :class="{ rotate: foldState }"
                @click="toggleSidebarState"
              />
              <custom-navbar></custom-navbar>
            </div>
            <el-collapse-transition>
              <custom-reuse-tab ref="reuse"></custom-reuse-tab>
            </el-collapse-transition>
          </div>
        </el-header>
        <el-main ref="main">
          <custom-menu-tab></custom-menu-tab>
          <custom-app-main ref="appMain"></custom-app-main>
        </el-main>
        <custom-back-top
          :right="50"
          :bottom="50"
          :fontSize="34"
        ></custom-back-top>
      </el-container>
      <div
        class="sidenav-mask"
        :class="{ show: isPhone && collapseState }"
        @click="toggleSidebarState"
      ></div>
    </el-container>
  </div>
</template>

<script>
import emitter from "shared/util/emitter";
import {
  CustomNavBar,
  CustomSidebar,
  CustomAppMain,
  CustomReuseTab,
  CustomMenuTab,
  CustomBackTop,
} from "@/component/layout";

const navBarHeight = 66;
const reuseTabHeight = 70;
const marginHeight = 20;
const sidebarWidth = "210px";
const totalHeight = navBarHeight + reuseTabHeight + marginHeight;

export default {
  components: {
    CustomNavBar,
    CustomSidebar,
    CustomAppMain,
    CustomReuseTab,
    CustomMenuTab,
    CustomBackTop,
  },
  data() {
    return {
      collapseState: false,
      sidebarWidth,
      clientWidth: 0,
      clientHeight: 0,
      foldState: false,
      isPhone: false,
    };
  },
  mounted() {
    this.setResize();
    if (this.clientWidth < 500) {
      this.isPhone = true;
    } else {
      this.isPhone = false;
      if (this.clientWidth <= 768) {
        emitter.emit("removeSidebarSearch");
        this.collapseState = true;
      } else {
        this.collapseState = false;
        emitter.emit("showSidebarSearch");
      }
    }
    window.onresize = () => {
      this.setResize();
      if (this.clientWidth <= 500) {
        this.isPhone = true;
      } else if (this.clientWidth <= 800) {
        this.isPhone = false;
      }
    };

    emitter.on("noReuse", () => {
      this.$refs.operate.style.height = "86px";
    });
    emitter.on("hasReuse", () => {
      this.$refs.operate.style.height = "45px";
    });
  },
  computed: {
    elMenuCollapse() {
      if (this.isPhone) {
        return false;
      }
      return this.collapseState;
    },
    customAsideStyle() {
      const style = {};
      if (this.isPhone) {
        style.position = "absolute";
        style.height = `${this.clientHeight}px`;
        style.zIndex = 12;
        if (this.collapseState === false) {
          style.transform = `translateX(-${sidebarWidth})`;
        } else {
          style.transform = "translateX(0)";
        }
      }
      return style;
    },
  },
  methods: {
    toggleSidebarState() {
      this.collapseState = !this.collapseState;
      if (this.collapseState) {
        emitter.emit("removeSidebarSearch");
      } else {
        emitter.emit("showSidebarSearch");
      }
      this.foldState = !this.foldState;
    },
    setResize() {
      this.clientHeight = document.body.clientHeight;
      this.clientWidth = document.body.clientWidth;
      this.$refs.appMain.$el.style.minHeight = `${
        this.clientHeight - totalHeight + 20
      }px`;
    },
  },
  watch: {
    collapseState() {
      if (this.isPhone) {
        this.sidebarWidth = sidebarWidth;
        if (this.collapseState === false) {
          this.transX = 0;
        } else {
          this.transX = -1 * sidebarWidth;
        }
      } else {
        this.transX = 0;
        this.sidebarWidth =
          this.collapseState === false ? sidebarWidth : "64px";
      }
    },
    $route() {
      this.showBackTop = false;
      if (this.scrollY <= 70) {
        this.backTop();
      }
      if (this.isPhone && this.collapseState) {
        this.toggleSidebarState();
      }
    },
  },
  beforeUnmount() {
    emitter.off("noReuse");
    emitter.off("hasReuse");
  },
};
</script>

<style lang="scss" scoped>
.aside {
  background: rgb(25, 42, 94);
  overflow-x: hidden;

  &::-webkit-scrollbar {
    width: 0px;
    height: 0px;
  }
}

.header {
  padding: 0;
  background: $header-background;
  height: $header-height !important;
  display: flex;
  align-items: center;
  justify-content: space-between;
  box-shadow: 0px 2px 6px 0px rgba(190, 204, 216, 0.4);
  border-bottom: 1px solid rgba(190, 204, 216, 0.4);

  .left {
    height: 100%;
    width: 100%;

    .operate {
      display: flex;
      align-items: center;
      background: $header-background;
      padding-left: 20px;
      height: 86px;

      .iconfont {
        font-size: 16px;
        font-weight: 500;
        color: $right-side-font-color;
        cursor: pointer;
        transform: rotate(0deg);
        transition: all 0.3s linear;
        margin-right: 10px;

        &:hover {
          color: #3963bc;
        }
      }

      .rotate {
        transform: rotate(180deg);
        transition: all 0.3s linear;
      }
    }
  }

  .right-info {
    display: flex;
    align-items: center;
  }
}

.el-main {
  overflow-y: auto;
  position: relative;
  padding: 0;
}

.backTop {
  position: fixed;
  display: inline-block;
  text-align: center;
  cursor: pointer;
  right: 50px;
  bottom: 50px;
  width: 22px;
  height: 22px;
  line-height: 22px;
  border-radius: 50%;
  z-index: 99;
  background: #fff;

  .iconfont {
    font-size: 36px;
  }
}

.sidenav-mask {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.3);
  z-index: 10;
  display: none;
  cursor: pointer;

  &.show {
    display: block;
  }
}
</style>
