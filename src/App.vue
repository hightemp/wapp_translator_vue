<template>
  <div class="wrapper">
    <div class="text-block">
      <textarea v-model="sInput"></textarea>
    </div>
    <div class="options">
      from: 
      <select class="form-control" v-model="sFromLang">
        <option value="auto">auto</option>
        <option value="en">en</option>
        <option value="ru">ru</option>
      </select>
      to:
      <select class="form-control" v-model="sToLang">
        <option value="en">en</option>
        <option value="ru">ru</option>
      </select>
      <select class="form-control" v-model="sTarnslator">
        <option value="google">Google</option>
        <option value="yandex">Yandex</option>
        <option value="bing">Bing</option>
      </select>
      <button class="btn btn-light" @click="fnTranslate">Перевести</button>
    </div>
    <div class="output-block">{{sOutput}}</div>
  </div>

</template>

<script>

export default {
  name: 'App',

  components: {

  },

  data() {
    return {
      sFromLang: "en",
      sToLang: "ru",
      sTarnslator: "google",
      sInput: "",
      sOutput: "",
    }
  },

  methods: {
    async fnTranslate() {
      // fetch('https://translate.googleapis.com/translate_a/single?client=gtx&sl=auto&tl=en&hl=ru&dt=t&dt=bd&dj=1&source=icon&tk=467103.467103&q=привет', {
      //   method: 'POST',
      // }).then(res=>res.json())
      //   .then(function(res){
      //       console.log(res);
      //       console.log(res['sentences']);
      //       //делаем что-то с данными.....
      //   });

      // https://translate.google.com.hk/?hl=ru&sl=en&tl=ru&text=The%20Response&op=translate
      // https://translate.google.com/?hl=ru&sl=en&tl=ru&text=The%20Response&op=translate
      var oTranslOptions = {
        google: {
          url: `https://translate.googleapis.com/translate_a/single?client=gtx&dt=t&dt=bd&dj=1&source=icon&tk=467103.467103&`,
          args: {
            hl: 'en',
            sl: this.sFromLang,
            tl: this.sToLang,
            q: this.sInput,
          },
          post: true,
        },
        yandex: {
          url: `https://translate.yandex.net/api/v1/tr.json/translate?srv=yawidget&`,
          post: true,
          args: {
            text: this.sInput,
            lang: `${this.sFromLang}-${this.sToLang}`
          }
        },
        bing: {
          url: `https://www.bing.com/translator`,
          args: {
            text: this.sInput,

          }
        }
      }

      var oT = oTranslOptions[this.sTarnslator]

      var oFetchOptions = {
        mode: 'no-cors',
        headers: {
          'Accept': 'application/json, text/plain, */*',
          'Content-Type': 'application/json'
        },
      }

      if (oT.post) {
        var oF;
        if (oT.post_form) {
          oF = new FormData()
          for (var sK in oT.args) {
            oF.append(sK, oT.args[sK])
          }
        } else {
          oF = new URLSearchParams(oT.args).toString()
        }

        oFetchOptions = {
          method: 'POST'
        }
        if (oT.full) {
          oFetchOptions = {
            method: 'POST', // *GET, POST, PUT, DELETE, etc.
            mode: 'no-cors', // no-cors, *cors, same-origin
            cache: 'no-cache', // *default, no-cache, reload, force-cache, only-if-cached
            credentials: 'same-origin', // include, *same-origin, omit
            headers: {
              // 'Content-Type': 'application/json'
              'Accept': 'application/json, text/plain, */*',
              'Content-Type': 'application/json',
              'Content-Type': 'application/x-www-form-urlencoded',
            },
            redirect: 'follow', // manual, *follow, error
            referrerPolicy: 'no-referrer', // no-referrer, *no-referrer-when-downgrade, origin, origin-when-cross-origin, same-origin, strict-origin, strict-origin-when-cross-origin, unsafe-url
            body: oF
          }
        }
      }

      var sURL = oT.url
      if (oT.args) {
        sURL += new URLSearchParams(oT.args)
      }

      console.log({ sURL, oFetchOptions })
      var oF = await fetch(sURL, oFetchOptions);
      var oR = await oF.json()

      this.sOutput = ""

      if (this.sTarnslator=="google") {
        console.log(oR)
        var aR = oR.sentences.map((oI) => oI.trans)
        this.sOutput = aR.join("");
      }
      if (this.sTarnslator=="yandex") {
        this.sOutput = oR.text.join("");
      }
    }
  },
}
</script>

<style>
body, html {
  height: 100%;
  padding: 0px;
  margin: 0px;
}
#app {
  height: 100%;
}
.wrapper {
  height: 100%;
  display: grid;
  grid-template-rows: 1fr 50px 1fr;
}
.text-block textarea {
  width: 100%;
  height: 100%;
}
.output-block {
  width: 100%;
  height: 100%;
  background: #eee;
}
.options {
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
