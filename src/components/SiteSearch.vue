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
        <div class="s-sitesearch-status__cont" v-if="noResults">No Results</div>
      </transition>
    </div>
    <transition
      name="expand"
      @enter="open"
      @after-enter="afterOpen"
      @leave="close"
    >
      <div
        class="s-sitesearch-results__cont"
        :key="limitedResult.length"
        v-if="searchOpen && limitedResult.length <= 0"
      >
        <div class="s-sitesearch-quicklinks">Quick Links</div>
        <a
          :href="searchData[quickLinkLoc[i]].route"
          v-for="(suggested, i) in suggestedLinks"
          :key="suggested.item.name"
          class="s-sitesearch-results"
          :class="{ 's-active-result': currentResult === i }"
          @mouseover="currentResult = i"
          @mouseup="blurSearch"
        >
          <div class="s-sitesearch__result--image">
            <i
              :class="searchData[quickLinkLoc[i]].image"
              class="s-sitesearch__result--image"
            ></i>
          </div>
          <div class="s-sitesearch__result--title">
            {{ searchData[quickLinkLoc[i]].title }}
          </div>
        </a>
      </div>
    </transition>
    <transition
      name="expand"
      @enter="open"
      @after-enter="afterOpen"
      @leave="close"
    >
      <div
        class="s-sitesearch-results__cont"
        :key="limitedResult.length"
        v-if="searchOpen && limitedResult.length >= 1"
      >
        <transition-group name="s-sitesearch--result">
          <a
            :href="searchResult.item.route"
            v-for="(searchResult, i) in limitedResult"
            :key="searchResult.item.name"
            @mouseover="currentResult = i"
            @mouseup="blurSearch"
            class="s-sitesearch--href"
          >
            <div
              class="s-sitesearch-results"
              :class="{ 's-active-result': currentResult === i }"
              v-if="searchResult.item.type !== 'user'"
            >
              <div class="s-sitesearch__result--image">
                <i
                  :class="searchResult.item.image"
                  class="s-sitesearch__result--image"
                ></i>
              </div>
              <div class="s-sitesearch__result--title">
                {{ searchResult.item.title }}
              </div>
            </div>
          </a>
          <div class="s-sitesearch__result--divider" v-if="limitedUserResult.length >= 1" key="div">
            Users
          </div>

          <div class="s-usersearch-results" :class="{ 's-active': currentResult === i + limitedResult.length}"

           v-for="(userResult, i) in limitedUserResult"
          :key="userResult.item.id"
          @mouseover="currentResult = i + limitedResult.length"
          @mouseup="blurSearch">


              <div class="s-usersearch__result--left">
                <div class="s-usersearch__result--image">
              
                  <i class="icon-question s-usersearch__result--image"></i>
                </div>
                <div class="s-usersearch__result--username">
                  {{ userResult.item.name }}
                </div>
                <div class="s-usersearch__result--othernames">
                  {{ userResult.item.otherNames}}
                </div>
              </div>
              <div class="s-usersearch__result--mid">
                <div class="s-usersearch__result--lastevent">
                  {{ userResult.item.recentEvent }}
                </div>
                <div class="s-usersearch__result--replayevent" v-tooltip="'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Fusce diam mauris, ullamcorper et ipsum vel.'">
                  <i class="icon-reset"></i>
                </div>
              </div>
              <div class="s-usersearch__result--right">
                <div class="s-usersearch__result--totaltips">
                  {{ userResult.item.totalTips }}
                </div>
                <div class="s-usersearch__result--action">
                  <div class="s-usersearch__result--ban">
                    <a href="#" v-tooltip="'Ban this person'">
                      Ban
                    </a>
                  </div>
                 
                </div>
              </div>



          </div>
        </transition-group>

















<!-- 
            <div class="s-usersearch-results" :class="{ 's-active': currentResult === i }" v-if="searchResult.item.type === 'user'">



              <div class="s-usersearch__result--left">
                <div class="s-usersearch__result--image">
              
                  <i class="icon-question s-usersearch__result--image"></i>
                </div>
                <div class="s-usersearch__result--username">
                  {{ searchResult.item.name }}
                </div>
                <div class="s-usersearch__result--othernames">
                  {{ searchResult.item.otherNames}}
                </div>
              </div>
              <div class="s-usersearch__result--mid">
                <div class="s-usersearch__result--lastevent">
                  {{ searchResult.item.recentEvent }}
                </div>
                <div class="s-usersearch__result--replayevent">
                  <i class="icon-reset"></i>
                </div>
              </div>
              <div class="s-usersearch__result--right">
                <div class="s-usersearch__result--totaltips">
                  {{ searchResult.item.totalTips }}
                </div>
                <div class="s-usersearch__result--action">
                  <div class="s-userserach__result--ban">
                    Ban
                  </div>
                 
                </div>
              </div>


            </div> -->














      </div>
    </transition>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Watch, Vue } from "vue-property-decorator";
import Fuse from "fuse.js";
import Tooltip from "../directives/Tooltip"

@Component({
  directives: {
    Tooltip
  }
})
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
    if (this.result.length === 0 && this.value != "") {
      return true;
    } else {
      return false;
    }
  }

  // get fullSort() {
  //   let completeResult = this.result.concat(this.resultUser);
  //   return completeResult;
  // }


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
    if (this.result.length === 0) {
      if (event.keyCode === 40 && this.currentResult < 5) {
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
@import "./../styles/directives/Tooltips";

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
    color: #91979a;
    opacity: 1;
  }

  .s-sitesearch__result--title {
    font-size: 14px;
    color: @day-paragraph;
    font-weight: @medium;
  }

  .s-sitesearch__result--image,
  .s-usersearch__result--image {
    width: 14px;
    height: 100%;
    color: @icon;
    .margin-right();
    > i {
      width: 100%;
    }
  }

  .s-sitesearch--searchbar__cont {
    display: flex;
    flex-direction: row;
    align-items: center;
    .input-padding();
  }

  .s-sitesearch--icon {
    display: flex;
    align-items: center;
    height: 39px;
    color: @icon;
    padding-bottom: 1px; // Aligns Icon Better Visually
  }

  .s-sitesearch--href {
    text-decoration: none;
  }

  .s-sitesearch-results__cont {
    display: flex;
    flex-direction: column;
    overflow: hidden;
    transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);

    .s-sitesearch-quicklinks {
      display: flex;
      align-items: center;
      height: 32px;
      min-height: 32px;
      font-size: 12px;
      color: @label;
      .input-padding();
    }
  }

  .s-usersearch-results {
    display: flex;
    transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
    height: 32px;
    min-height: 32px;
    flex-direction: row;
    align-items: center;
    justify-content: space-between;
    .input-padding();
    .padding-v-sides();
    text-decoration: none;

    &.s-active {
      background-color: @day-dropdown-bg;
      .s-sitesearch__result--image,
      .s-sitesearch__result--title {
        color: @day-title;
      }
    }

    &:hover {
      cursor: default;
    }
  }

  .s-usersearch__result--left,
  .s-usersearch__result--mid,
  .s-usersearch__result--right {
    display: flex;
    flex-direction: row;
    align-content: center;
  }

  .s-sitesearch__result--divider {
    .input-padding();
    .padding-v-sides();
    font-size: 12px;
    font-weight: @normal;
  }

  .s-usersearch__result--username {
    font-size: 14px;
    color: @day-paragraph;
    font-weight: @medium;
    .margin-right();
  }

    .s-usersearch__result--othernames {
    font-size: 14px;
    color: @label;
    font-weight: @medium;
  }

  .s-usersearch__result--lastevent,
  .s-usersearch__result--totaltips {
    font-size: 14px;
    color: @day-paragraph;
    .margin-right();
  }

  .s-usersearch__result--ban,
  .s-usersearch__result--unban {
    font-size: 14px;
    font-weight: @medium;

    a {
      color: @warning;
    }
  }

  // .s-usersearch-header-cont {
  //   height: 32px;
  //   width: 100%;
  //   display: flex;
  //   flex-direction: row;
  //   align-content: center;
  //   transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);

  //   &.s-active {
  //     height: 56px;
  //   }
  // }

  // .s-usersearch-title-cont {
  //   display: flex;
  //   flex-direction: column;
  //   align-items: left;
  //   justify-content: space-between;
  // }

  // .s-usersearch--title {
  //   display: inline-flex;
  //   top: 8px;
  //   left: 8px;
  //   font-size: 14px;
  //   color: @day-paragraph;
  //   font-weight: @medium;
  //   transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);

  //   &.s-active {
  //     color: @day-title;
  //     font-size: 20px;
  //     font-weight: @bold;
  //   }
  // }

  // .s-usersearch--badges {
  //   display: flex;
  //   flex-direction: row;
  //   align-items: center;
  //   font-size: 10px;
  //   @color: @day-paragraph;

  //   > i {
  //     display: inline-flex;
  //     color: @teal;
  //     .margin-right();
  //   }

  //   > span {
  //     display: inline-flex;
  //     .margin-right(2);
  //   }
  // }

  // .s-usersearch__result--image {
  //   display: inline-flex;
  //   font-size: 14px;
  //   height: 100%;
  //   color: @icon;
  //   transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);

  //   .margin-right();
  //   > i {
  //     width: 100%;
  //   }

  //   &.s-active {
  //     font-size: 32px;
  //   }
  // }

  // .s-usersearch--stats {
  //   display: flex;
  //   flex-direction: column;
  //   .padding-top(2);

  //   .s-usersearch-stats--row {
  //     display: flex;
  //     flex-direction: row;
  //     align-items: center;
  //     .padding(0.25);

  //     .s-usersearch-stats--icon {
  //       width: 48px;
  //       padding-left: 16px;
  //       color: @day-title;
  //     }

  //     .s-usersearch-stats--content {
  //       color: @day-title;
  //       font-weight: @medium;
  //       flex-grow: 3;
  //     }

  //     .s-usersearch-stats--action {
  //       width: 50px;
  //       color: @warning;
  //       font-size: 14px;
  //       font-weight: @medium;
  //       align-self: flex-end;
  //       flex-grow: 1;

  //       .s-usersearch-stats--ban {
  //         display: flex;
  //         flex-direction: row;
  //         align-items: center;

  //         > i {
  //           .margin-left();
  //         }
  //         > span {
  //           display: block;
  //           white-space: nowrap;
  //         }
  //       }
  //     }
  //   }
  // }

  .s-sitesearch-status__cont {
    font-size: 14px;
    white-space: nowrap;
    color: @icon;
    position: absolute;
    right: 0;
    display: block;
    .margin-right();
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
      background-color: @day-dropdown-bg;
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
      color: @night-title;
    }

    .s-sitesearch__result--title,
    .s-usersearch--title {
      color: @night-paragraph;
    }

    .s-sitesearch__result--image,
    .s-usersearch__result--image {
      color: @icon;
    }

    .s-sitesearch-results {
      &.s-active-result {
        background-color: @night-dropdown-bg;
        .s-sitesearch__result--image,
        .s-sitesearch__result--title {
          color: @night-title;
        }
      }
    }

    .s-usersearch-results {
      &.s-active {
        background-color: @night-dropdown-bg;
      }
    }

    .s-usersearch__result--image,
    .s-usersearch--title {
      &.s-active {
        color: @night-title;
      }
    }

    .s-usersearch--stats {
      .s-usersearch-stats--row {
        .s-usersearch-stats--icon {
          color: @night-title;
        }

        .s-usersearch-stats--content {
          color: @night-title;
        }
      }
    }

    .s-sitesearch-results__cont {
      .s-sitesearch-quicklinks {
        color: @label;
      }
    }

    .s-sitesearch-status__cont {
      color: @icon;
    }
  }
}
</style>
