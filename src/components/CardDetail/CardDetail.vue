<template>
  <transition :name="slide">
    <div class="carddetail">
      <div class="header" ref="header" :style="{backgroundColor:color}">
        <div class="picname" ref="picname">
          <div class="pic" :style="{backgroundColor:color}">{{userNameCharAt}}</div>
          <div class="userName">{{card.userName}}</div>
        </div>
      </div>
      <div class="bg-layer" ref="layer">
      </div>
      <scroll @scroll="scroll" :listen-scroll="listenScroll" :probe-type="probetype" class="scrolldetail">
        <ul class="scrolldetailul">
          <li>
            <span>姓名</span>
            <span class="value" v-show="!isEdit">{{card.userName}}</span>
            <input type="text" v-show="isEdit" class="edit-input" v-model="editcard.userName">
          </li>
          <li>
            <span>电话</span>
            <span class="value" v-show="!isEdit">{{card.phoneNum}}</span>
            <input type="text" v-show="isEdit" class="edit-input" v-model="editcard.phoneNum">
          </li>
          <li>
            <span>邮箱</span>
            <span class="value" v-show="!isEdit">{{card.email}}</span>
            <input type="text" v-show="isEdit" class="edit-input" v-model="editcard.email">
          </li>
          <li class="li-btn">
            <span class="edit" @click="editOrcancel">{{isEdit?'取消':'编辑'}}</span>
            <span class="delete" @click="removeOrsave">{{isEdit?'保存':'删除'}}</span>
          </li>
          <li class="li-btn">
            <router-link to="/Send" class="share">分享</router-link>
          </li>
        </ul>
      </scroll>
      <div class="modalbg" v-show="canremove">
        <div class="modal">
          <div class="title">确定要删除吗</div>
          <div class="btns">
            <div class="btn btncancel" @click="cencel">取消</div>
            <div class="btn btnsave" @click="remove">确定</div>
          </div>
        </div>
      </div>
      <modal :msg="msg" :mdShow="mdShow" @closeMd="closeMd"></modal>
    </div>
  </transition>
</template>

<script>
import { mapGetters } from "vuex";
import { mapMutations } from "vuex";
import axios from "axios";
import scroll from "base/scroll/scroll";
import modal from "base/modal/modal";

const MIN_SCROLL = 50;
const HEADER_HEIGHT = 250;

export default {
  data() {
    return {
      scrollY: 0,
      isEdit: false,
      editcard: {},
      slide: "slide",
      color: "#" + this.$route.query.color,
      canremove: false,
      msg: "",
      mdShow: false
    };
  },
  mounted() {
    this._getStart();
  },
  created() {
    this.listenScroll = true;
    this.probetype = 3;
  },
  methods: {
    scroll(pos) {
      this.scrollY = pos.y;
    },
    editOrcancel() {
      if (this.isEdit) {
        this.isEdit = false;
      } else {
        this.editcard.key = this.card.key;
        this.editcard.cardid = this.card.cardid;
        this.editcard.userName = this.card.userName;
        this.editcard.phoneNum = this.card.phoneNum;
        this.editcard.email = this.card.email;
        this.isEdit = true;
      }
    },
    cencel() {
      this.canremove = false;
    },
    remove() {
      axios
        .post("/users/removeCard", {
          userName: this.$cookie.get("userName"),
          cardId: this.card.cardid
        })
        .then(res => {
          if (res.data.status == 0) {
            this.setReFresh(true); //重新加载CardList
            this.$router.push({ path: "/CardList" });
          } else {
            console.log(res.data);
          }
        });
    },
    removeOrsave() {
      if (this.isEdit) {
        if (
          this.editcard.userName.trim() == "" ||
          this.editcard.phoneNum.trim() == ""
        ) {
          this.msg = "姓名和电话不能为空";
          this.mdShow = true;
          return;
        }
        axios
          .post("/users/editCard", {
            userName: this.$cookie.get("userName"),
            card: this.editcard
          })
          .then(res => {
            if (res.data.status == 0) {
              this.setReFresh(true); //重新加载CardList
              this.isEdit = false;
              this.setCard(this.editcard);
            } else {
              console.log(res.data);
            }
          });
      } else {
        this.canremove = true;
      }
    },
    _getStart() {
      if (!this.card.userName) {
        this.$router.push("/CardList");
        return;
      }
    },
    closeMd() {
      this.mdShow = false;
    },
    ...mapMutations({
      setCard: "SET_CARD_MUTATION",
      setReFresh: "SET_REFRESH_MUTATION"
    })
  },
  watch: {
    scrollY(newY) {
      if (-newY < HEADER_HEIGHT - MIN_SCROLL && newY < 0) {
        this.$refs.header.style.height = HEADER_HEIGHT + "px";
        this.$refs.header.style.zIndex = 0;
        this.$refs.header.style.transform = `scale(1)`;
        this.$refs.layer.style.transform = `translate3d(0,${newY}px,0)`;
        this.$refs.picname.style.transform = `translate3d(${newY /
          40 *
          9}px,${newY}px,0)`;
      } else if (newY < 0) {
        this.$refs.header.style.height = MIN_SCROLL + "px";
        this.$refs.header.style.zIndex = 10;
        this.$refs.layer.style.transform = `translate3d(0,-190px,0)`;
        this.$refs.picname.style.transform = `translate3d(-40px,-190px,0)`;
      }
      if (newY > 0) {
        let height = HEADER_HEIGHT + newY;
        let delta = height / HEADER_HEIGHT;
        this.$refs.header.style.zIndex = 10;
        this.$refs.header.style.transform = `scale(${delta})`;
        this.$refs.layer.style.transform = `translate3d(0,0,0)`;
        this.$refs.picname.style.transform = `translate3d(0,0,0)`;
      }
    }
  },
  computed: {
    ...mapGetters(["card"]),
    userNameCharAt() {
      if (this.card) {
        return this.card.userName.charAt(0);
      }
    }
  },
  components: {
    scroll,
    modal
  }
};
</script>

<style lang="stylus" scoped>
@import '~common/stylus/variable'

.carddetail
  position fixed
  // 从父原则
  z-index 100
  top 0
  left 0
  right 0
  bottom 0
  background-color #eee

  .header
    position fixed
    top 0
    width 100%
    height 250px

    .picname
      position relative
      height 100%
      margin-left 30px

      .pic
        position absolute
        width 35px
        height 35px
        border-radius 17px
        background-color #fff
        top 200px
        left 20px
        font-size 20px
        line-height 35px
        text-align center
        border 1px solid #555
        color #fff

      .userName
        position absolute
        font-size 20px
        color #fff
        top 208px
        left 70px

  .bg-layer
    height 100%
    top 250px
    position relative
    background-color #eee

  .scrolldetail
    position fixed
    top 250px
    bottom 0
    width 100%

    .scrolldetailul
      min-height 800px

      li
        margin 0 40px
        height 60px
        line-height 60px
        display block
        border-bottom 1px solid #bbb
        padding 0 10px

        .value
          margin-left 40px

        .edit-input
          width 160px
          margin-left 40px
          height 30px
          outline none
          font-size 16px
          border-radius 4px
          border 1px solid #aaa
          transition all 0.5s
          padding-left 10px

          &:focus
            border 1px solid $color-theme

      .li-btn
        margin-top 20px
        border none
        display flex
        justify-content space-around
        text-align center
        font-size 18px
        align-items center

        .edit
          border 1px solid #333
          color #fff
          width 80px
          border-radius 5px
          height 35px
          line-height 35px
          background-color #444
          transition all 0.5s

        .delete
          border 1px solid #333
          width 80px
          border-radius 5px
          color #fff
          height 35px
          line-height 35px
          background-color #ec4c5d
          transition all 0.5s

        .share
          margin-top -10px
          border 1px solid #333
          width 160px
          border-radius 5px
          height 35px
          line-height 35px
          color #000

  .modalbg
    position fixed
    top 0
    left 0
    right 0
    bottom 0
    background rgba(238, 238, 238, 0.5)
    animation comein 0.5s

    .modal
      position absolute
      top 50%
      left 50%
      width 80%
      transform translate(-50%, -50%)
      background-color #fff
      border-radius 6px
      border 1px solid $color-theme

      .title
        text-align center
        margin 0 20px
        padding 40px 10px
        font-size 20px
        border-bottom 1px solid #aaa

      .btns
        display flex
        justify-content space-around
        margin 15px 0px

        .btn
          width 60px
          height 30px
          font-size 18px
          color #fff
          line-height 30px
          border-radius 5px
          text-align center
          margin 0 auto
          float left
          background-color $color-theme

        .btncancel
          background-color #ec4c5d

@keyframes comein
  from
    bottom -500px
    opacity 0.3

  to
    opacity 1

.slide-enter-active, .slide-leave-active
  transition all 0.3s

.slide-enter, .slide-leave-to
  opacity 0
  transform translate3d(100%, 0, 0)
</style>

