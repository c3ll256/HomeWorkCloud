<template>
  <v-container class="card">
    <v-card>
      <v-card-title>作业提交</v-card-title>
      <div class="form">
        <v-select :items="items" label="选择课程"> </v-select>
        <v-text-field
          v-model="stu_name"
          :counter="10"
          label="名称"
          required
        ></v-text-field>
        <v-text-field
          v-model="stu_id"
          :counter="10"
          label="学号"
          required
        ></v-text-field>
        <v-file-input truncate-length="15"></v-file-input>
      </div>
      <v-card-actions>
        <v-btn color="deep-purple lighten-2" text @click="reserve">
          确认上传
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-container>
</template>

<script>
export default {
  data: () => ({
    stu_id: "",
    stu_name: "",
    items: ["a", "b", "c"],
  }),
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