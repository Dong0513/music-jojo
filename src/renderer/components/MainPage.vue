<template>
    <div>
        <el-row type="flex" align="middle">
            <el-col :span="23">
                <el-input placeholder="请输入搜索的歌名，歌手，专辑" v-model="keyword" @keyup.enter.native="search" clearable prefix-icon="el-icon-search" class="input-with-select">
                    <el-select v-model="searchEngine" slot="prepend" placeholder="请选择平台">
                        <el-option label="QQ" value="qq"></el-option>
                        <el-option label="酷我" value="kw"></el-option>
                        <el-option label="虾米" value="xm"></el-option>
                        <el-option label="酷狗" value="kg"></el-option>
                        <el-option label="百度" value="bd"></el-option>
                        <el-option label="网易" value="wy"></el-option>
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
                        <el-dropdown-item command="setdir">
                            设置下载路径
                        </el-dropdown-item>
                    </el-dropdown-menu>
                </el-dropdown>
            </el-col>
        </el-row>

        <el-table
                v-loading="loading"
                :data="songData"
                height="460"
                style="width: 100%">
            <el-table-column
                    prop="songname_ori"
                    label="歌名"
                    >
                <template slot-scope="songData">
                    <span v-html="songData.row.songname_ori"></span>
                </template>
            </el-table-column>
            <el-table-column
                    prop="singer"
                    label="歌手"
                    >
                <template slot-scope="songData">
                    <span v-html="songData.row.singer"></span>
                </template>
            </el-table-column>
            <el-table-column
                    prop="albumname_ori"
                    label="专辑">
                <template slot-scope="songData">
                    <span v-html="songData.row.albumname_ori"></span>
                </template>
            </el-table-column>
            <el-table-column
                    fixed="right"
                    label="操作">
                <template slot-scope="scope">
                    <el-button
                            @click.native.prevent="audition(scope.$index, songData)"
                            icon="el-icon-service"
                            circle
                            :loading="songData[scope.$index].loading"
                            size="small">
                    </el-button>
                    <el-button
                            @click.native.prevent="select(scope.$index, songData)"
                            type="text"
                            size="small">
                        获取链接
                    </el-button>
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

        <aplayer ref="aplayer" :lrcType="1" :audio="auditionInfo"/>
             <!--:audio="{-->
            <!--name: '给我一个理由忘记',-->
            <!--artist: 'A-Lin',-->
            <!--url: 'http://m10.music.126.net/20190125150518/3fc8873e7995a8a1bf256f692bec497f/yyaac/2a61/69c1/567f/3df92acb1e2da76c5d8308d2ff79d491.m4a',-->
            <!--cover: 'http://p2.music.126.net/YCSTxlHAvrZ-rEs1_dwQUw==/46179488379897.jpg',-->
            <!--lrc: '[00:47.480]雨都停了这片天灰什麽呢\r\n[00:54.480]我还记得你说我们要快乐\r\n[01:01.160]深夜里的脚步声总是刺耳\r\n[01:06.370]害怕寂寞就让狂欢的城市陪我关灯\r\n[01:13.730]只是哪怕周围再多人感觉还是一个人\r\n[01:21.130]每当我笑了心却狠狠的哭着\r\n[01:26.810]\r\n[01:26.900]给我一个理由忘记那麽爱我的你\r\n[01:34.500]给我一个理由放弃当时做的决定\r\n[01:41.800]有些爱越想抽离却越更清晰\r\n[01:45.640]那最痛的距离是你不在身边\r\n[01:51.600]却在我的心里\r\n[01:59.130]\r\n[02:12.690]当我走在去过的每个地方\r\n[02:20.099]总会听到你那最自由的笑\r\n[02:26.919]当我回到一个人住的地方\r\n[02:32.550]最怕看到冬天你最爱穿的那件外套\r\n[02:40.110]只是哪怕周围再多人感觉还是一个人\r\n[02:47.279]每当我笑了心却狠狠的哭着\r\n[02:53.139]\r\n[02:54.890]给我一个理由忘记那么爱我的你\r\n[03:01.390]给我一个理由放弃当时做的决定\r\n[03:08.839]有些爱越想抽离却越更清晰\r\n[03:13.359]那最痛的距离是你不在身边\r\n[03:18.959]却在我的心里\r\n[03:26.750]\r\n[03:45.060]我找不到理由忘记大雨里的别离\r\n[03:52.799]我找不到理由放弃我等你的决心\r\n[03:59.149]有些爱越想抽离却越更清晰\r\n[04:03.780]那最痛的距离是你不在身边\r\n[04:09.179]却在我的心里\r\n[04:26.800]我想你\r\n[04:31.600]'-->
          <!--}"/>-->


        <el-dialog
                :title="'版本：V' + version"
                :visible.sync="dialogAbout"
                width="40%">
            <span>一款高颜值的音乐下载器, 让你能非常优雅的下载音乐, 详细信息可以访问 Github<br>
                <a href="https://github.com/liuzhuoling2011/music-jojo" target="_blank">https://github.com/liuzhuoling2011/music-jojo</a>
            </span>
            <span slot="footer" class="dialog-footer">
            <el-button type="primary" @click="dialogAbout = false">确 定</el-button>
            </span>
        </el-dialog>
        <el-dialog
                title="下载选择"
                :visible.sync="dialogVisible"
                width="40%">
            <div v-loading.lock="selectLoading">
                <img v-if="songUrlData.hasOwnProperty('专辑封面')" :src="songUrlData['专辑封面']" style="width: 55%; vertical-align: top;"/>
                <div style="margin-left: 5px;width: 40%; display: inline-block">
                    <div style="margin-bottom: 5px"><el-button @click="getLink('24AAC', songUrlData['24AAC'], downloadFile)" size="mini" round v-if="songUrlData.hasOwnProperty('24AAC')">24AAC</el-button></div>
                    <div style="margin-bottom: 5px"><el-button @click="getLink('128MP3', songUrlData['128MP3'], downloadFile)" size="mini" type="primary" round v-if="songUrlData.hasOwnProperty('128MP3')">128MP3</el-button></div>
                    <div style="margin-bottom: 5px"><el-button @click="getLink('320MP3', songUrlData['320MP3'], downloadFile)" size="mini" type="success" round v-if="songUrlData.hasOwnProperty('320MP3')">320MP3</el-button></div>
                    <div style="margin-bottom: 5px"><el-button @click="getLink('FLAC', songUrlData['FLAC'], downloadFile)" size="mini" type="info" round v-if="songUrlData.hasOwnProperty('FLAC')">FLAC</el-button></div>
                    <div style="margin-bottom: 5px"><el-button @click="getLink('APE', songUrlData['APE'], downloadFile)" size="mini" type="warning" round v-if="songUrlData.hasOwnProperty('APE')">APE</el-button></div>
                    <el-button-group>
                        <el-button @click="getLink('MV', songUrlData['MV'], downloadFile)" size="mini" type="danger" round v-if="songUrlData.hasOwnProperty('MV')">MV</el-button>
                        <el-button @click="getLink('lrc', songUrlData['lrc'], downloadFile)" size="mini" round v-if="songUrlData.hasOwnProperty('lrc')">歌词</el-button>
                    </el-button-group>
                </div>

                <el-progress style="margin-top: 5px" :stroke-width="10" :percentage="percentage"></el-progress>
            </div>
            <span slot="footer" class="dialog-footer">
                <el-button type="success" size="mini" round @click="openFolder">打开下载文件夹</el-button>
                <el-button type="primary" size="mini" round @click="dialogVisible = false">确定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
  export default {
    name: 'MainPage',
    data () {
      return {
        version: '1.0.4',
        saveDir: '',
        keyword: '',
        searchEngine: 'qq',
        loading: false,
        selectLoading: false,
        auditionLoading: false,
        songData: [],
        songUrlData: {},
        auditionInfoTmp: {
          name: '',
          artist: '',
          url: '',
          cover: '',
          lrc: ''
        },
        auditionInfo: {
          name: '',
          artist: '',
          url: '',
          cover: '',
          lrc: ''
        },
        songFilename: '',
        dialogVisible: false,
        dialogAbout: false,
        redotAbout: true,
        redotUpdate: false,
        currentPage: 1,
        pageSize: 15,
        totalSize: 0,
        headers: {
          'Cookie': 'Tip_of_the_day=2; encrypt_data=5ccfba13bd5b4343b3428176458710e1ffc926359bd13e3ec2ee75e261891484a92dec8d269619a2cc57ada1a2aabe6012baf2f135a17dc6ae4e5f2735054d438dd90f33889ddcc150025d7aaee1fb8956996beeef4ac0937599c42c0a90f689be52dedb0d0aef057255fb2a8e86cc84643a139acb83217e8d3a9a1ab022f583',
          'Host': 'moresound.tk',
          'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.102 Safari/537.36',
          'X-Requested-With': 'XMLHttpRequest',
          'content-type': 'multipart/form-data'
        },
        percentage: 0,
        updateUrl: 'http://www.zoranjojo.top:9926/api/v1/',
        updateFiles: [],
        platform: require('os').platform(),
        shell: require('electron').shell,
        ipc: require('electron').ipcRenderer,
        app: require('electron').remote.app
      }
    },
    mounted () {
      let ret = this.ipc.sendSync('Config', {
        method: 'get'
      })
      this.saveDir = ret['saveDir']
      this.redotAbout = ret['redotAbout']
      this.getNotice()
      this.getUpdate()
    },
    methods: {
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
          this.ipc.sendSync('Config', {
            method: 'set',
            setting: {
              'saveDir': this.saveDir,
              'redotAbout': false
            }
          })
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
        if (name === 'setdir') {
          let that = this
          this.app.dialog.showOpenDialog({
            properties: ['openDirectory']
          }, function (files) {
            if (files) {
              console.log(files[0])
              that.saveDir = files[0]
              that.ipc.sendSync('Config', {
                method: 'set',
                setting: {
                  'saveDir': files[0],
                  'redotAbout': that.redotAbout
                }
              })
              that.$message.success('设置下载目录成功')
            }
          })
        }
      },
      search () {
        this.songData = []
        var options = {
          url: 'http://moresound.tk/music/api.php?search=' + this.searchEngine,
          headers: this.headers,
          timeout: 5000,
          formData: {
            // 'w': encodeURIComponent(this.keyword),
            'w': this.keyword,
            'p': this.currentPage,
            'n': this.pageSize
          }
        }
        this.loading = true
        let that = this
        this.app.request_remote.post(options, (error, response, body) => {
          that.loading = false
          if (error) {
            that.$message.warning('搜索超时，请重新尝试')
            console.log('error: ', error)
          }
          if (!error && response.statusCode === 200) {
            console.log(body)
            let res = JSON.parse(body)
            if (res['code'].toString() !== '0') {
              that.$message.warning(res['msg'])
              return
            }
            if (res['num'].toString() === '0') {
              that.$message.warning('抱歉，没有找到相关歌曲')
              return
            }
            that.currentPage = parseInt(res['page'])
            that.totalSize = parseInt(res['totalnum'])
            that.songData = []
            for (let i = 0; i < res['song_list'].length; i++) {
              let singer = res['song_list'][i]['singer'][0]['name']
              let songname = res['song_list'][i]['songname']
              let albumname = res['song_list'][i]['albumname']
              let songmid = res['song_list'][i]['songmid']
              that.songData.push({
                'id': i,
                'loading': false,
                'songname_ori': songname,
                'songname': songname.replace(/<sup.*>(.|\n|\r)*<\/sup>/, '').replace('&nbsp;', ' ').replace(/^\s+|\s+$/g, ''),
                'albumname_ori': albumname,
                'albumname': albumname.replace('&nbsp;', ' ').replace(/^\s+|\s+$/g, ''),
                'singer': singer,
                'songmid': songmid
              })
            }
          }
        })
      },
      playMusic (url) {
        this.auditionLoading = false
        this.auditionInfoTmp.url = url
        this.auditionInfo = this.auditionInfoTmp
        let that = this
        setTimeout(() => that.$refs.aplayer.play(), 200)
      },
      audition (index, rows) {
        let itemData = rows[index]
        console.log(itemData)
        itemData.loading = true
        let options = {
          url: 'http://moresound.tk/music/api.php?get_song=' + this.searchEngine,
          headers: this.headers,
          timeout: 5000,
          formData: {
            'mid': itemData['songmid']
          }
        }
        let that = this
        this.app.request_remote.post(options, (error, response, body) => {
          if (error) {
            itemData.loading = false
            that.$message.warning('超时，请重新尝试')
            console.log('error: ', error)
          }
          if (!error && response.statusCode === 200) {
            let res = JSON.parse(body)
            console.log(res)
            that.auditionInfoTmp.artist = res['singer']
            that.auditionInfoTmp.name = res['song']
            that.auditionInfoTmp.cover = res['url']['专辑封面']
            that.getLink('lrc', res['url']['lrc'], (type, body) => {
              that.auditionInfoTmp.lrc = body
            })
            if (that.searchEngine === 'qq' || that.searchEngine === 'kw') {
              that.getLink('24AAC', res['url']['24AAC'], (type, body) => {
                console.log(body)
                let res = JSON.parse(body)
                itemData.loading = false
                that.playMusic(res['url'])
              })
            }
            if (that.searchEngine === 'kg' || that.searchEngine === 'xm') {
              that.getLink('32AAC', res['url']['32AAC'], (type, body) => {
                if (body === undefined) {
                  that.getLink('128MP3', res['url']['128MP3'], (type, body) => {
                    console.log(body)
                    let res = JSON.parse(body)
                    itemData.loading = false
                    that.playMusic(res['url'])
                  })
                } else {
                  console.log(body)
                  let res = JSON.parse(body)
                  itemData.loading = false
                  that.playMusic(res['url'])
                }
              })
            }
            if (that.searchEngine === 'bd') {
              that.getLink('128MP3', res['url']['128MP3'], (type, body) => {
                console.log(body)
                let res = JSON.parse(body)
                itemData.loading = false
                that.playMusic(res['url'])
              })
            }
            if (that.searchEngine === 'wy') {
              that.getLink('64AAC', res['url']['64AAC'], (type, body) => {
                console.log(body)
                let res = JSON.parse(body)
                itemData.loading = false
                that.playMusic(res['url'])
              })
            }
          }
        })
      },
      select (index, rows) {
        this.dialogVisible = true
        this.selectLoading = true
        let itemData = rows[index]
        console.log(itemData)

        let options = {
          url: 'http://moresound.tk/music/api.php?get_song=' + this.searchEngine,
          headers: this.headers,
          timeout: 5000,
          formData: {
            'mid': itemData['songmid']
          }
        }
        let that = this
        this.app.request_remote.post(options, (error, response, body) => {
          that.selectLoading = false
          if (error) {
            that.$message.warning('超时，请重新尝试')
            console.log('error: ', error)
          }
          if (!error && response.statusCode === 200) {
            let res = JSON.parse(body)
            console.log(res)
            let reg = new RegExp('/', 'g')
            that.songFilename = res['song'] + ' - ' + res['singer'].replace(reg, '&')
            that.songFilename = that.songFilename.replace(/\\+|\/+|:+|\*+|\?+|"+|<+|>+|\|+/, '')
            that.songUrlData = res['url']
          }
        })
      },
      downloadFile (type, body) {
        if (!this.checkSaveDir()) {
          return
        }
        this.$message.success('获取直连成功，开始下载')
        this.percentage = 0
        if (type === 'lrc') {
          this.app.fs.writeFile(this.saveDir + '/' + this.songFilename + '.lrc', body, (error) => {
            if (error) {
              console.log(error)
            }
          })
          this.percentage = 100
          return
        }
        let res = JSON.parse(body)
        var receivedBytes = 0
        var totalBytes = 0

        var req = this.app.request_remote({
          method: 'GET',
          uri: res['url']
        })

        var out = this.app.fs.createWriteStream(this.saveDir + '/' + this.songFilename + '.' + res['suffix'])
        req.pipe(out)

        req.on('response', function (data) {
          // Change the total bytes value to get progress later.
          totalBytes = parseInt(data.headers['content-length'], 10)
        })

        let that = this
        req.on('data', function (chunk) {
          receivedBytes += chunk.length
          that.percentage = parseInt(receivedBytes / totalBytes * 100)
          console.log(receivedBytes, totalBytes, that.percentage)
        })

        req.on('end', function () {
          that.percentage = 100
          console.log('下载完成')
        })
      },
      getLink (type, url, func) {
        if (url === undefined) {
          func(type, undefined)
        }
        console.log(type, url)
        let options = {
          url: 'http://moresound.tk/music/' + url,
          headers: this.headers
        }
        this.app.request_remote.get(options, (error, response, body) => {
          if (!error && response.statusCode === 200) {
            console.log(body)
            if (type === 'lrc') {
              func(type, body)
              return
            }
            let res = JSON.parse(body)
            if (res['code'].toString() !== '0') {
              this.$message.error(res['msg'])
              return
            }
            func(type, body)
          }
        })
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

<style scoped>
    .el-select {
        width: 120px;
    }
    .item {
        margin-top: 5px;
        margin-right: 40px;
    }
</style>
