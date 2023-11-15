<template>
  <div v-if="showSidebarSearch" style="margin-top: 15px">
    <div class="search-display" v-if="!showSearchList" @click="toSearch">
      <i class="el-icon-search"></i>
    </div>
    <el-select
      clearable
      filterable
      class="search"
      v-model="selectedItem"
      ref="searchInput"
      v-if="showSearchList"
      :filter-method="filterSearch"
      @change="handleSelectionChange"
      placeholder="Enter keyword"
    >
      <el-option
        v-for="group in searchGroups"
        :key="group.key"
        :label="group.title"
        :value="group.path"
      >
      </el-option>
    </el-select>
  </div>
</template>

<script>
import { mapGetters } from "vuex";
import emitter from "shared/util/emitter";

import Config from "@/config/index";

export default {
  data() {
    return {
      searchGroups: [],
      selectedItem: "",
      showSearchList: false,
      showSidebarSearch: Config.showSidebarSearch,
    };
  },
  computed: {
    ...mapGetters(["sidebarList"]),
  },
  mounted() {
    emitter.on("removeSidebarSearch", () => {
      this.showSidebarSearch = false;
    });
    emitter.on("showSidebarSearch", () => {
      if (Config.showSidebarSearch) {
        this.showSidebarSearch = true;
      }
    });
  },
  methods: {
    handleSelectionChange(value) {
      this.searchGroups = [];
      this.selectedItem = "";
      this.showSearchList = false;
      this.$router.push(value);
    },
    toSearch() {
      this.showSearchList = true;
      setTimeout(() => {
        this.$refs.searchInput.focus();
      }, 200);
    },
    filterSearch(keyword) {
      this.searchGroups = [];

      function deepTraversal(config, callback) {
        if (Array.isArray(config)) {
          config.forEach((subConfig) => {
            deepTraversal(subConfig, callback);
          });
        } else if (config.children) {
          config.children.forEach((subConfig) => {
            deepTraversal(subConfig, callback);
          });
        } else {
          callback(config);
        }
      }

      deepTraversal(this.sidebarList, (viewConfig) => {
        if (viewConfig.title.includes(keyword)) {
          const viewRouter = {};
          viewRouter.path = viewConfig.path;
          viewRouter.title = viewConfig.title;
          viewRouter.key = Math.random();
          this.searchGroups.push(viewRouter);
        }
      });
    },
  },
};
</script>

<style lang="scss" scoped>
.search-display {
  position: relative;
  width: 80%;
  margin: 0 auto;
  height: 36px;
  border-bottom: 1px rgb(185, 190, 195) solid;
  cursor: pointer;

  .el-icon-search {
    position: absolute;
    left: 1px;
    top: 10px;
    color: rgb(185, 190, 195);
  }
}

.search {
  width: 80%;
}
</style>
