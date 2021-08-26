<template>
  <div class="cata-wrapper" :style="popupTheme">
    <div class="title-zone">
      <div class="title">书架</div>
      <div :class="{ 'title-btn': true, loading: refreshLoading }">
        <!-- <span class="home-btn" @click="backToHome">回首页</span> -->
        <span :class="{ loading: refreshLoading }" @click="refreshShelf">
          <i class="el-icon-loading" v-if="refreshLoading"></i>
          {{ refreshLoading ? "刷新中..." : "刷新" }}
        </span>
      </div>
    </div>
    <div
      class="data-wrapper"
      ref="bookList"
      :class="{ night: isNight, day: !isNight }"
    >
      <div class="cata">
        <div
          class="log"
          v-for="(book, index) in bookList"
          :class="{ selected: isSelected(book) }"
          :key="index"
          @click="changeBook(book)"
          ref="book"
        >
          <div class="book-name">
            {{ book.name }}
          </div>
          <div class="chapter-text">
            {{ book.durChapterTitle }}
          </div>
          <div class="extra-text">
            {{ book.durChapterIndex + 1 }} / {{ book.totalChapterNum }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import jump from "../plugins/jump";
import config from "../plugins/config";
import Axios from "axios";
import "../assets/fonts/popfont.css";
export default {
  name: "BookShelf",
  data() {
    return {
      isNight: this.$store.state.config.theme == 6,
      bookList: [],
      refreshLoading: false
    };
  },
  props: ["popBookShelfVisible"],
  computed: {
    theme() {
      return this.$store.state.config.theme;
    },
    popupTheme() {
      return {
        background: config.themes[this.theme].popup
      };
    }
  },
  mounted() {},
  watch: {
    theme(theme) {
      if (theme == 6) {
        this.isNight = true;
      } else {
        this.isNight = false;
      }
    },
    popBookShelfVisible(isVisible) {
      if (isVisible) {
        this.getBookshelf();
      }
    }
  },
  methods: {
    isSelected(book) {
      return book.bookUrl == this.$store.state.readingBook.bookUrl;
    },
    getBookshelf(refresh) {
      Axios.get("http://" + localStorage.url + `/getBookshelf`, {
        params: {
          refresh: refresh ? 1 : 0
        }
      }).then(
        res => {
          this.refreshLoading = false;
          if (res.data.isSuccess) {
            this.bookList = res.data.data || [];
            if (this.bookList.length) {
              this.jumpToActive();
            }
          } else {
            this.$message.error(res.data.errorMsg);
          }
        },
        err => {
          this.loading.close();
          this.$message.error("加载书架信息失败");
          throw err;
        }
      );
    },
    changeBook(book) {
      this.$message.info("换书成功");
      sessionStorage.setItem("bookUrl", book.bookUrl);
      sessionStorage.setItem("bookName", book.name);
      sessionStorage.setItem("chapterIndex", book.durChapterIndex);
      const readingBook = {
        bookName: book.name,
        bookUrl: book.bookUrl,
        index: book.durChapterIndex
      };
      localStorage.setItem("readingRecent", JSON.stringify(readingBook));

      this.$store.commit("setReadingBook", readingBook);
      sessionStorage.setItem("bookUrl", readingBook.bookUrl);
      localStorage.setItem(readingBook.bookUrl, JSON.stringify(readingBook));
      this.$emit("close");
      this.$emit("loadCatalog");
    },
    refreshShelf() {
      if (this.refreshLoading) return;
      this.refreshLoading = true;
      this.getBookshelf(true);
    },
    jumpToActive() {
      this.$nextTick(() => {
        let index = -1;
        this.bookList.some((v, i) => {
          if (v.bookUrl == this.$store.state.readingBook.bookUrl) {
            index = i;
            return true;
          }
        });
        if (index < 0) {
          return;
        }
        let wrapper = this.$refs.bookList;
        jump(this.$refs.book[index], {
          container: wrapper,
          duration: 0
        });
      });
    },
    backToHome() {
      this.$emit("toShelf");
    }
  }
};
</script>

<style lang="stylus" scoped>
.cata-wrapper {
  margin: -16px;
  padding: 18px 25px 24px 25px;

  // background: #ede7da url('../assets/imgs/themes/popup_1.png') repeat;
  .title-zone {
    margin: 0 0 20px 0;
    width: 100%;
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    justify-content: space-between;
  }

  .title {
    font-size: 18px;
    font-weight: 400;
    font-family: FZZCYSK;
    color: #ed4259;
    border-bottom: 1px solid #ed4259;
    width: fit-content;
  }

  .title-btn {
    font-size: 14px;
    line-height: 26px;
    color: #ed4259;
    cursor: pointer;
    .home-btn {
      display: inline-block;
      margin-right: 25px;
    }
    &.loading {
      color: #606266;
    }
  }

  .data-wrapper {
    height: 300px;
    overflow: auto;

    .cata {
      display: flex;
      flex-direction: row;
      flex-wrap: wrap;
      justify-content: space-between;

      .selected {
        color: #EB4259;
      }

      .log {
        width: 100%;
        height: 40px;
        cursor: pointer;
        float: left;
        display: flex;
        flex-direction: row;
        flex-wrap: wrap;
        font: 16px / 40px PingFangSC-Regular, HelveticaNeue-Light, 'Helvetica Neue Light', 'Microsoft YaHei', sans-serif;

        .book-name {
          margin-right: 26px;
          overflow: hidden;
          white-space: nowrap;
          text-overflow: ellipsis;
          width: 25%;
        }

        .chapter-text {
          overflow: hidden;
          white-space: nowrap;
          text-overflow: ellipsis;
          flex: 1;
        }

        .extra-text {
        }
      }
    }
  }

  .night {
    >>>.log {
      border-bottom: 1px solid #666;
    }
  }

  .day {
    >>>.log {
      border-bottom: 1px solid #f2f2f2;
    }
  }
}
</style>