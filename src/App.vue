<template>
  <div class="content">
    <el-card class="box-card">
      <el-input
        v-model="q"
        @keydown.enter="translate"
        ref="ipt"
        @focus="q = ''"
        :placeholder="placeholder"
      />
      <el-button type="primary" @click="translate"> 回车或点此翻译 </el-button>
      <div class="result">
        变量名为:<span>{{ r }}</span>
      </div>
    </el-card>
  </div>
</template>

<script setup>
import axios from "axios";
import Qs from "qs";
import { ElMessage } from "element-plus";
import MD5 from "md5";
import { ref } from "vue";
const placeholder = ref("请输入要翻译的文本");
let q = ref(""); //查询文本
const r = ref(""); //翻译结果
const ipt = ref(""); // 获取input元素，用于自动聚焦
const appid = "20210927000958371"; //应用id
const salt = Math.random(); // 随机数
const key = "DJO8xUvWBadaNCK5pbZu"; // 密钥
const translate = async () => {
  //请求参数
  const data = {
    q: q.value,
    appid,
    salt,
    from: "zh",
    to: "en",
    sign: MD5(appid + q.value + salt + key),
  };
  // 请求结果
  const res = await axios({
    headers: {
      "Content-Type": "application/x-www-form-urlencoded",
    },
    method: "post",
    url: "/api/trans/vip/translate",
    data: Qs.stringify(data),
  });
  console.log(res);
  if (!q.value) {
    console.log(placeholder);
    ElMessage.error(placeholder.value);
    return;
  }
  if (res.status === 200) {
    if (res.data.error_code === "54003") {
      ElMessage.error("请求百度翻译接口过于频繁，请控制在1s左右");
      console.log("测试");
    } else {
      // 驼峰命名
      const result = res.data.trans_result[0].dst.split(" ");
      let name = result
        .map((v, i) => {
          return i != 0
            ? v.charAt(0).toLocaleUpperCase() + v.slice(1)
            : v.toLocaleLowerCase();
        })
        .join("");
      console.log("name", name);
      // 处理逗号
      name = name.split(",").join("");
      // 处理撇号 如i'm
      name = name.split("'").join("");
      name = name.split("?").join("");
      r.value = name;
      console.log(r.value);
      // 自动复制
      const input = document.createElement("input");
      document.body.appendChild(input);
      input.setAttribute("id", "saveStr");
      document.getElementById("saveStr").value = r.value;
      input.value = r.value;
      input.select();
      document.execCommand("Copy");
      document.body.removeChild(input);
      ElMessage.success("已自动复制");
      // 自动聚焦
      ipt.value.focus("");
    }
  } else {
    ElMessage.error("请求百度翻译接口出错,请稍后再试");
  }
};
</script>
<style>
body {
  background-image: url("./assets/bg.jpg");
  background-size: cover;
}
.el-card {
  width: 500px;
  min-width: 300px;
  padding: 50px;
  text-align: center;
}

.el-input {
  min-width: 300px;
}

.el-input__inner {
  text-align: center !important;
}

.el-button {
  margin-top: 20px;
}

.result {
  margin-top: 20px;
  min-width: 300px;
}

.result span {
  color: #409eff;
  font-weight: bold;
  width: inherit;
  display: block;
  white-space: normal;
}
.content {
  display: flex;
  justify-content: center;
  margin: 200px 0;
}
</style>
