<template>
  <div>
    <slot name="loadingIndicator" v-if="showOverallLoadingIndicator"></slot>
    <slot :items="paginatedResults" :page-number="internalCurrentPage" v-else></slot>
    <slot name="paginator"
          :is-loading="isLoading"
          :next-page="nextPage"
          :previous-page="previousPage"
          :update-pagination="updatePagination"
          :last-page="lastPage"
          :currentPage="internalCurrentPage">
      <button @click="nextPage" v-if="!lastPage || internalCurrentPage < lastPage">Load more</button>
    </slot>
  </div>
</template>

<script lang="ts">
import {Component, Prop, Vue, Watch} from 'vue-property-decorator';

@Component
export default class RequestPaginator extends Vue {
  /**
   * The request function that will be called for all pagination requests
   */
  @Prop({required: true}) readonly request!: (pageNumber: number, itemsPagePage: number, params?: { [index: string]: { message: any } }) => Promise<any>
  /**
   * The params that will be send to the 3th parameter of the request function
   */
  @Prop() readonly params: { [index: string]: { message: any; }; } | undefined
  @Prop({default: 10}) readonly itemsPerPage!: number
  /**
   * Determines if it should display a single page at a time,
   * or that the result is concatenated with the previous pages
   */
  @Prop({default: true}) readonly singlePage!: boolean
  /**
   * A function that distills the results from the request's response
   */
  @Prop({default: () => (response: any): [] => response}) readonly resultDistiller!: ((response: any) => [])
  @Prop() readonly lastPage!: number
  @Prop() readonly currentPage!: number

  isLoading = true
  isInitialLoading = true
  paginatedResults = []
  internalCurrentPage = 1

  @Watch('currentPage') onCurrentPageChange(): void {
    this.internalCurrentPage = this.currentPage;
  }

  @Watch('internalCurrentPage', {immediate: true}) onInternalCurrentPageChange(): void {
    this.handleRequest();
  }

  /**
   * When there is only one page at a time, the global loading indicator should be visible during
   * the switch between pages.
   * When the result of pages is stacked you should only see the global loading indicator the first time
   */
  get showOverallLoadingIndicator(): boolean {
    return this.singlePage ? this.isLoading : this.isInitialLoading;
  }

  nextPage(): void {
    this.updatePagination(this.internalCurrentPage + 1);
  }

  previousPage(): void {
    this.updatePagination(this.internalCurrentPage - 1);
  }

  reload(): void {
    if (this.internalCurrentPage !== 1) {
      this.updatePagination(1);
      return;
    }
    this.handleRequest();
  }

  updatePagination(page: number): void {
    this.$emit('currentPage', page);
    this.internalCurrentPage = page;
  }

  async handleRequest(): Promise<void> {
    this.isLoading = true;
    const response = await this.request(this.internalCurrentPage, this.itemsPerPage);
    await this.applyResponse(response);
    this.isLoading = false;
    this.isInitialLoading = false;
  }

  applyResponse(response: any): void {
    if (this.singlePage) {
      this.paginatedResults = this.resultDistiller(response);
      return;
    }
    this.paginatedResults = [...this.paginatedResults, ...this.resultDistiller(response)];
  }
}
</script>
