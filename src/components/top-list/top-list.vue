<template>
    <transition name="slide">
      <music-list :rank="rank" :title="title" :bg-image="bgImage" :songs="songs"></music-list>
    </transition>
</template>

<script>
  import MusicList from 'components/music-list/music-list'
  import {mapGetters} from 'vuex'
  import {getMusicList} from '../../api/rank'
  import {ERR_OK} from '../../api/config'
  import {getSongVkey} from '../../api/singer'
  import {createSong} from '../../common/js/song'

  export default {
    computed: {
      title() {
        return this.topList.topTitle
      },
      bgImage() {
        return this.topList.picUrl
      },
      ...mapGetters([
        'topList'
      ])
    },
    created() {
      this._getMusicList()
    },
    data() {
      return {
        songs: [],
        rank: true
      }
    },
    methods: {
      _getMusicList() {
        if (!this.topList.id) {
          this.$router.push('/rank')
          return
        }
        getMusicList(this.topList.id).then((res) => {
          // console.log(res)
          if (res.code === ERR_OK) {
            // console.log(res.songlist)
            this.songs = this._normalizeSongs(res.songlist)
          }
        })
      },
      _normalizeSongs(list) {
        let ret = []
        list.forEach((item) => {
          const musicData = item.data
           // console.log(musicData)
          getSongVkey(musicData.songmid).then((res) => {
            // console.log('res.data', res.req_0.data.midurlinfo[0].purl)
            // console.log('res.data', res)
            const songVkey = res.req_0.data.midurlinfo[0].purl
            if (musicData.songid && musicData.albumid) {
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
  .slide-enter-active, .slide-leave-active
    transition: all 0.3s ease

  .slide-enter, .slide-leave-to
    transform: translate3d(100%, 0, 0)
</style>
