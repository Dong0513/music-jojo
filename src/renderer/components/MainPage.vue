<template>
    <div>
        <el-row type="flex" align="middle">
            <el-col :span="23">
                <el-input placeholder="请输入搜索的歌名，歌手，专辑" v-model="keyword" @keyup.enter.native="search" clearable prefix-icon="el-icon-search" class="input-with-select">
                    <el-select v-model="searchEngine" slot="prepend" placeholder="请选择平台">
                        <el-option label="QQ" value="qq"></el-option>
                        <!--<el-option label="网易" value="wy"></el-option>-->
                        <!--<el-option label="酷狗" value="kg"></el-option>-->
                        <!--<el-option label="百度" value="bd"></el-option>-->
                        <!--&lt;!&ndash;<el-option label="酷我" value="kw"></el-option>&ndash;&gt;-->
                        <!--<el-option label="虾米" value="xm"></el-option>-->
                    </el-select>
                    <el-button @click="search" slot="append" type="success">搜索</el-button>
                </el-input>
            </el-col>
            <el-col :span="1" style="text-align: right">
                <el-dropdown @command="setting">
                    <span class="el-dropdown-link">
                        <i style="font-size: 22px" class="el-icon-setting"></i>
                    </span>
                    <el-dropdown-menu slot="dropdown">
                        <el-dropdown-item command="about">
                            <el-badge v-if="redotAbout" is-dot>关于</el-badge>
                            <el-badge v-if="!redotAbout" is-dot hidden>关于</el-badge>
                        </el-dropdown-item>
                        <el-dropdown-item command="update">
                            <el-badge v-if="redotUpdate" is-dot>更新</el-badge>
                            <el-badge v-if="!redotUpdate" is-dot hidden>更新</el-badge>
                        </el-dropdown-item>
                        <el-dropdown-item command="setting">
                            设置
                        </el-dropdown-item>
                    </el-dropdown-menu>
                </el-dropdown>
            </el-col>
        </el-row>

        <el-table
                v-loading="loading"
                :data="songData"
                height="500"
                style="width: 100%">
            <el-table-column
                    prop="songname_ori"
                    label="歌名"
                    >
                <template slot-scope="songData">
                    <span style="cursor:pointer" v-html="songData.row.songname_ori" @click="searchKey(songData.row.songname_ori)"></span>
                </template>
            </el-table-column>
            <el-table-column
                    prop="singer"
                    label="歌手"
                    >
                <template slot-scope="songData" style="cursor:pointer" >
                    <span style="cursor:pointer" v-html="songData.row.singer" @click="searchKey(songData.row.singer)"></span>
                </template>
            </el-table-column>
            <el-table-column
                    prop="albumname_ori"
                    label="专辑">
                <template slot-scope="songData">
                    <span style="cursor:pointer" v-html="songData.row.albumname_ori" @click="searchKey(songData.row.albumname_ori)"></span>
                </template>
            </el-table-column>
            <el-table-column
                    label="操作">
                <template slot-scope="scope">
                    <el-button
                            @click.native.prevent="playMusic(scope.$index, songData)"
                            icon="el-icon-service"
                            circle
                            :loading="songData[scope.$index].loading"
                            size="small">
                    </el-button>
                    <el-button v-if="musicListIndex(songData[scope.$index].songmid) === -1"
                            @click.native.prevent="starMusic(scope.$index, songData)"
                            icon="el-icon-star-off"
                            circle
                            size="small">
                    </el-button>
                    <el-button v-if="musicListIndex(songData[scope.$index].songmid) !== -1"
                               @click.native.prevent="starMusic(scope.$index, songData)"
                               icon="el-icon-star-on"
                               circle
                               size="small">
                    </el-button>
                    <el-button
                            @click.native.prevent="downloadFile(scope.$index, songData)"
                            icon="el-icon-download"
                            circle
                            size="small">
                    </el-button>
                    <div style="vertical-align: top;margin-left: 10px;display: inline-block;line-height: 0;" v-if="songData[scope.$index].downloading">
                        <el-progress v-show="songData[scope.$index].percentage !== 100" type="circle" :width="32" :stroke-width="1" :percentage="songData[scope.$index].percentage"></el-progress>
                        <el-progress v-show="songData[scope.$index].percentage === 100" type="circle" :width="32" :stroke-width="1" :percentage="songData[scope.$index].percentage" status="success"></el-progress>
                    </div>

                </template>
            </el-table-column>
        </el-table>
        <div style="text-align: center;">
            <el-pagination
                :current-page="currentPage"
                :page-size="pageSize"
                :total="totalSize"
                @current-change="setCurPage"
                layout="prev, pager, next">
            </el-pagination>
        </div>

        <v-nm-player style="left: 0" ref="nmplayer" pos="bottom" :audios="audios" :async-play="playMusicInList" default-cover="http://qiniu.zoranjojo.top/default_cover.jpg" unique="songmid" :sheet-height="330"></v-nm-player>

        <el-dialog
                :title="'版本：V' + version"
                :visible.sync="dialogAbout"
                width="45%">
            <div style="text-align: center">
                <span>一款高颜值的音乐下载器, 让你能非常优雅的下载音乐, 详细信息可以访问 Github<br>
                    <a href="https://github.com/liuzhuoling2011/music-jojo" target="_blank">https://github.com/liuzhuoling2011/music-jojo</a>
                </span>
                <br/><br/>感谢 loliconer 提供的 NMPlayer<br/>
                <div>也可以关注下面公众号，获取最新信息~</div>
                <div><img style="width: 35%" src="http://qiniu.zoranjojo.top/qrcode_for_gh_81(03-26-07-35-31).jpg"/></div>
            </div>
            <span slot="footer" class="dialog-footer">
            <el-button type="primary" @click="dialogAbout = false">确 定</el-button>
            </span>
        </el-dialog>
        <el-dialog @closed="dataReload"
                title="设置"
                :visible.sync="dialogSetting"
                width="40%">
            <div>
                <el-switch
                    v-model="remoteSearch"
                    active-color="#13ce66"
                    active-text="开启代理模式，仅在海外使用打开">
                </el-switch>
            </div>
            <div style="margin-top: 10px">
                <el-button type="text" size="mini" @click="setDownloadDir">设置下载文件夹</el-button>
            </div>
            <span slot="footer" class="dialog-footer">
                <el-button type="success" size="mini" round @click="openFolder">打开下载文件夹</el-button>
                <el-button type="primary" size="mini" round @click="settingConfirm">确定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
  export default {
    name: 'MainPage',
    data () {
      return {
        version: '1.0.6',
        saveDir: '',
        remoteSearch: false,
        keyword: '',
        searchEngine: 'qq',
        loading: false,
        selectLoading: false,
        auditionLoading: false,
        firstStarFlag: true,
        songData: [],
        songUrlData: {},
        songFilename: '',
        dialogSetting: false,
        dialogAbout: false,
        redotAbout: true,
        redotUpdate: false,
        currentPage: 1,
        pageSize: 20,
        totalSize: 0,
        audios: [],
        fakeHeaders: {
          'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
          'Accept-Charset': 'UTF-8,*;q=0.5',
          'Accept-Encoding': 'gzip,deflate,sdch',
          'Accept-Language': 'en-US,en;q=0.8',
          'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64; rv:60.0) Gecko/20100101 Firefox/60.0',
          'referer': 'http://m.y.qq.com',
          'ios_useragent': 'Mozilla/5.0 (iPhone; CPU iPhone OS 9_1 like Mac OS X) AppleWebKit/601.1.46 (KHTML, like Gecko) Version/9.0 Mobile/13B143 Safari/601.1'
        },
        wgetHeaders: {
          'Accept': '*/*',
          'Accept-Encoding': 'identity',
          'User-Agent': 'Wget/1.19.5 (darwin17.5.0)'
        },
        baseUrl: 'http://www.zoranjojo.top:5199/',
        updateUrl: 'http://www.zoranjojo.top:9926/api/v1/',
        updateFiles: [],
        platform: require('os').platform(),
        got: require('got'),
        shell: require('electron').shell,
        ipc: require('electron').ipcRenderer,
        app: require('electron').remote.app
      }
    },
    created () {
      this.dataReload()
      this.getNotice()
      this.getUpdate()

      let that = this
      this.ipc.on('main-process-messages', (event, arg) => {
        that.dataSave()
      })
    },
    methods: {
      dataReload () {
        let ret = this.ipc.sendSync('DataOP', {
          method: 'get'
        })
        this.saveDir = ret['setting']['saveDir']
        this.redotAbout = ret['setting']['redotAbout']
        this.remoteSearch = ret['setting']['remoteSearch']
        this.audios = ret['playlist']
        for (let i = 0; i < this.audios.length; i++) {
          this.audios[i]['url'] = ''
        }
      },
      openFolder () {
        if (!this.checkSaveDir()) {
          return
        }
        this.shell.openItem(this.saveDir)
      },
      setCurPage (pageNum) {
        this.currentPage = pageNum
        this.search()
      },
      checkSaveDir () {
        if (this.saveDir === '') {
          this.$message.warning('请先设置下载目录')
          this.setting('setdir')
          return false
        }
        return true
      },
      setting (name) {
        if (name === 'about') {
          this.dialogAbout = true
          this.redotAbout = false
          this.dataSave()
        }
        if (name === 'update') {
          let msg = ''
          let title = ''
          if (this.updateFiles.length === 0) {
            title = '目前使用的版本已经是最新版~'
          } else {
            title = '可以下载新版本安装替换~'
            for (let i = 0; i < this.updateFiles.length; i++) {
              let url = 'http://' + this.updateFiles[i]
              msg = msg + '<a href="' + url + '">' + url + '</a><br/>'
            }
          }
          this.$alert(msg, title, {
            dangerouslyUseHTMLString: true
          })
        }
        if (name === 'setting') {
          this.dialogSetting = true
        }
      },
      dataSave () {
        this.ipc.sendSync('DataOP', {
          method: 'set',
          setting: {
            'saveDir': this.saveDir,
            'redotAbout': this.redotAbout,
            'remoteSearch': this.remoteSearch
          },
          audios: this.audios
        })
      },
      settingConfirm () {
        this.dataSave()
        this.dialogSetting = false
      },
      setDownloadDir () {
        let that = this
        this.app.dialog.showOpenDialog({
          properties: ['openDirectory']
        }, function (files) {
          if (files) {
            console.log(files[0])
            that.saveDir = files[0]
            that.dataSave()
            that.$message.success('设置下载目录成功')
          }
        })
      },
      searchKey (key) {
        this.keyword = key
        this.search()
      },
      searchCallback (res) {
        console.log(res)
        // let dfiles = this.app.fs.readdirSync(this.saveDir)
        // console.log(dfiles)

        if (res['code'].toString() !== '0') {
          this.$message.warning(res['msg'])
          return
        }
        if (res['num'].toString() === '0') {
          this.$message.warning('抱歉，没有找到相关歌曲')
          return
        }
        this.currentPage = parseInt(res['page'])
        this.totalSize = parseInt(res['totalnum'])
        this.songData = []
        for (let i = 0; i < res['song_list'].length; i++) {
          let singer = res['song_list'][i]['singer']
          let songname = res['song_list'][i]['title']
          let albumname = res['song_list'][i]['album']
          let songmid = res['song_list'][i]['mid']
          this.songData.push({
            'id': i,
            'loading': false,
            'downloading': false,
            'percentage': 0,
            'songname_ori': songname,
            'songname': songname.replace(/<sup.*>(.|\n|\r)*<\/sup>/, '').replace('&nbsp;', ' ').replace(/^\s+|\s+$/g, ''),
            'albumname_ori': albumname,
            'albumname': albumname.replace('&nbsp;', ' ').replace(/^\s+|\s+$/g, ''),
            'singer': singer,
            'songmid': songmid
          })
        }
      },
      qqSearch (callback) {
        let that = this
        let opts = `w=${encodeURIComponent(this.keyword)}&p=${this.currentPage}&n=${this.pageSize}`
        this.got(`http://c.y.qq.com/soso/fcgi-bin/search_for_qq_cp?format=json&${opts}`, {
          headers: this.fakeHeaders,
          responseType: 'json'
        }).then(response => {
          that.loading = false
          let body = JSON.parse(response.body)
          console.log(body)
          let lists = body['data']['song']['list']
          let musicList = []
          for (let i = 0; i < lists.length; i++) {
            let music = {}
            music['source'] = 'qq'
            music['id'] = lists[i]['songid']
            music['title'] = lists[i]['songname']
            let singers = []
            for (let j = 0; j < lists[i]['singer'].length; j++) {
              singers.push(lists[i]['singer'][j]['name'])
            }
            music['singer'] = singers.join('、')
            music['album'] = lists[i]['albumname']
            music['duration'] = parseInt(lists[i]['interval'])
            music['size'] = (lists[i]['size128'] / 1048576).toFixed(2)
            music['mid'] = lists[i]['songmid']
            musicList.push(music)
          }
          let res = {
            'code': 0,
            'msg': '',
            'num': body['data']['song']['curnum'],
            'page': body['data']['song']['curpage'],
            'totalnum': body['data']['song']['totalnum'],
            'song_list': musicList
          }
          callback(res)
        })
      },
      search () {
        this.songData = []
        this.loading = true

        if (!this.remoteSearch) {
          if (this.searchEngine === 'qq') {
            this.qqSearch(this.searchCallback)
          }
        } else {
          let options = {
            url: this.baseUrl + 'search_music',
            // fakeHeaders: this.fakeHeaders,
            timeout: 5000,
            formData: {
              // 'w': encodeURIComponent(this.keyword),
              'w': this.keyword,
              'p': this.currentPage,
              'n': this.pageSize,
              'engine': this.searchEngine
            }
          }
          let that = this
          this.app.request_remote.get(options, (error, response, body) => {
            that.loading = false
            if (error) {
              that.$message.warning('搜索超时，请重新尝试')
              console.log('error: ', error)
            }
            if (!error && response.statusCode === 200) {
              console.log(body)
              let res = JSON.parse(body)
              that.searchCallback(res)
            }
          })
        }
      },
      musicListIndex (mid) {
        return this.audios.findIndex((audio) => audio.songmid === mid)
      },
      addMusicToList (itemData) {
        let mIndex = this.musicListIndex(itemData['songmid'])
        if (mIndex === -1) {
          this.$refs.nmplayer.addAudio({
            cover: 'http://qiniu.zoranjojo.top/default_cover.jpg',
            author: itemData['singer'],
            name: itemData['songname'],
            lrc: '',
            url: '',
            songmid: itemData['songmid']
          })
          return this.audios.length - 1
        }
        return mIndex
      },
      getRandomNum (Min, Max) {
        var Range = Max - Min
        var Rand = Math.random()
        return (Min + Math.round(Rand * Range))
      },
      qqGetUrl (mid, callback) {
        // let that = this
        let guid = this.getRandomNum(1000000000, 10000000000)
        let opts = `guid=${guid}`
        this.got(`http://base.music.qq.com/fcgi-bin/fcg_musicexpress.fcg?format=json&json=3&${opts}`, {
          headers: this.fakeHeaders,
          responseType: 'json'
        }).then(response => {
          let body = JSON.parse(response.body)
          console.log(body)
          // let vkey = body.key

          let prefix = ['M800', 'M500', 'C400']
          for (let i = 0; i < prefix.length; i++) {
            // let lurl = `http://dl.stream.qqmusic.qq.com/${prefix[i]}${mid}.mp3`
            // (async () => {
            //   try {
            //     const request = await that.got(`${lurl}?vkey=${vkey}&guid=${guid}&fromtag=1`, {
            //       headers: this.wgetHeaders,
            //       stream: true,
            //       responseType: 'json'
            //     })
            //     console.log(request)
            //   } catch (error) {
            //     console.log(error)
            //   }
            // })()
            // that.loading = false
          }
        })
      },
      getUrlCallback () {

      },
      ayncUpdateUrl (audiosItem, callback, itemData = null) {
        let index = this.musicListIndex(audiosItem['songmid'])
        if (audiosItem['url'] !== '') {
          callback(index)
          return
        }

        // if (!this.remoteSearch) {
        //   if (this.searchEngine === 'qq') {
        //     this.qqGetUrl(this.getUrlCallback)
        //   }
        // } else {
        {
          let options = {
            url: this.baseUrl + 'download_url',
            timeout: 5000,
            headers: this.fakeHeaders,
            formData: {
              'mid': audiosItem['songmid'],
              'engine': this.searchEngine
            }
          }

          let that = this
          this.app.request_remote.get(options, (error, response, body) => {
            if (error) {
              that.$message.warning('超时，请重新尝试')
              console.log('error: ', error)
            }
            if (!error && response.statusCode === 200) {
              let res = JSON.parse(body)
              console.log(res)
              audiosItem['url'] = res['url']
              if (itemData !== null) {
                itemData['loading'] = false
              }
              callback(index)
            }
          })
        }
      },
      playMusicByIndex (index) {
        this.$refs.nmplayer.playNewAudio(this.audios[index])
      },
      playMusic (index, rows) {
        let itemData = rows[index]
        itemData['loading'] = true
        let mIndex = this.musicListIndex(itemData['songmid'])
        if (mIndex !== -1) {
          let audio = this.audios[mIndex]
          if (audio['url'] !== '') {
            itemData['loading'] = false
            this.playMusicByIndex(mIndex)
          } else {
            this.ayncUpdateUrl(audio, this.playMusicByIndex, itemData)
          }
        } else {
          let index = this.addMusicToList(itemData)
          let audio = this.audios[index]
          this.ayncUpdateUrl(audio, this.playMusicByIndex, itemData)
        }
      },
      async playMusicInList (index) {
        if (this.audios[index].url !== '') {
          this.playMusicByIndex(index)
        } else {
          this.ayncUpdateUrl(this.audios[index], this.playMusicByIndex)
        }
      },
      starMusic (index, rows) {
        let itemData = rows[index]
        let mIndex = this.musicListIndex(itemData['songmid'])
        if (mIndex === -1) {
          mIndex = this.addMusicToList(itemData)
          if (this.firstStarFlag) {
            this.firstStarFlag = false
            let audio = this.audios[mIndex]
            this.ayncUpdateUrl(audio, () => {}, itemData)
          }
        } else {
          this.audios.splice(mIndex, 1)
        }
      },
      getDirectUrl (index, rows, urlCallback) {
        let itemData = rows[index]
        console.log(itemData)
        itemData.loading = true
        let options = {
          url: this.baseUrl + 'download_url',
          timeout: 5000,
          headers: this.fakeHeaders,
          formData: {
            'mid': itemData['songmid'],
            'engine': this.searchEngine
          }
        }

        let that = this
        this.app.request_remote.get(options, (error, response, body) => {
          if (error) {
            itemData.loading = false
            that.$message.warning('超时，请重新尝试')
            console.log('error: ', error)
          }
          if (!error && response.statusCode === 200) {
            let res = JSON.parse(body)
            console.log(res)
            itemData.loading = false
            urlCallback(index, rows, res)
          }
        })
      },
      downloadCallback (index, rows, res) {
        this.$message.success('获取直连成功，开始下载')
        let itemData = rows[index]
        console.info(itemData)
        itemData.percentage = 0

        var receivedBytes = 0
        var totalBytes = 0

        var req = this.app.request_remote({
          method: 'GET',
          uri: res['url']
        })

        var out = this.app.fs.createWriteStream(this.saveDir + '/' + this.songFilename + '.mp3')
        req.pipe(out)

        req.on('response', function (data) {
          // Change the total bytes value to get progress later.
          totalBytes = parseInt(data.headers['content-length'], 10)
        })

        let curPercent = 0
        req.on('data', function (chunk) {
          receivedBytes += chunk.length
          let per = parseInt(receivedBytes / totalBytes * 100)
          if (per > curPercent + 10) {
            curPercent = per
            itemData.percentage = per
          }
          // console.log(receivedBytes, totalBytes, itemData.percentage)
        })

        req.on('end', function () {
          itemData.percentage = 100
          console.log('下载完成')
        })
      },
      downloadFile (index, rows) {
        if (!this.checkSaveDir()) {
          return
        }

        let itemData = rows[index]
        console.log(itemData)

        itemData.downloading = true
        this.songFilename = itemData['songname'] + ' - ' + itemData['singer']
        this.songFilename = this.songFilename.replace(/\\+|\/+|:+|\*+|\?+|"+|<+|>+|\|+/, '')

        this.getDirectUrl(index, rows, this.downloadCallback)
      },
      getNotice () {
        let options = {
          url: this.updateUrl + 'notice'
        }
        let that = this
        this.app.request_remote.get(options, (error, response, body) => {
          if (!error && response.statusCode === 200) {
            console.log(body)
            let storageKey = 'music_jojo_notice_time'
            let notices = JSON.parse(body).data
            for (let i = 0; i < notices.length; i++) {
              if (localStorage[storageKey] == null || localStorage[storageKey] < notices[i].CreatedAt) {
                localStorage.setItem(storageKey, notices[i].CreatedAt)
                that.$notify.success({
                  title: '温馨提示',
                  duration: 10000,
                  message: notices[i].Msg
                })
              }
            }
          }
        })
      },
      getUpdate () {
        const os = require('os')
        let options = {
          url: this.updateUrl + 'update?os=' + os.platform() + '&arch=' + os.arch() + '&version=' + this.version
        }
        console.log(options)
        let that = this
        this.app.request_remote.get(options, (error, response, body) => {
          if (!error && response.statusCode === 200) {
            console.log(body)
            let updateFiles = JSON.parse(body).data.filename
            console.log(updateFiles)
            if (updateFiles !== null && updateFiles.length > 1) {
              updateFiles.shift()
              that.redotUpdate = true
              that.updateFiles = updateFiles
            }
          }
        })
      }
    }
  }
</script>

<style>
    .el-select {
        width: 120px;
    }
    .item {
        margin-top: 5px;
        margin-right: 40px;
    }
    .el-progress__text {
        font-size: 12px !important;
    }

    .el-table::before {
        z-index: unset !important;
    }

    .v-nm-player .nm-sheet .s-body .s-cell:nth-child(3) {
        font-size: 1.2rem;
    }

    ::-webkit-scrollbar {
        width: 8px;
        height: 8px;
    }

    ::-webkit-scrollbar-track {
        border-radius: 4px;
        box-shadow: inset 0 0 6px rgba(0, 0, 0, .2);
        background: #ffff;
    }

    ::-webkit-scrollbar-thumb {
        border-radius: 4px;
        box-shadow: inset 0 0 6px rgba(0, 0, 0, .2);
        background-color: #d9d9d9;
    }

    ::-webkit-scrollbar-thumb:hover {
        background-color: #919191;
    }

    *, *::after, *::before {
        box-sizing: border-box;
        max-height: 999999px;
    }

    html {
        font-size: 62.5%;
        -webkit-text-size-adjust: 100%;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-rendering: optimizeLegibility;
        scroll-behavior: smooth;
        line-height: 1.15;
    }
</style>
