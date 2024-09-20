<script setup>
import { ref } from "vue";
import axios from "axios";
import $ from "jquery";

const novelUrl = ref("418953");
let dgyx = "";

// 將合併的章節導出為文件
function createfiles(data, name) {
  //Blob为js的一个对象，表示一个不可变的, 原始数据的类似文件对象，这是创建文件中不可缺少的！
  var urlObject = window.URL || window.webkitURL || window;
  var export_blob = new Blob([data]);
  var save_link = document.createElement("a");
  save_link.href = urlObject.createObjectURL(export_blob);
  save_link.download = name;
  save_link.click();
}

// 請求網頁，或者內容，並格式化每一個章節
function requestNet(url) {
  try {
    axios({
      url,
      method: "GET",
    }).then((res) => {
      console.log(res);
      let htmlContent = res.data;
      if (!htmlContent) {
        if (dgyx) {
          createfiles(dgyx, "道詭異仙");
        }
        return;
      }

      let book = $(htmlContent).filter("#book")[0];

      let next = $(book).find(".content").find(".page_chapter").find("ul")[1]
        .children[2];
      let nextUrl = $(next).find("a")[0].href;
      console.log("nextUrl", nextUrl);

      let title = $(book).find(".content").find("h1")[0];
      title = title.innerText;

      let content = $(book).find("#content")[0];
      let text = content.innerText;
      text = text.replace(/\n\n/gi, "\n");

      let novel = title + "\r\n\n" + text + "\r\n\n\n\n";
      dgyx += novel;
      requestNet(nextUrl);
    });
  } catch (error) {
    if (dgyx) {
      createfiles(dgyx, "道詭異仙");
    }
  }
}

// 開始獲取內容
function getNovel() {
  let key = Number(novelUrl.value);
  if (!key) return;
  dgyx = "";
  requestNet(`/html/47/47244/${key}.html`);
}
</script>

<template>
  <div class="about">
    <input v-model="novelUrl" />
    <button @click="getNovel">獲取</button>
  </div>
</template>

<style>
@media (min-width: 1024px) {
  .about {
    min-height: 100vh;
    display: flex;
    align-items: center;
  }
}
</style>
