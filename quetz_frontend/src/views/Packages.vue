<template>
<div class="bx--grid">
  <div class="bx--row">
    <div class="bx--col-lg-13 bx--offset-lg-3">
        <h3 class="bx--data-table-header">Packages in {{ $route.params.channel_id }}</h3>
        <cv-data-table
          :columns="columns"
          :data="data"
          :pagination="pagination"
          @search="onFilter" searchLabel="Filter" searchPlaceholder="Filter" searchClearLabel="Clear filter"
          @pagination="loadPagination"
          ref="table">
        </cv-data-table>
    </div>
  </div>
</div>
</template>

<script>
  export default {
    data: function () {
      return {
        columns: [],
        data: [],
        loading: true
      }
    },
    methods: {
      // TODO double request currently, also save value of #packages to display in local storage?
      loadPagination: function(args, searchquery) {
        let url = "/api/channels/" + this.$route.params.channel_id + "/packages?"
        this.selected_pagesize = args['length'];
        url += new URLSearchParams({
          limit: args['length'],
          skip: args['start'] - 1
        });
        if (searchquery)
        {
          url += "&" + new URLSearchParams({
            q: searchquery
          })
        }
        return fetch(url).then((msg) => {
          return msg.json().then((decoded) => {
              console.log(decoded);
              this.columns = ["Name", "Description"];
              console.log(decoded.pagination)
              this.pagination = {
                numberOfItems: decoded.pagination.all_records_count,
                pageSizes: [25, 50, 100, 1000]
              };
              this.numberOfItems = decoded.pagination.all_records_count;
              this.data = decoded.result.map((el) => [el.name, el.description]);
          });
        });
      },
      onFilter: function(query) {
        this.loadPagination({length: 25, start: 1}, query)
      },
    },
    created: function() {
      this.pagination = {
        numberOfItems: 0,
        pageSizes: [25, 50, 100, 1000]
      };
      this.loadPagination({length: 25, start: 1});
    },
}
</script>

<style>
html {
   height: 100%;
}

body, #app {
   min-height: 100%;
}

</style>
