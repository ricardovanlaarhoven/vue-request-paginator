<template>
  <div>
    <request-paginator :request="request" :items-per-page="itemsPerPage">
      <template #loadingIndicator>
        <div v-for="n in itemsPerPage" :key="n">Loading..</div>
      </template>
      <template #default="{items}">
        <div v-for="item in items" :key="item.id">
          {{ item.owner }} - {{ item.pet }}
        </div>
      </template>
    </request-paginator>
  </div>
</template>

<script lang="ts">
import {Component, Vue} from "vue-property-decorator";
import RequestPaginator from "../src/RequestPaginator.vue";
import * as faker from 'faker';

@Component({
  components: {RequestPaginator}
})
export default class Playground extends Vue {
  items = []
  itemsPerPage = 10;

  created(): void {
    for (let i = 0; i < 100; i++) {
      this.items.push({
        id: i + 1,
        owner: faker.name.findName(),
        pet: faker.animal.dog()
      })
    }
  }

  async request(pageNumber: number, itemsPagePage: number): Promise<any> {
    await this.sleep(1000);
    return this.items.slice((pageNumber - 1) * itemsPagePage, (pageNumber - 1) * itemsPagePage + itemsPagePage);
  }

  sleep(ms: number): Promise<any> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}
</script>
