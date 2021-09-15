<template>
  <div>
    <h3>{{ isNew ? "创建" : "编辑" }}课程</h3>
    <ele-form
      :form-data="data"
      :form-desc="fields"
      :request-fn="submit"
    ></ele-form>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";

@Component({})
export default class CoursesEdit extends Vue {
  @Prop(String) id!: string;
  data = {};
  fields = {
    name: { label: "课程名称", type: "input" },
    cover: { label: "课程封面", type: "input" },
  };

  get isNew() {
    return !this.id;
  }
  async fetch() {
    const res = await (this as any).$http.get("courses");
    this.data = res.data;
  }

  async submit(data) {
    // console.log(data);
    this.data = {};
    const url = this.isNew ? `courses` : `courses/${this.id}`;
    const method = this.isNew ? "post" : "put";
    await (this as any).$http[method](url, data);
    (this as any).$message.success("保存成功");

    (this as any).$router.go(-1);
  }

  created() {
    !this.isNew && this.fetch();
  }
}
</script>

<style></style>
