<script>
import axios from 'axios'
import convertData from './resource/convert2.json'  //  엑셀 > csv to json > import
export default {
  data() {
    return {
      target: 'ja', //  변경할 언어
      tempText: {},
      result: undefined,
      translatedSum: [],
      long: false,
      many: false,
    }
  },
  methods: {
    manyText() {  //  현재 여기 사용 (128초과 버튼)
      this.many = false;
      this.long = false;
      // const manyText = '슬퍼하는 이름과, 무성할 나의 남은 이런 하나의 까닭이요, 듯합니다. 잠, 라이너 않은 아무 이런 다하지 별을 노루, 내 계십니다. 언덕 하나에 걱정도 가을 소녀들의 별 까닭입니다. 별에도 위에 나의 차 무엇인지 가난한 된 어머님, 봅니다. 어머니 추억과 헤는 가을로 사람들의 까닭이요, 봅니다. 피어나듯이 위에 하나에 별 추억과 이런 별 차 하나 계십니다. 보고, 어머니 피어나듯이 파란 라이너 다 나는 버리었습니다.';
      
      const manyText = convertData; //  데이터
      // let convertResource = Object.keys(manyText.ko);
      // let convertResource = Object.values(manyText.ko);
      let text = Object.values(manyText.ko);
      let textBuff = [];

      text.forEach(item => {
        textBuff.push(item);
      });

      let bigText = [];
      let translated = [];

      translateM();

      async function translateM () {

        bigText = [];

        if (textBuff.length > 128) {
          for (let i = 0; i < 128; i++) {
            console.log('shift');
            bigText.push(textBuff.shift());
          }
        } else {
          console.log('ok');
          for (let i = 0; i < textBuff.length; i++) {
            bigText.push(textBuff.shift());
          }
          console.log('how many?', bigText.length);
        }

        // console.log(convertResource);

        let testResources = {
          target: 'ja',
          'text[]': bigText
        }

        await axios.post('/api/ait/translate',
          testResources,
          {
            headers: {
              'Content-Type': 'application/x-www-form-urlencoded'
            }
          })
          .then(res => {
            console.log('res: ', res);
            let result = res.data.result;

            console.log(result);
  
            result.forEach((translation, i) => {
              // console.log(translation.translatedText);
              // console.log(`${i}, ${bigText[i]} => (${testResources.target}) ${translation.translatedText}`);
              translated.push(result[i].translatedText);
            });
           
            console.log('transalted sum + ', translated);
          })
          console.log('--23-4-23-40-', textBuff.length);
          if (textBuff.length === 0) {
            console.log('bye');
            return;
          } else {
            translateM();
          }
      } 
      console.log('length', textBuff.length);
      this.result = translated;

      console.log('final result', this.result);
      this.many = true;
    },
    longText() {
      this.many = false;
      this.long = false;

      console.log('rge', this.target);
      const longText = '잘부탁해';
    
      // for (let i = 0; i < 10; i++) {
      //   longText.repeat(i + 1);
      //   console.log('llll', longText);
      // }
      // let strSum = '';
      // strSum += longText.repeat(100000);
      // console.log('llll', strSum);

      let testResources = {
        target: this.target,
        'text[]': longText
      }

      axios.post('/api/ait/translate',
      testResources,
        {
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded'
          }
        })
        .then(res => {
          console.log('res: ', res);
          this.result = res.data.result;
        })
      this.long = true;
    },
    sumbit() {
      const inputText = Object.values(this.tempText);

      let testResources = {
        target: this.target,
        'text': inputText
      }

      axios.post('/api/ait/translate',
        testResources,
        {
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded'
          }
        })
        .then(res => {
          console.log('res: ', res);
        })
    },
    downloadJson() {
      const blob = new Blob([data], { type: 'text/plain' });
      const e = document.createEvent('MouseEvent'),
      a = document.createElement('a');
      a.download = 'test.json';
      a.href = window.URL.createObjectURL(blob);
      a.dataset.downloadurl = ['text/json', a.download, a.href].join(':');
      e.initEvent('click', true, false, window, 0, 0, 0, 0, 0, false, false, false, false, 0, null);
      a.dispatchEvent(e);
    }
  }
}
</script>

<template>
  <header>
    <img alt="Vue logo" class="logo" src="./assets/logo.svg" width="125" height="125" />

    <div class="wrapper">
    </div>
  </header>

  <main>
    <form action="">
      <input type="text" v-model="tempText.name">이름
      <input type="text" v-model="tempText.address">주소
      <input type="text" v-model="tempText.age">나이
      <input type="text" v-model="tempText.sex">성별
      <input type="text" v-model="tempText.hobby">취미
      <input type="text" v-model="tempText.interest">관심사

      <button type="button" @click="sumbit">저장 & 번역</button>
    </form>
    <button type="button" @click="longText">204800Bytes 초과</button>
    <button type="button" @click="manyText">128 초과</button>

    <div>
      결과
      <div v-if="long && result">
        <div v-for="row in result">
          {{ row.translatedText }}
        </div>
      </div>
      <div v-if="many && result">
        <div v-for="row in result">
          {{ row }}
        </div>
      </div>
      
    </div>
  </main>
</template>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
