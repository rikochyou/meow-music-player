<template>
  <transition name = "slide">
    <music-list :title="title" :bg-image="bgImage" :songs="songs"></music-list>
  </transition>
</template>

<script>
  import {mapGetters} from 'vuex'
  import {getSingerDetail, getSongVkey} from '../../api/singer'
  import {ERR_OK} from '../../api/config'
  import {createSong} from '../../common/js/song'
  import MusicList from '../../components/music-list/music-list'
  export default {
    data() {
      return {
        songs: []
      }
    },
    computed: {
      title() {
        return this.singer.name
      },
      bgImage() {
        return this.singer.avatar
      },
      ...mapGetters([
        'singer'
      ])
    },
    created() {
      this._getDetail()
    },
    methods: {
      _getDetail() {
        if (!this.singer.id) {
          this.$router.push('/singer')
          return
        }
        getSingerDetail(this.singer.id).then((res) => {
          if (res.code === ERR_OK) {
            this.songs = this._normalizeSong(res.data.list)
          // console.log(this.songs)
          }
        })
      },
      _normalizeSong(list) {
        let ret = []
        list.forEach((item) => {
          let {musicData} = item // 得到music对象
          // console.log('musicData', musicData)
          getSongVkey(musicData.songmid).then((res) => {
           //  console.log('res.data', res.req_0.data.midurlinfo[0].purl)
            const songVkey = res.req_0.data.midurlinfo[0].purl
            if (musicData.songmid && musicData.albummid) {
              ret.push(createSong(musicData, songVkey))
            }
          })
        })
        // console.log(ret)
       return ret
  }
    },
    components: {
      MusicList
    }
  }
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
  @import "~common/stylus/variable"

  .slide-enter-active, .slide-leave-active
    transition: all 0.3s

  .slide-enter, .slide-leave-to
    transform: translate3d(100%, 0, 0)
</style>
