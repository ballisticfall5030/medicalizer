<template>
  <div>
    <header class="header">
      <h1>医薬品名メーカー</h1>
    </header>

    <section class="main-section">
      <div id="result"><div id="resulttext">{{ result }}</div></div>
      <div id="result-detail"><div id="detailtext" v-html="detail"></div></div>
      <label>
        <input type="checkbox" v-model="usePreset" />
        プリセットを使用する
      </label>
      <div class="flexBox">
      <input
        type="file"
        @change="handleFileChange"
        accept=".txt"
        :disabled="usePreset"
      />
      <button @click="generateName" :disabled="!canGenerate">名前を生成</button>
      </div>
    </section>
  </div>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue';

const usePreset = ref(false);
const names = ref<string[]>([]);
const result = ref('');
const detail = ref('');
const canGenerate = ref(false);

const presetNames = [
  'イーブイ', 'シャワーズ', 'サンダース', 'ブースター',
  'エーフィ', 'ブラッキー', 'リーフィア', 'グレイシア', 'ニンフィア'
];

watch(usePreset, (val) => {
  if (val) {
    names.value = [...presetNames];
    canGenerate.value = true;
  } else {
    names.value = [];
    canGenerate.value = false;
  }
});

function handleFileChange(event: Event) {
  const input = event.target as HTMLInputElement;
  const file = input.files?.[0];
  if (!file) return;

  const reader = new FileReader();
  reader.onload = (e) => {
    const content = e.target?.result as string;
    names.value = content
      .split(/\r?\n/)
      .map(line => line.trim())
      .filter(line => line.length > 0);

    if (names.value.length > 0) {
      canGenerate.value = true;
      alert(`名前を ${names.value.length} 件読み込みました`);
    }
  };
  reader.readAsText(file, 'utf-8');
}

function maybe(prob: number): boolean {
  return Math.random() < prob;
}

function randomChoice<T>(arr: T[]): T {
  return arr[Math.floor(Math.random() * arr.length)];
}

function insertAtRandom(str: string, insert: string): string {
  const pos = Math.floor(Math.random() * (str.length + 1));
  return str.slice(0, pos) + insert + str.slice(pos);
}

function modifyName(name: string, attempts = 0): string {
    let origName : string = name; 
    let detailname : string = name;

    // 2. 語尾に語尾追加
    if (maybe(0.5)) {
    // 長さに応じて1～2文字削除
    if (name.length === 4 && maybe(0.5)) {
    name = name.slice(0, 3);
    detailname = detailname.slice(0, 3) + '<span style="color: #ddd;">' + detailname.slice(3) + "</span>";
    } else if (name.length >= 5 && maybe(0.5)) {
    name = name.slice(0, -2);
    detailname = detailname.slice(0, 3) + '<span style="color: #ddd;">' + detailname.slice(3) + "</span>";
    }
    let addingword : string = randomChoice(["ン", "リン", "ニン", "ル", "－ル"]); 
    name += addingword;
    detailname += " + " + addingword
    }

    // 1. 伸ばし棒「ー」を追加（含まれていない場合のみ）
    if (name.indexOf("ー") !== -1 && maybe(0.4) && name.length > 1) {
    const pos = Math.floor(Math.random() * (name.length - 1)) + 1;
    name = name.slice(0, pos) + "ー" + name.slice(pos);
    detailname = detailname.slice(0, pos) + " ー " + detailname.slice(pos);
    }

    // 3. 医薬品に良く使われる単語を挿入
    if (maybe(0.1)) {
    let addingword : string = randomChoice(["ケア", "キュア", "ヒール", "メディ"]); 
    name = insertAtRandom(name, addingword);
    detailname += " + " + addingword
    }

    // 4. 元の名前と変わっていなければ、1 ~ 3 を再度繰り返す
    if (origName === name && attempts < 5) {
      return modifyName(name, attempts + 1);
    }
    detail.value = detailname;
    return name;
}

function generateName() {
  if (names.value.length === 0) {
    alert('ファイルが読み込まれていません。');
    return;
  }
  let name : string = randomChoice(names.value);
  name = modifyName(name);
  result.value = name;
}
</script>

<style scoped>
body {
  font-family: sans-serif;
  margin: 0;
  padding: 0;
  background-color: #e6f2ff;
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 100vh;
}

.header {
  background-color: #cce6ff;
  width: 96%;
  padding: 20px;
  margin: 0;
  text-align: center;
  border-radius: 0 0 20px 20px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.header h1 {
  margin: 0;
  font-size: 2em;
  color: #004080;
}

.main-section {
  background-color: #f0f8ff;
  border-radius: 20px;
  padding: 30px 40px;
  margin: 40px auto;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  text-align: center;
  width: 90%;
  max-width: 500px;
}

button,
input[type='file'] {
  font-size: 1em;
  padding: 10px 10px;
  border: none;
  border-radius: 8px;
  background-color: #99ccff;
  color: #003366;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

button:disabled,
input[type='file']:disabled {
  background-color: #d0e6f7;
  color: #888;
  cursor: not-allowed;
}

button:hover:enabled {
  background-color: #80bfff;
}

#result {
    width: 100%;
    margin-bottom: 10px;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    height: 100px;
    border-radius: 8px;
    background-color: #fff;
}

#result-detail {
    width: 100%;
    margin-bottom: 10px;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    height: 30px;
    border-radius: 8px;
    background-color: #fff;
}
#resulttext {
    font-size: 2.4em;
    color: #003366;
    word-break: break-word;
    text-align:center;
}

#detailtext {
    font-size: 1em;
    color: #003366;
    word-break: break-word;
    text-align:center;
}

.flexBox {
    display: flex;
    justify-content:space-between;
}

label {
  display: block;
  margin-top: 10px;
  font-size: 1em;
  color: #004080;
}
</style>
