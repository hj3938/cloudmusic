<template>
  <div class="album" v-if="show">
      <!-- <div class="albumPic" :style = '{ "backgroundImage" : "url("+singerDetail.artist.img1v1Url+")" }'></div> -->
      <slot name="pic">

      </slot>
      
      <div class="songContainer allCenter"  :style="{width:`${width}%`}">
          <slot name="header">

          </slot>
          
          <ul class="songs">
              <li class="songDetail left alignCenter" v-for="(item,index) in Songsc"
              :key="index" @dblclick="playsong(index)" :class="{'songActive' :currentSong.name == item.name}">
                  <span class="index" v-if="currentSong.mid != item.mid">{{index+1 | addZero}}</span>
                  <span class="index iconfont icon-horn-copy" v-if="currentSong.mid == item.mid"></span>
                  <i class="love iconfont icon-xinaixin1"  v-if="types!=3 && types!=4 && !isCollect(item.mid)" @click="collect(item.mid)" :plain="true"></i>
                  <i class="love iconfont icon-xinaixin" style="color:red;" :plain="true" @click="canCollect(item.mid)" v-if="types!=3 && types!=4 && isCollect(item.mid)"></i>
                  <i class="down iconfont icon-xiazai" v-if="types!=3 && types!=4"></i>
                  <div class="nameContainer left alignCenter" :style="{width:`${nameWidth}%`}">
                      <div class="songName alignCenter">{{item.name}}
                          <p class="songIns" v-if="item.alia">({{item.alia}})</p>
                      </div>
                      <i class="sq iconfont icon-sq"></i>
                      <i class="play iconfont icon-mv"></i>
                  </div>
                  <div class="aSinger"  v-if="types === 1 || types === 3 || types === 4" @click.stop="toSinger(item.id)">
                      {{item.singer}}
                  </div>
                  <div class="aAblum" v-if="types === 1" @click.stop="toAlbum(item.aid)">
                      {{item.album}}
                  </div>
                  <span class="duration" v-if="types === 0 || types === 3 || types === 4 || types === 5">{{item.duration | time}}</span>
                  <span class="duration" v-if="types === 1" style="text-align:left;">{{item.duration | time}}</span>
              </li>
          </ul>
          <p class="all" @click.stop="more" v-if="all && types === 0" >查看全部50首></p>
      </div>
  </div>
</template>

<script>
import { create } from 'domain';
import {Axios,getSongUrl,likeMusic,getAlbumDetail} from '../common/api'
import {mapGetters,mapMutations,mapActions} from 'vuex'
export default {
    props: {
        Songs: {
            type: Array,
            default: []
        },
        show: {
            type: Boolean,
            default: false,
        },
        test: {
            type: String,
            default:''
        },
        width: {
            type: Number,
            default:60
        },
        types: {
            type: Number,
            default: 0
        },
        nameWidth: {
            type: Number,
            default: 70
        }
    },
    computed: {
        ...mapGetters([
            'playList',
            'currentSong',
            'collectSong'
        ]),
        playCls() {
            return  ''
        },
    },
    data() {
        return {
            all: true,
        }
    },
    created() {
        if(this.types == 0) {
            this.Songsc = this.Songs.slice(0,10)
        }else {
            this.Songsc = this.Songs.slice(0)
        }
    },
    methods: {
        isCollect(id) {
            return this.collectSong.includes(id)
        },
        more() {
            this.Songsc = this.Songs.slice(0,50)
            this.all = false;
        },
        playsong(index) {
            let url = ''
            if(this.types === 4) {
                Axios(getSongUrl,{
                    id:this.Songsc[index].mid
                }).then((res) => {
                    url = res.data[0].url
                    this.Songsc[index].url = url
                    this.insertSong(this.Songsc[index])
                })
            }else {
                if(!this.Songsc[index].picUrl) {
                    let aid = this.Songsc[index].aid
                    Axios(getAlbumDetail,{
                        id: aid
                    }).then((res) => {
                        let picUrl = res.album.picUrl
                        this.Songsc[index].picUrl = picUrl
                    })
                }
                Axios(getSongUrl,{
                    id: this.Songsc[index].mid
                }).then((res) => {
                    url = res.data[0].url
                    this.Songsc[index].url = url
                    this.selectPlay({
                       list:this.Songsc,
                       index:index
                    })
                })
            }  
        },
        collect(id) {
            let params = {
                 like:true,
                id: id,
                timestamp: (new Date()).getTime()
            }
            Axios(likeMusic,params).then((res) => {
                let collectList = this.collectSong.slice(0)
                collectList.push(id)
                this.set_collectSong(collectList)
                this.$message({
                    type:'success',
                    message:'喜欢该音乐成功',
                    center: true
                });
            })      
        },
        canCollect(id) {
            let params = {
                like:false,
                id: id,
                timestamp: (new Date()).getTime()
            }
            Axios(likeMusic,params).then((res) => {
                let collectList = this.collectSong.slice(0)
                let index = collectList.findIndex((item) => {
                    return item == id
                })
                collectList.splice(index,1)
                this.set_collectSong(collectList)
                this.$message({
                    type:'success',
                    message:'取消喜欢音乐成功',
                    center: true
                });
            })         
        },
        toSinger(id) {
            this.$router.push(`/singerDetail/${id}`)
        },
        toAlbum(id) {
            this.$router.push(`/album/${id}`)
        },
        ...mapActions([
            'selectPlay',
            'insertSong'
        ]),
        ...mapMutations({
            setPlayList:'SET_PLAYLIST',
            set_collectSong:'SET_COLLECTSONG'
        })
    }
}
</script>

<style lang='scss'>
@import '../assets/css/base.scss';
    .album {
        font-size: 12px;
        display: flex;
        width: 100%;
        .albumPic {
            width: 150px;
            height: 150px;
            margin-right: 50px;
            cursor: pointer;
        }
        .songContainer {
            
            display: inline-block;
            .Navheader {
                display: flex;
                justify-content: space-between;
                .title {
                    margin-bottom: 10px;
                }
                i {
                    color: #999999;
                    cursor: pointer;
                    padding-right: 10px;
                    &:first-child {
                        padding-right: 15px;
                    }
                    &:hover {
                        color:#333;
                    }
                }
            }
            .songs {
                border: 1px solid $borderColor;
                width: 100%;
                .songActive{
                    background: #EBECED;
                }
                .songDetail {
                    color:#777;
                    padding: 7px 0;
                    width: 100%;
                    &:hover {
                        background: #EBECED;
                    }
                    i {
                        font-size: 13px;
                        cursor: pointer;
                    }
                    .index {
                        width: 25px;
                        padding-left: 20px;
                    }
                    .icon-horn-copy {
                        color:#C62F2F;
                    }
                    .love {
                        width: 30px;
                        &:hover {
                            color:#000;
                        }
                    }
                    .down {
                        width: 30px;
                        &:hover {
                            color:#000;
                        }
                    }
                    .nameContainer {
                        width: 100%;
                        .songName {
                            cursor: pointer;
                            width: 100%;
                            color:#333; 
                            overflow: hidden;
                            white-space: nowrap;
                            text-overflow: ellipsis;
                            .songIns {
                                color:#888888;
                                overflow: hidden;
                                white-space: nowrap;
                                text-overflow: ellipsis;
                            }
                        }
                        .sq {
                            font-size: 18px;
                            color:#FE672E;
                            padding: 0 5px;
                            position: relative;
                            top:4px;
                        }
                        .play {
                            font-size: 18px;
                            color:#D65151;
                            padding: 0 5px;
                            line-height: 15px;
                        }
                    }
                    .aSinger {
                        cursor: pointer;
                        width: 21%;
                        overflow: hidden;
                        white-space: nowrap;
                        text-overflow: ellipsis;
                    }
                    .aAblum {
                        cursor: pointer;
                        margin-left: 10px;
                        overflow: hidden;
                        white-space: nowrap;
                        text-overflow: ellipsis;
                        width: 22%;
                        box-sizing: border-box;
                        padding-left: 3px;
                    }
                    .duration {
                        width: 10%;
                        text-align: right;
                    }
                }
            }
            .all {
                margin: 15px 0 0 15px;
                color: #777;
                cursor: pointer;
                width: 100px;
                transition: color 0.1s ease-in-out;
                &:hover{
                    color:#333;
                    transition: color 0.1s ease-in-out;
                }
            }
        }
    }
    @media screen and (min-width:1200px) {
        .album {
            .songContainer {
                .songs {
                    .songDetail {
                            .aSinger {
                                width: 22%;
                            }
                            .aAblum {
                                width: 22%;
                            }    
                        }
                    }
            }
        }
    }
</style>