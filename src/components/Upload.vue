<template>
  <v-container class="card">
    <v-card>
      <v-card-title>作业提交</v-card-title>
      <div class="form">
        <v-select
          :items="items"
          label="选择作业"
          v-model="info.selectedWork"
          :disabled="isLoading"
        >
        </v-select>
        <v-text-field
          v-model="info.stu_name"
          :counter="10"
          label="名称"
          required
          :disabled="isLoading"
        ></v-text-field>
        <v-text-field
          v-model="info.stu_id"
          :counter="10"
          label="学号"
          required
          :disabled="isLoading"
        ></v-text-field>
        <v-file-input
          v-model="info.fileInfo"
          truncate-length="15"
          :disabled="isLoading"
          accept=".zip,.rar,.7zip"
        ></v-file-input>
        <v-progress-linear
          color="light-blue"
          height="15"
          :value="progress"
          striped
        >
        </v-progress-linear>
      </div>
      <v-card-actions>
        <v-btn
          color="deep-purple lighten-2"
          text
          @click="uploadFile"
          :loading="isLoading"
          :disabled="uploadDisable"
        >
          确认上传
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-container>
</template>

<script>
const COS = require("cos-js-sdk-v5");
import DayJs from "dayjs";
const url = require("./config.json").url;
const works = require("./works.json");

export default {
  watch: {
    changeData: {
      handler(newValue) {
        if (
          newValue.selectedWork === "" ||
          newValue.stu_id === "" ||
          newValue.stu_name === "" ||
          newValue.fileInfo.name == undefined
        )
          this.uploadDisable = true;
        else this.uploadDisable = false;
      },
    },
  },
  computed: {
    changeData() {
      const { selectedWork, fileInfo, stu_id, stu_name } = this.info;
      return {
        selectedWork,
        fileInfo,
        stu_id,
        stu_name,
      };
    },
  },
  data: () => ({
    progress: 0,
    uploadDisable: true,
    isLoading: false,
    info: {
      selectedWork: "",
      fileInfo: {},
      stu_id: "",
      stu_name: "",
    },
    items: works.works_title,
  }),
  methods: {
    uploadFile() {
      this.uploadDisable = true;
      this.isLoading = true;
      this.signInAvatarLoading = true;
      const _this = this;
      const cos = new COS({
        getAuthorization: function (options, callback) {
          // 异步获取临时密钥
          _this.axios
            .get(url + "/cos/authkey?bucket=homework-1251621542")
            .then((res) => {
              var credentials = res.data.credentials;
              console.log(credentials);
              callback({
                TmpSecretId: credentials.tmpSecretId,
                TmpSecretKey: credentials.tmpSecretKey,
                XCosSecurityToken: credentials.sessionToken,
                // 建议返回服务器时间作为签名的开始时间，避免用户浏览器本地时间偏差过大导致签名错误
                StartTime: res.data.startTime, // 时间戳，单位秒，如：1580000000
                ExpiredTime: res.data.expiredTime, // 时间戳，单位秒，如：1580000900
              });
            });
        },
      });
      cos.putObject(
        {
          Bucket: "homework-1251621542" /* 必须 */,
          Region: "ap-guangzhou" /* 存储桶所在地域，必须字段 */,
          Key:
            this.info.selectedWork +
            "/" +
            this.info.stu_id +
            "/" +
            DayJs().format("YYYY-DD-MM-HH-mm-ss") +
            this.info.stu_id +
            "-" +
            this.stu_name +
            this.info.fileInfo.name,
          StorageClass: "STANDARD",
          Body: this.info.fileInfo, // 上传文件对象
          onProgress: function (progressData, ) {
            console.log(progressData)
            _this.progress = (progressData.percent * 100);
          },
        },
        (err, data) => {
          this.uploadDisable = false;
          this.isLoading = false;
          if (err) {
            console.log(err);
          } else {
            console.log(data);
            if (data.statusCode === 200) {
              console.log(data.Location);
              alert("成功");
            } else {
              alert("失败");
            }
          }
        }
      );
    },
  },
};
</script>

<style lang="scss" scoped>
.card {
  margin: auto;
  width: 50%;
}

.form {
  margin: 50px;
}
</style>