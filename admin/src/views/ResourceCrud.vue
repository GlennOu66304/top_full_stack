<template>
  <div>
    <avue-crud
        v-if="option.column"
      :data="data.data"
      :option="option"
        :page="page"
      @row-save="create"
      @row-update="update"
      @row-del="remove"
        @on-load="changePage"
        @sort-change="changeSort"
        @search-change="search"
    ></avue-crud>
  </div>
</template>

<script lang="ts">
import {Component, Vue,Prop} from "vue-property-decorator";

@Component({})
export default class ResourceList extends Vue {
  @Prop(String) resource : string;
  data = {};
  // fields = {
  //   _id: { label: "ID" },
  //   name: { label: "课程名称" },
  //   cover: { label: "课程封面" },
  // };

  option = {};
  page = {
    pageSize:2,
    pageSizes: [ 2, 5, 10],
    total: 2
  };
  query:any={
    limit:2
  }

  async fetchOption() {
    const res = await (this as any).$http.get(`${this.resource}/option`);
    this.option = res.data;
  }

  async fetch() {
    const res = await (this as any).$http.get(`${this.resource}`,{
      params:{
        query:this.query
      }
    });
    this.page.total = res.data.total
    this.data = res.data;
  }
async changePage({currentPage, pageSize}){
    this.query.page = currentPage
    this.query.limit = pageSize
   this.fetch();
}
async changeSort({prop,order}){
    if(!order){
      this.query.sort = null
    }else{
      this.query.sort = {
        [prop]: order === 'ascending' ? 1 : -1
      }
    }

  this.fetch();
}
  async search(where){

    for (let k in where) {
      const field = this.option.column.find(v=> v.prop === k);
      if(field.regex){
        where[k] = {$regex:where[k]};
      }
    }

   this.query.where = where
    this.fetch();
  }
  // async uploadBefore (file,done) {
  //   let params = new FormData()
  //   params.append('file',file)
  //   let res:any = await this.$http.post('upload',params, {
  //     headers: { 'Content-Type': 'multipart/form-data;charset=UTF-8' }
  //   })
  //   console.log(res)
  //   this.obj.cover = res.data.url
  //   done()
  // }

  async create(row, done) {
    await (this as any).$http.post(`${this.resource}`, row);
    (this as any).$message.success("创建成功");
    done();
    this.fetch();
  }

  async update(row, index, done) {
    const data = JSON.parse(JSON.stringify(row));
    delete data.$index;
    await (this as any).$http.put(`${this.resource}/${row._id}`, data);
    (this as any).$message.success("更新成功");
    this.fetch();
    done();
  }

  async remove(row) {
    try {
      await (this as any).$confirm("是否确认删除");
    } catch (e) {
      return;
    }

    await (this as any).$http.delete(`${this.resource}/${row._id}`);
    (this as any).$message.success("删除成功");
    this.fetch();
  }

  created() {
    this.fetchOption()
    this.fetch();
  }
}
</script>

<style></style>
