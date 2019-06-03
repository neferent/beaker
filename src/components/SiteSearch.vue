<template>
  <div class="s-sitesearch">
    <div class="s-sitesearch--searchbar__cont">
      <div class="s-sitesearch--icon">
        <i class="icon-search"></i>
      </div>
      <input
        ref="search_input"
        type="text"
        v-model="value"
        placeholder="Search Streamlabs..."
        class="s-sitesearch__input"
        @focus.stop.prevent="toggleSearch"
        @blur.stop.prevent="toggleSearch"
        @keyup.stop.prevent="keyEvent"
      />
      <transition name="s-sitesearch--status">
        <div class="s-sitesearch-status__cont" v-if="noResults && !noQuery">No Results</div>
      </transition>
    </div>
    <transition-group name="expand" @enter="open" @after-enter="afterOpen" @leave="close">
      <!-- Quick Links -->
      <div class="s-sitesearch-results__cont" key="quick_container" v-if="searchOpen && noResults">
        <transition-group name="s-sitesearch--result">
          <div class="s-sitesearch__type--header" key="quick_head">Quick Links</div>
          <div v-if="noResults" key="quick_results">
            <a :href="searchData[quickLinkLoc[i]].route" v-for="(suggested, i) in suggestedLinks" :key="suggested.item.name" class="s-sitesearch-results" :class="{ 's-active-result': currentResult === i }" @mouseover="currentResult = i" @mouseup="blurSearch">
              <div class="s-sitesearch__result--image">
                <i :class="searchData[quickLinkLoc[i]].image" class="s-sitesearch__result--image"></i>
              </div>
              <div class="s-sitesearch__result--title">
                {{ searchData[quickLinkLoc[i]].title }}
              </div>
            </a>
          </div>
        </transition-group>
      </div>
      <!-- Page Results -->
      <div class="s-sitesearch-results__cont" key="page_container" v-if="searchOpen && limitedResult.length >= 1">
        <transition-group name="s-sitesearch--result">
          <div class="s-sitesearch__type--header" key="page_head">Pages</div>
          <div v-if="limitedResult.length >= 1" key="page_results">
            <a :href="searchResult.item.route" v-for="(searchResult, i) in limitedResult" :key="searchResult.item.name" @mouseover="currentResult = i" @mouseup="blurSearch" class="s-sitesearch--href">
              <div class="s-sitesearch-results" :class="{ 's-active-result': currentResult === i }" v-if="searchResult.item.type !== 'user'">
                <div class="s-sitesearch__result--image">
                  <i :class="searchResult.item.image" class="s-sitesearch__result--image"></i>
                </div>
                <div class="s-sitesearch__result--title">
                  {{ searchResult.item.title }}
                </div>
              </div>
            </a>
          </div>
        </transition-group>
      </div>
      <!-- User Results -->
      <div class="s-sitesearch-results__cont" key="user_container" v-if="searchOpen && limitedUserResult.length >= 1">
        <transition-group name ="s-siteserach--result">
          <div class="s-sitesearch__type--header" key="user_head">Users</div>
          <div v-if="limitedUserResult.length >= 1" key="user_results">
            <div class="s-sitesearch-results" :class="{ 's-active-result': currentResult === i + limitedResult.length}" v-for="(userResult, i) in limitedUserResult" :key="userResult.item.id" @mouseover="currentResult = i + limitedResult.length" @mouseup="blurSearch">
                <div class="s-sitesearch__result--image">
                  <i class="icon-question"></i>
                </div>
                <div class="s-sitesearch__result--title">
                  {{ userResult.item.name }}
                </div>
                <div class="s-sitesearch__result--subtitle">
                  {{ userResult.item.otherNames}}
                </div>
            </div>
          </div>
        </transition-group>
      </div>
    </transition-group>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Watch, Vue } from "vue-property-decorator";
import Fuse from "fuse.js";

@Component({})
export default class SiteSearch extends Vue {
  $refs!: {
    search_input: HTMLInputElement;
  };

  result: any = [];
  resultUser: any = [];
  completeResult: any = [];
  private isOpen: Boolean = false;
  private searchOpen: Boolean = false;
  private resultLimit: Number = 4;
  private resultLimitUser: Number = 3;
  private fuse: any = null;
  private user: any = null;
  private value: String = "";
  private quickLinkLoc: any = [];
  private keyEvents: any = [];
  private currentResult: number = 0;

  private tempUserData: any = {
    user: [
      {
        type: "user",
        id: "dfd2fwdf2",
        name: "Neferent",
        title: "Neferent",
        keywords: "Neferent, neferentlive@gmail.com",
        email: "neferentlive@gmail.com",
        subscriber: true,
        tipper: true,
        recentEvent: "Subbed 31m ago",
        totalTips: "$198.50 (52)",
        otherNames: "eager2cheese",
        weight: 15
      },
      {
        type: "user",
        id: "df333333",
        name: "Neferentlive",
        title: "Neferentlive",
        keywords: "Neferent, neferentlive@gmail.com",
        email: "neferentlive@gmail.com",
        subscriber: true,
        tipper: true,
        recentEvent: "Tipped 2w ago",
        totalTips: "$8.00 (3)",
        otherNames: "thatfeker",
        weight: 15
      },
      {
        type: "user",
        id: "fffffffffffffffffffffff",
        name: "gggggg",
        title: "gggggg",
        keywords: "gggggg",
        subscriber: true,
        tipper: true,
        recentEvent: "Followed 1y ago",
        totalTips: "$0.00 (0)",
        otherNames: "geegeegee",
        weight: 15
      }
    ]
  };

  @Prop()
  jsonSearch!: any;
  searchData = this.jsonSearch;

  @Prop({ default: "" })
  search!: String;

  @Prop({ default: "fuseResultsUpdated" })
  eventName!: string;

  @Prop({ default: "fuseInputChanged" })
  inputChangeEventName!: string;

  @Prop()
  quickLinks!: any[];

  get suggestedLinks() {
    return this.quickLinks.filter(i => {
      let findResult: any = this.searchData.find(
        data => data.name === i.item.name
      );
      let suggestResult: any = this.searchData.indexOf(findResult);
      this.quickLinkLoc.push(suggestResult);
      return suggestResult;
    });
  }

  get options() {
    let options = {
      caseSensitive: false,
      includeScore: true,
      includeMatches: false,
      tokenize: false,
      matchAllTokens: false,
      findAllMatches: false,
      shouldSort: true,
      threshold: 0.2,
      location: 0,
      distance: 0.9,
      maxPatternLength: 16,
      minMatchCharLength: 1,
      keys: [
        {
          name: "keywords",
          weight: 0.9
        },
        {
          name: "title",
          weight: 0.1
        }
      ]
    };
    return options;
  }

  get noResults() {
    if (this.result.length === 0 && this.resultUser.length === 0) {
      return true;
    } else {
      return false;
    }
  }

  get noQuery() {
    if (this.value === "") {
      return true;
    } else {
      return false;
    }
  }

  get limitedResult() {
    return this.resultLimit
      ? this.result.slice(0, this.resultLimit)
      : this.result;
  }

  get limitedUserResult() {
    return this.resultLimit
      ? this.resultUser.slice(0, this.resultLimitUser)
      : this.resultUser;
  }

  toggleSearch() {
    this.searchOpen = !this.searchOpen;
  }

  afterOpen(element) {
    element.style.height = "auto";
  }

  open(element) {
    let width = getComputedStyle(element).width;
    element.style.width = width;
    let maxWidth = getComputedStyle(element).width;
    element.style.maxWidth = maxWidth;
    element.style.position = `absolute`;
    element.style.visibility = `hidden`;
    element.style.height = `auto`;
    let height = getComputedStyle(element).height;
    element.style.width = null;
    element.style.position = null;
    element.style.visibility = null;
    element.style.height = 0;
    getComputedStyle(element).height;
    setTimeout(() => {
      element.style.height = height;
    });
  }

  close(element) {
    let height = getComputedStyle(element).height;
    element.style.height = height;
    getComputedStyle(element).height;
    setTimeout(() => {
      element.style.height = 0;
    });
  }

  @Watch("searchData")
  watchSearchData() {
    this.fuse.searchData = this.searchData;
    this.fuseSearch();
  }

  @Watch("tempUserData")
  watchUserData() {
    this.user.tempUserData = this.tempUserData;
    this.userSearch();
  }

  @Watch("search")
  watchSearch() {
    this.value = this.search;
  }

  @Watch("value")
  watchValue() {
    this.$parent.$emit(this.inputChangeEventName, this.value);
    this.$emit(this.inputChangeEventName, this.value);
    this.fuseSearch();
    this.userSearch();
  }

  @Watch("result")
  watchResult(val: [], oldVal: []) {
    if (this.noResults || this.value == "" || val.length != oldVal.length) {
      this.currentResult = 0;
    }
    this.$emit(this.eventName, this.result);
    this.$parent.$emit(this.eventName, this.result);
  }

  keyEvent(event) {
    // KEYPRESS UP
    if (event.keyCode === 38 && this.currentResult > 0) {
      this.currentResult--;
    }
    // KEYPRESS DOWN
    if (this.result.length === 0 && this.result.length === 0 && this.resultUser.length === 0) {
      if (event.keyCode === 40 && this.currentResult < 5) {
        this.currentResult++;
      }
    } else if (this.result.length >= 1 && this.resultUser.length === 0) {
       if (event.keyCode === 40 && this.currentResult < this.limitedResult.length + this.limitedUserResult.length - 1) {
        this.currentResult++;
      }
    } else if (this.result.length === 0 && this.resultUser.length >= 1) {
      if (event.keyCode === 40 && this.currentResult < this.limitedUserResult.length - 1) {
        this.currentResult++;
      }
    } else {
      if (event.keyCode === 40 && this.currentResult < this.limitedResult.length + this.limitedUserResult.length - 1) {
        this.currentResult++;
      }
    }
    // KEYPRESS ENTER
    if (event.keyCode === 13 && this.searchOpen) {
      if (this.result <= 0) {
        window.location.href = this.searchData[
          this.quickLinkLoc[this.currentResult]
        ].route;
        this.blurSearch();
      } else {
        window.location.href = this.limitedResult[
          this.currentResult
        ].item.route;
        this.blurSearch();
      }
    }
    // KEYPRESS ESC
    if (event.keyCode === 27 && this.searchOpen) {
      this.blurSearch();
    }
  }

  initFuse() {
    this.fuse = new Fuse(this.searchData, this.options);
    if (this.search) {
      this.value = this.search;
    }
  }

  initUser() {
    this.user = new Fuse(this.tempUserData.user, this.options);
    if (this.search) {
      this.value = this.search;
    }
  }

  blurSearch() {
    this.value = "";
    this.$refs.search_input.blur();
    this.currentResult = 0;
  }

  fuseSearch() {
    if (this.value.trim() === "") {
      this.result = [];
    } else {
      this.result = this.fuse.search(this.value.trim());
    }
  }

  userSearch() {
    if (this.value.trim() === "") {
      this.resultUser = [];
    } else {
      this.resultUser = this.user.search(this.value.trim());
    }
  }

  sortWeight(a, b) {
    let aResult: any = this.result.find(data => data.item.name === a.item.name);
    let bResult: any = this.result.find(data => data.item.name === b.item.name);
    return b.item.weight * bResult.score - a.item.weight * aResult.score;
  }

  mounted() {
    this.initFuse();
    this.initUser();
  }

  updated() {
    //this.completeResult = this.result.concat(this.resultUser);
    //console.log(this.completeResult);
    //console.log(this.user)
    //console.log(this.userSearch)
    //console.log(this.result);
    //console.log(this.resultUser)
    //console.log(this.searchData);
    //console.log(this.tempUserData.user)
  }
}
</script>

<style lang="less">
@import "./../styles/Imports";

.s-sitesearch {
  border: 1px solid @day-input-border;
  border-radius: @radius;
  max-width: 500px;
  position: relative;
  transform-origin: top;
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  background-color: @day-bg;

  .s-sitesearch__input {
    margin: 0;
    border: none;
    height: 39px;
    font-size: 14px;
    .padding--input();
    background: @day-input-bg;
    font-family: "Roboto";
    color: @day-title;
    width: 100%;
  }

  ::placeholder {
    color: @label;
    opacity: 1;
  }

  .s-sitesearch--href {
    text-decoration: none;
  }

  .s-sitesearch--searchbar__cont {
    display: flex;
    flex-direction: row;
    align-items: center;
    .input-padding();
  }

  .s-sitesearch-status__cont {
    font-size: 14px;
    white-space: nowrap;
    color: @icon;
    position: absolute;
    right: 0;
    display: block;
    .margin-right();
  }

  .s-sitesearch__type--header {
    .input-padding();
    .padding-v-sides();
    font-size: 12px;
    font-weight: @normal;
  }

  .s-sitesearch__result--image {
    width: 14px;
    height: 100%;
    color: @icon;
    .margin-right();

    > i {
      width: 100%;
    }
  }

  .s-sitesearch__result--title {
    font-size: 14px;
    color: @day-paragraph;
    font-weight: @medium;
  }

  .s-sitesearch__result--subtitle {
    .margin-left();
    font-size: 14px;
    color: @label;
  }

  .s-sitesearch-results__cont {
    display: flex;
    flex-direction: column;
    overflow: hidden;
    transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  }

  .s-sitesearch-results {
    display: flex;
    transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
    height: 32px;
    min-height: 32px;
    flex-direction: row;
    align-items: center;
    align-content: center;
    .input-padding();
    .padding-v-sides();
    text-decoration: none;

    &.s-active-result {
      background-color: @light-2;
      .s-sitesearch__result--image,
      .s-sitesearch__result--title {
        color: @day-title;
      }
    }
  }

  &:hover {
    cursor: pointer;
  }
}

.expand-enter-active,
.expand-leave-active {
  transition: all 0.35s cubic-bezier(0.4, 0, 0.2, 1);
  overflow: hidden;
}
.expand-enter,
.expand-leave-to {
  transition: all 0.35s cubic-bezier(0.4, 0, 0.2, 1);
  height: 0;
  opacity: 0;
}

.s-sitesearch--result-enter-active,
.s-sitesearch--status-enter-active {
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  opacity: 1;
}

.s-sitesearch--result-leave-active {
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  position: absolute;
  opacity: 0;
}

.s-sitesearch--status-leave-active {
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  opacity: 0;
}

.s-sitesearch--result-enter,
.s-sitesearch--status-enter,
.s-sitesearch--result-leave-to,
.s-sitesearch--status-leave-to {
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  transform: translateX(10px);
  opacity: 0;
}

.s-sitesearch--result-move,
.s-sitesearch--status-move {
  transition: transform 0.25s cubic-bezier(0.4, 0, 0.2, 1);
}

.s-usersearch--status-enter-active {
  transition: all 0.25s 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  opacity: 1;
}

.s-usersearch--status-leave-active {
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  opacity: 0;
}

.s-usersearch--status-leave-active {
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  opacity: 0;
}

.s-usersearch--status-enter,
.s-usersearch--status-leave-to {
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  transform: translateX(10px);
  opacity: 0;
}

.night {
  .s-sitesearch {
    border: 1px solid @night-input-border;
    background-color: @night-bg;

    .s-sitesearch__input {
      background: @night-input-bg;
      color: @night-title;
    }

    .s-sitesearch-status__cont {
      color: @icon;
    }

    .s-sitesearch__result--image {
      color: @icon;
    }

    .s-sitesearch__result--title {
      color: @night-paragraph;
    }

    .s-sitesearch__result--subtitle {
      color: @label;
    }

    .s-sitesearch-results {

      &.s-active-result {
        background-color: @dark-4;
        .s-sitesearch__result--image,
        .s-sitesearch__result--title {
          color: @night-title;
        }
      }
    }
  }
}

</style>
