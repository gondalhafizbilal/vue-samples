<template>
  <section
    :class="{
      widget: true,
      customClassName,
      collapsed: state === 'collapse',
      fullscreened: state === 'fullscreen',
      loading: fetchData,
    }"
    ref="widget"
  >
    <h5
      v-if="widgetTitle && typeof widgetTitle === 'string' && !isCustomHeader"
      class="title"
    >
      {{ widgetTitle }}
    </h5>
    <header
      v-if="widgetTitle && isCustomHeader"
      class="title"
      v-html="widgetTitle"
    ></header>
    <div
      v-if="!isCustomControls && hasMainControls"
      class="widgetControls widget-controls"
    >
      <a v-if="hasSettings || hasSettingsInverse" href="#"
        ><i class="la la-cog"></i
      ></a>
      <a
        @click="loadWidget($event)"
        v-if="hasRefresh"
        href="#"
        :id="`reloadId-${randomId}`"
      >
        <strong v-if="typeof hasRefresh === 'string'" class="text-gray-light">{{
          hasRefresh
        }}</strong>
        <i v-else class="la la-refresh"></i>
        <b-tooltip
          v-if="showTooltip"
          :placement="tooltipPlacement"
          :target="`reloadId-${randomId}`"
          >Reload
        </b-tooltip>
      </a>
      <a
        @click="changeState($event, 'fullscreen')"
        v-if="hasFullscreen && state !== 'fullscreen'"
        href="#"
        :id="`fullscreenId-${randomId}`"
      >
        <i class="glyphicon glyphicon-resize-full"></i>
        <b-tooltip
          v-if="showTooltip"
          :placement="tooltipPlacement"
          :target="`fullscreenId-${randomId}`"
          >Fullscreen
        </b-tooltip>
      </a>
      <a
        @click="changeState($event, 'default')"
        v-if="hasFullscreen && state === 'fullscreen'"
        href="#"
        :id="`restoreId-${randomId}`"
      >
        <i class="glyphicon glyphicon-resize-small"></i>
        <b-tooltip
          v-if="showTooltip"
          :placement="tooltipPlacement"
          :target="`restoreId-${randomId}`"
          >Restore
        </b-tooltip>
      </a>
      <span v-if="hasCollapse && state !== 'collapse'">
        <a
          href="#"
          @click="changeState($event, 'collapse')"
          :id="`collapseId-${randomId}`"
        >
          <i class="la la-angle-down"></i>
          <b-tooltip
            v-if="showTooltip"
            :placement="tooltipPlacement"
            :target="`collapseId-${randomId}`"
            >Collapse
          </b-tooltip>
        </a>
      </span>
      <span v-if="hasCollapse && state === 'collapse'">
        <a
          href="#"
          @click="changeState($event, 'default')"
          :id="`expandId-${randomId}`"
        >
          <i class="la la-angle-up"></i>
          <b-tooltip
            v-if="showTooltip"
            :placement="tooltipPlacement"
            :target="`expandId-${randomId}`"
            >Expand
          </b-tooltip>
        </a>
      </span>
      <a
        v-if="hasClose"
        href="#"
        @click="closeWidget($event)"
        :id="`closeId-${randomId}`"
      >
        <strong v-if="typeof hasClose === 'string'" class="text-gray-light">{{
          hasClose
        }}</strong>
        <i v-else class="la la-remove"></i>
        <b-tooltip
          v-if="showTooltip"
          :placement="tooltipPlacement"
          :target="`closeId-${randomId}`"
          >Close
        </b-tooltip>
      </a>
    </div>
    <div
      v-if="isCustomControls"
      v-html="customControls"
      ref="customControlsRef"
      class="widgetControls widget-controls"
    ></div>
    <div
      :class="`widgetBody widget-body ${bodyClassName}`"
      ref="widgetBodyRef"
      :style="{ display: state === 'collapse' ? 'none' : '' }"
    >
      <Loader
        v-if="fetchData && showLoader"
        :class="'widget-loader'"
        :size="40"
      ></Loader>
      <slot v-else></slot>
    </div>
  </section>
</template>

<script>
import Loader from "../Loader/Loader";

export default {
  name: "Widget",
  data: function () {
    return {
      state: this.isCollapsed ? "collapse" : "default",
    };
  },
  props: {
    isCustomHeader: { type: Boolean, default: false },
    tooltipPlacement: { default: "top" },
    showTooltip: { type: Boolean, default: false },
    hasClose: { type: [Boolean, String], default: false },
    hasFullscreen: { type: [Boolean, String], default: false },
    hasCollapse: { type: [Boolean, String], default: false },
    hasSettings: { type: [Boolean, String], default: false },
    hasSettingsInverse: { type: Boolean, default: false },
    hasRefresh: { type: [Boolean, String], default: false },
    customClassName: { default: "" },
    widgetTitle: { default: "" },
    customControls: { default: null },
    bodyClassName: { default: "" },
    options: { default: () => ({}) },
    fetchData: { type: Boolean, default: false },
    showLoader: { type: Boolean, default: true },
    isCollapsed: { type: Boolean, default: false },
    autoload: { type: [Boolean, Number], default: false },
  },
  components: { Loader },
  computed: {
    randomId() {
      return Math.floor(Math.random() * 100);
    },
    hasMainControls() {
      return !!(
        this.hasClose ||
        this.hasFullscreen ||
        this.hasCollapse ||
        this.hasRefresh ||
        this.hasSettings ||
        this.hasSettingsInverse
      );
    },
  },
  mounted() {
    if (this.autoload && this.$listeners && this.$listeners.load) {
      this.loadWidget();
      if (typeof this.autoload === "number") {
        setInterval(() => {
          this.loadWidget();
        }, this.autoload);
      }
    }
    if (this.isCustomControls) {
      let close = this.$refs.customControlsRef.querySelector("[control=close]");
      close && close.addEventListener("click", this.closeWidget);
      let collapse =
        this.$refs.customControlsRef.querySelector("[control=collapse]");
      collapse &&
        collapse.addEventListener(
          "click",
          this.changeState.bind(this, null, "collapse")
        );
      let expand =
        this.$refs.customControlsRef.querySelector("[control=expand]");
      expand &&
        expand.addEventListener(
          "click",
          this.changeState.bind(this, null, "default")
        );
      let fullscreen = this.$refs.customControlsRef.querySelector(
        "[control=fullscreen]"
      );
      fullscreen &&
        fullscreen.addEventListener(
          "click",
          this.changeState.bind(this, null, "fullscreen")
        );
      let restore =
        this.$refs.customControlsRef.querySelector("[control=restore]");
      restore &&
        restore.addEventListener(
          "click",
          this.changeState.bind(this, null, "default")
        );
      let load = this.$refs.customControlsRef.querySelector("[control=load]");
      load && load.addEventListener("click", this.loadWidget);
    }
  },
  methods: {
    closeWidget(e) {
      e && e.preventDefault();
      let $parentEl = this.$el.parentElement;
      let length = $parentEl.classList.length;
      let parentToRemove = false;
      for (let i = 0; i < length; i++) {
        if (/col.*/.test($parentEl.classList[i])) {
          parentToRemove = true;
          break;
        }
      }

      let removeFunction = () => {
        parentToRemove ? $parentEl.remove() : this.$el.remove();
      };

      if (this.$listeners && this.$listeners.close) {
        this.$listeners.close(removeFunction.bind(this));
      } else {
        removeFunction();
      }
    },
    changeState(e, state) {
      e && e.preventDefault();
      this.state = state;
    },
    loadWidget(e) {
      e && e.preventDefault();
      this.$emit("load");
    },
  },
};
</script>

<style src="./Widget.scss" lang="scss" />
