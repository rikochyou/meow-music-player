<template>
    <scroll class="suggest"
            :data="result"
            :pullup="pullup"
            :beforeScroll="beforeScroll"
            @scrollToEnd="searchMore"
            @beforeScroll="listScroll"
            ref="suggest"
    >
      <ul class="suggest-list">
        <li @click="selectItem(item)" class="suggest-item" v-for="(item, index) in result" :key="index">
          <div class="icon">
            <i :class="getIconCls(item)"></i>
          </div>
          <div class="name">
            <p class="text" v-html="getDisplayName(item)"></p>
          </div>
        </li>
        <loading v-show="hasMore" title=""></loading>
      </ul>
      <div v-show="!hasMore && !result.length" class="no-result-wrapper">
        <no-result title="抱歉，暂无搜索结果"></no-result>
      </div>
    </scroll>
</template>

<script>
  import {search} from '../../api/search'
  import {ERR_OK} from '../../api/config'
  import {createSong} from '../../common/js/song'
  import {getSongVkey} from '../../api/singer'
  import Scroll from '../../base/scroll/scroll'
  import Loading from '../../base/loading/loading'
  import Singer from 'common/js/singer'
  import {mapMutations, mapActions} from 'vuex'
  import NoResult from 'base/no-result/no-result'

  const TYPE_SINGER = 'singer'
  const perpage = 20
 // let _genRet = []
  export default {
    components: {Scroll, Loading, NoResult},
    props: {
      query: {
        type: String,
        default: ''
      },
      showSinger: {
        type: Boolean,
        default: true
      }
    },
    data() {
      return {
        page: 1,
        result: [],
        pullup: true,
        hasMore: true,
        beforeScroll: true
      }
    },
    methods: {
      search() {
        this.page = 1
        this.hasMore = true
        this.$refs.suggest.scrollTo(0, 0)
       // console.log('query', this.query, this.page, this.showSinger)
        search(this.query, this.page, this.showSinger, perpage).then((res) => {
          // console.log(res)
          if (res.code === ERR_OK) {
          // console.log('search', res.data)
          //  this._genResult(res.data)
          //  setTimeout(() => { this.result = _genRet }, 700)
          // let b = this._genRetSinger(res.data)
            this.result = this._genResult(res.data)
            console.log('searchresult', this.result)
            this._checkMore(res.data)
          }
        })
      },
      searchMore() {
        if (!this.hasMore) {
          return
        }
        this.page++
        search(this.query, this.page, this.showSinger, perpage).then((res) => {
          if (res.code === ERR_OK) {
            this.result = this.result.concat(this.result, this._genResult(res.data))
            this._checkMore(res.data)
          }
        })
      },
      getIconCls(item) {
        if (item.type === TYPE_SINGER) {
          return 'icon-mine'
        } else {
          return 'icon-music'
        }
      },
      getDisplayName(item) {
        console.log('item', item)
        if (item.type === TYPE_SINGER) {
          return item.singername
        } else {
          return `${item.name} - ${item.singer}`
        }
      },
      selectItem(item) {
        console.log(item)
        if (item.type === TYPE_SINGER) {
          const singer = new Singer({
            id: item.singermid,
            name: item.singername
          })
          this.$router.push({
            path: `/search/${singer.id}`
          })
          this.setSinger(singer)
        } else {
          this.insertSong(item)
        }
        this.$emit('select')
      },
      refresh() {
        this.$refs.suggest.refresh()
      },
      listScroll() {
        this.$emit('listScroll')
      },
      _checkMore(data) {
        const song = data.song
        if (!song.list.length || (song.curnum + song.curpage * perpage) >= song
          .totalnum) {
          this.hasMore = false
        }
      },
      _genRetSinger(data) {
        let ret = []
        if (data.zhida && data.zhida.singerid) {
          ret.push({...data.zhida, ...{type: TYPE_SINGER}})
        }
        return ret
      },
      _genResult(data) {
       console.log('genResultdata', data)
        let ret = []
        let ret1 = []
        if (data.zhida && data.zhida.singerid) {
          ret.push({...data.zhida, ...{type: TYPE_SINGER}})
        }
        if (data.song) {
          /*
           console.log('retdata', ret)
           let ret1 = this._normalizeSongs(data.song.list)
           setTimeout(() => {
             _genRet = ret.concat(ret1)
             console.log('_genRet1', _genRet)
           }, 600) */
          ret1 = this._normalizeSongs(data.song.list)
        }
        console.log(ret1)
        return ret1
      },
      _normalizeSongs(list) {
        let ret = []
        list.forEach((musicData) => {
          // console.log(musicData)
          getSongVkey(musicData.songmid).then((res) => {
            // console.log('res.data', res.req_0.data.midurlinfo[0].purl)
            const songVkey = res.req_0.data.midurlinfo[0].purl
            if (musicData.songid && musicData.albumid) {
                ret.push(createSong(musicData, songVkey))
            }
          })
        })
       // console.log('getVkey', ret)
        return ret
      },
      ...mapMutations({
        setSinger: 'SET_SINGER'
      }),
      ...mapActions(['insertSong'])
    },
    watch: {
      query() {
        this.search()
      }
    }
  }
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
  @import "~common/stylus/variable"
  @import "~common/stylus/mixin"

  .suggest
    height: 100%
    overflow: hidden
    .suggest-list
      padding: 0 30px
      .suggest-item
        display: flex
        align-items: center
        padding-bottom: 20px
      .icon
        flex: 0 0 30px
        width: 30px
        [class^="icon-"]
          font-size: 14px
          color: $color-text-d
      .name
        flex: 1
        font-size: $font-size-medium
        color: $color-text-d
        overflow: hidden
        .text
          no-wrap()
    .no-result-wrapper
      position: absolute
      width: 100%
      top: 50%
      transform: translateY(-50%)
</style>
