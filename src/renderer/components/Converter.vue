<template>
  <div :class="getPointerClass()">
    <v-row>
      <v-col
        cols="12"
        sm="5"
      >
        <div class="textarea-wrapper">
          <v-textarea
            v-model="fromText" rows="40" auto-grow outlined
            :disabled="busy"
            />
        </div>
      </v-col>
      <v-col
        cols="12"
        sm="2"
      >
        <v-row>
          <v-col>
            <v-text-field
              v-model="resHead"
              placeholder="レスヘッダの先頭文字列"
              persistent-hint
              hint="レスヘッダの先頭文字列"
              :disabled="busy"
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col>
            <v-text-field
              v-model="quoteHead"
              placeholder="引用文の先頭文字列"
              persistent-hint
              hint="引用文の先頭文字列"
              :disabled="busy"
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col>
            <v-btn
              :disabled="!(resHead.length > 0 && quoteHead.length > 0 && !busy)"
              @click="convert"
            >変換</v-btn>
          </v-col>
          <v-col>
            <v-btn
              :disabled="!(toText.length > 0 && !busy)"
              @click="$copyText(toText)"
            >変換後をコピー</v-btn>
          </v-col>
        </v-row>
        <v-row>
          <v-col v-if="lines.length">
            <span>行数:{{ lines.length }}</span>
          </v-col>
        </v-row>
        <v-row>
          <v-col v-if="fromText.length">
            <span>文字数:{{ fromText.length }}</span>
          </v-col>
        </v-row>
        <v-row>
          <v-col v-if="responses.length">
            <span>レス数:{{ responses.length }}</span>
          </v-col>
        </v-row>
      </v-col>
      <v-col
        cols="12"
        sm="5"
      >
        <div class="textarea-wrapper">
          <v-textarea
            v-model="toText" rows="40" auto-grow outlined
            :disabled="busy"
            />
        </div>
      </v-col>
    </v-row>
  </div>

</template>

<script>
import Electron from 'electron'

export default {
  data: () => ({
    resHead: '…',
    quoteHead: '>',
    fromText: '… 920/03/04(水)00:06:08No.668156394+' + '\n' +
      '結局初期メンバー3人でニノ隊と当たる展開になったか' + '\n' +
      'それでどうするんですこれ…' + '\n' +
      '… 1020/03/04(水)00:06:32No.6681565108' + '\n' +
      'ニノ隊誰も落ちないのは予想外だった' + '\n' +
      '… 1120/03/04(水)00:07:03No.668156638+' + '\n' +
      '>結局初期メンバー3人でニノ隊と当たる展開になったか' + '\n' +
      '>それでどうするんですこれ…' + '\n' +
      '展開的には熱いが勝ちの目が見えねえ…' + '\n' +
      '… 1220/03/04(水)00:07:55No.668156909+ ' + '\n' +
      '>ニノ隊誰も落ちないのは予想外だった' + '\n' +
      '玉狛とうまいこと同じ条件での対決になったな…' + '\n' +
      '… 1320/03/04(水)00:08:43No.668157121+' + '\n' +
      '>結局初期メンバー3人でニノ隊と当たる展開になったか' + '\n' +
      '>それでどうするんですこれ…' + '\n' +
      'ヒュースとチカちゃんで火力戦か' + '\n' +
      'ヒュースとユーマで接近戦挑みます！' + '\n' +
      '… 1420/03/04(水)00:08:58No.668157201+' + '\n' +
      '今まで誰も使ってた記憶が無いけどニノの玉型シールドって何なのそれ…' + '\n' +
      '… 1520/03/04(水)00:09:09No.668157249+' + '\n' +
      '>まさかあの弓場サンが一人も落とせないとは' + '\n' +
      'ランク戦前に持ち上げてた里見さんばつが悪そう' + '\n' +
      '…  1620/03/04(水)00:09:21No.66815732022' + '\n' +
      'ベイルアウト後のリーゼント解いた弓場ちゃんセクシーすぎね？' + '\n' +
      '… 1720/03/04(水)00:09:31No.6681573771' + '\n' +
      'チカちゃんはライトニング構えてるから火力戦はやらないってもう確定してるよ' + '\n' +
      '… 1820/03/04(水)00:09:44No.66815744312' + '\n' +
      '>ヒュースとチカちゃんで火力戦か' + '\n' +
      '>ヒュースとユーマで接近戦挑みます！' + '\n' +
      'ヒュースもういねえだろ！' + '\n' +
      '… 1920/03/04(水)00:10:01No.66815754413' + '\n' +
      '>>結局初期メンバー3人でニノ隊と当たる展開になったか' + '\n' +
      '>>それでどうするんですこれ…' + '\n' +
      '>ヒュースとチカちゃんで火力戦か' + '\n' +
      '>ヒュースとユーマで接近戦挑みます！' + '\n' +
      ' …ヒュースもういねーじゃん',
    toText: '',
    lines: [],
    responses: [],
    converted: [],
    busy: false
  }),
  created () {
    Electron.webFrame.setZoomFactor(0.7)
  },
  methods: {
    convert () {
      this.busy = true
      this.getResponses()
      this.converted = []

      for (const res of this.responses) {
        console.debug('res of response.header:', res.header)
        const causedBy = this.getQuoteLine(res)
        console.debug('res convert caused by:', causedBy)
        if (causedBy.length) {
          // 引用元を探す
          let addQuotes = false
          for (const convertedRes of this.converted) {
            // 変換後の配列が引用元かどうかチェック
            addQuotes = this.execConvert(res, causedBy, convertedRes)
            if (addQuotes) break
          }
          if (!addQuotes) {
          // 変換後の末尾に追加
            res.quotes = res.quotes || []
            this.converted.push(res)
          }
        } else {
          console.debug('converted.push(res):', res)
          // 変換後の末尾に追加
          res.quotes = res.quotes || []
          this.converted.push(res)
        }
      }

      // 変換後を文字列へ
      let lines = []
      for (const convertedRes of this.converted) {
        console.debug('変換後のレス配列を行配列に. convertedRes:', convertedRes.header)
        lines = lines.concat(this.getLinesFromConverted(convertedRes))
      }
      this.toText = lines.join('\n')
      this.busy = false
      alert('変換終了')
    },
    getLinesFromConverted (convertedRes) {
      let lines = []
      lines.push(convertedRes.header)
      console.debug('lines pushed:', lines)
      console.debug('convertedRes.body:', convertedRes.body)
      lines = lines.concat(convertedRes.body)
      console.debug('lines concated:', lines)
      for (const resChild of convertedRes.quotes) {
        console.debug('変換後のレスの子供レスを行配列に. resChild:', resChild.header)
        const childLines = this.getLinesFromConverted(resChild)
        lines = lines.concat(childLines)
      }
      console.debug('lines.length:', lines.length)
      return lines
    },
    /**
      本文から引用符で始まる行を返す
     */
    getQuoteLine (res) {
      let causedBy = ''
      const regex = new RegExp(`^[${this.quoteHead}]([^${this.quoteHead}].*)`)
      // 引用符で始まる本文があるかどうか
      for (const line of res.body) {
        if (line.match(regex)) {
          causedBy = line.replace(regex, '$1')
          break
        }
      }
      return causedBy
    },
    /**
      src: 変ま前の元のレス
      causedBy: 変換前のレスで、引用文とみなした行
      target: 引用元かどうか確認する変換後のレス
      return:
        引用元がtargetの子孫でみつかった場合、true
     */
    execConvert (src, causedBy, target) {
      if (!causedBy.length) return false

      // 本文で引用しているため、どのレスの子供にするか判定
      let addQutes = false
      console.debug('変換後のレス本文をチェック. target.header:', target.header)
      // 変換後のレスを1行ずつチェック
      for (const line of target.body) {
        console.debug('変換後のレス本文をチェック. line:', line)
        // 一行ずつチェックする
        if (causedBy === line) {
          console.debug('変換後のレス本文に引用元があった！ line:', line)
          target.quotes = target.quotes || []
          target.quotes.push(src)
          addQutes = true
          break
        }
      }
      // 変換後のレス本文に引用元がなかった場合
      // その子供（変換後のレス本文から引用したレス）もチェック
      if (!addQutes) {
        console.debug('変換後のレス本文になかったので子供をチェック')
        for (const child of target.quotes) {
          addQutes = this.execConvert(src, causedBy, child)
          if (addQutes) {
            break
          }
        }
      }
      return addQutes
    },
    // 変換元のレスを配列に
    getResponses () {
      this.responses = []
      this.lines = this.fromText.split(/\n/)

      let resHeader = ''
      let resLines = []
      const regex = new RegExp(`^${this.resHead}\\s+\\d*\\d{1,4}/\\d{1,2}/\\d{1,2}.*`)
      for (const line of this.lines) {
        if (line.match(regex) &&
          resLines.length) {
          this.responses.push({
            header: resHeader,
            body: resLines,
            quotes: []
          })
          resHeader = line
          resLines = []
        } else {
          if (!resHeader.length) {
            resHeader = line
          } else {
            resLines.push(line)
          }
        }
      }
      if (resLines.length) {
        this.responses.push({
          header: resHeader,
          body: resLines,
          quotes: []
        })
      }
    },
    getPointerClass () {
      if (this.busy) {
        return 'busy'
      } else {
        return 'not-busy'
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.element {
  font-size: 0.7em;
}
.textarea-wrapper {
  margin-left: 5px;
  margin-right: 5px;
}
.busy {
  cursor: progress;
}
.not-busy {
  cursor: default;
}
</style>