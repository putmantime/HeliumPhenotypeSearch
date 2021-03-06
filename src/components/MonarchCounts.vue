<template>

  <div class="card bg-light" style="height: 100%; max-height: 450px">
    <div class="card-header">
      <strong>Monarch Association Counts</strong>
      <img style="max-height: 20px; float:right" src="../assets/img/monarch-logo.png">
    </div>
    <div class="card-body">
      <div v-if="hasData">
      <b-table
          small
          hover
          :items="items"
          :fields="fields"
          @row-clicked="nodePage"
      >
        <!-- A virtual column -->
        <template slot="association" slot-scope="data">
          <div class="py-2">{{firstCap(data.item.association)}}</div>
        </template>
        <template slot="count" slot-scope="data">
          <div class="py-2">
            <a target="_blank" :href="monarchUrlAnchored(data.item.category)">{{data.item.count}}</a></div>

        </template>
      </b-table>
      </div>
      <div v-else>No Associations</div>
    </div>
  </div>
</template>
<script>
  import * as BL from '@/api/BioLink';

  export default {
    name: 'MonarchCounts',
    props: {
      curie: {
        type: String,
        required: true,
      },
      category: {
        type: String,
        required: true,
      }
    },
    data() {
      return {
        hasData: false,
        items: [],
        fields: [
          {key: 'association', label: 'Association', class: 'bold'},
          {key: 'count', label: 'Count', class: 'bold'}
        ]
      };
    },
    mounted() {
      this.fetchCounts(this.curie, this.category);
    },
    watch: {
      curie() {
        this.fetchCounts(this.curie, this.category);
      }
    },
    methods: {
      async fetchCounts(nodeID, nodeType) {
        try {
          const searchResponse = await BL.getCountsForNode(nodeID, nodeType);
          this.formatCounts(searchResponse);
        }
        catch (e) {
          console.log('nodeResponse ERROR', e, this);
        }
      },
      formatCounts(countsData) {
        this.items = [];
        if (countsData) {
          this.hasData = true;
          Object.entries(countsData).forEach((value, key) => {
            this.items.push(
              {
                association: value[0],
                count: value[1].totalCount
              }
            );
          })
        } else {
          this.hasData = false;

        }
      },
      firstCap(term) {
        return  term.charAt(0).toUpperCase() + term.substr(1);
      },
      monarchUrlAnchored(cardType) {
        let url = `https://monarchinitiative.org/${this.category.toLowerCase()}/${this.curie}#${cardType}s`;
        return url;
      },
      dictToList(key, value) {
        return {
          'association': key,
          'count': {'url': key, 'count': value['totalCount']},
        }
      },
      nodePage(item) {
        location.assign(this.monarchUrlAnchored(item.association));
      }
    }
  };
</script>
<style>
  .bold {
    font-weight: bold;
  }
</style>
